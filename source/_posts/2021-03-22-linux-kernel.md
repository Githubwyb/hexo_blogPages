---
title: linux kernel源码学习记录
date: 2021-03-22 16:55:36
tags: [Linux]
categories: [Program, C/C++]
---

# 一、前言

本文为研究linux kernel源码所记录的一些笔记

源码下载路径
```
https://mirrors.edge.kernel.org/pub/linux/kernel/
```

# 二、linux启动过程

## 1. 从引导加载程序内核

### 1.1. cpu上电

1. 主板通电后，会启动cpu
2. cpu启动复位后，开始在实模式下工作
  - 所有x86兼容处理器均支持实模式
3. 启动后开始从地址`0xfffffff0`执行第一条指令，这个地址被放置了BIOS的入口
4. 但是实模式下只有16位寄存器，不能索引到上面的地址。其实这个地址被映射到了rom中
5. ROM中存放各个硬件厂商定制的BIOS或者UEFI的启动代码，用于找到硬盘并引导系统启动

### 1.2. bios启动

**UEFI和BIOS的区别**

BIOS启动流程：

- 系统开机
- 上电自检（Power On Self Test 或 POST）。POST过后初始化用于启动的硬件（磁盘、键盘控制器等）
- BIOS会运行BIOS磁盘启动顺序中第一个磁盘的首440bytes（MBR启动代码区域）内的代码。
- 启动引导代码从BIOS获得控制权，然后引导启动下一阶段的代码（如果有的话）（一般是系统的启动引导代码）。
- 再次被启动的代码（二阶段代码）（即启动引导）会查阅支持和配置文件。根据配置文件中的信息，启动引导程序会将内核和initramfs文件载入系统的RAM中，然后开始启动内核。

UEFI启动流程：

- 系统开机
- 上电自检（Power On Self Test 或 POST）。UEFI 固件被加载，并由它初始化启动要用的硬件。
- 固件读取其引导管理器以确定从何处（比如，从哪个硬盘及分区）加载哪个 UEFI 应用。
- 固件按照引导管理器中的启动项目，加载UEFI 应用。
- 已启动的 UEFI 应用还可以启动其他应用（对应于 UEFI shell 或 rEFInd 之类的引导管理器的情况）或者启动内核及initramfs（对应于GRUB之类引导器的情况），这取决于 UEFI 应用的配置。

**MBT和GPT**

MBR

1. bios在初始化和检查硬件之后，需要找一个可引导设备
   1. 初始可引导设备列表存在bios配置中，根据顺序一个一个找
2. 对于硬盘，引导扇区在第一个扇区（512字节）的头446字节，并且引导扇区最后必须是`0x55`和`0xaa`，这两个字节可以称为魔术字节，如果bios看到这两个字节，认为这个设备是可引导设备，这个也是MBR硬盘的第一个扇区的构成
3. 在实模式下，内存的组成如下，所以我们写bootloader程序需要加载到`0x7c00`，如果需要操作显示，需要写入到`0xa0000 ~ 0xbffff`

```
0x00000000 - 0x000003FF - Real Mode Interrupt Vector Table
0x00000400 - 0x000004FF - BIOS Data Area
0x00000500 - 0x00007BFF - Unused
0x00007C00 - 0x00007DFF - Our Bootloader
0x00007E00 - 0x0009FFFF - Unused
0x000A0000 - 0x000BFFFF - Video RAM (VRAM) Memory
0x000B0000 - 0x000B7777 - Monochrome Video Memory
0x000B8000 - 0x000BFFFF - Color Video Memory
0x000C0000 - 0x000C7FFF - Video ROM BIOS
0x000C8000 - 0x000EFFFF - BIOS Shadow Area
0x000F0000 - 0x000FFFFF - System BIOS
```

- linux kernel启动函数（main函数）并不是常规的main，是`init/main.c`里面的`start_kernel()`函数

# 三、数据结构

## 1. `container_of`从节点找value

- 这个东西看了好久才看懂，不过真的很强大
- 考虑一个场景，我们定义树需要怎么写，类似下面这样

```cpp
struct treeNode {
  treeNode *left;
  treeNode *right;
  void *value;
};
```

- 这时发现这个value每次都要定义，并且每个treeNode都需要新建地址
- linux的这群大佬就开始搞事情，如果treeNode的地址和value的地址结合一下，不用每次创建两个地址，一个地址搞定
- 这时候就出现一种定义方式

```cpp
struct treeNode {
  treeNode *left;
  treeNode *right;
};

struct valueTemplate {
  treeNode node;
  int value;
}
```

- 这样写，对树操作时不用关心value是啥，只需要关心自己的数据结构实现就好了
- 但是怎么找到value呢，`container_of`就出现了

```cpp
#define container_of(ptr, type, member) \
    (type *)((char *)(ptr) - (char *) &((type *)0)->member)

// 示例用法
void func(treeNode *node) {
  valueTemplate *value = container_of(node, struct valueTemplate, node)
}
```

- 展开一下

```cpp
valueTemplate *value = (valueTemplate *)((char *)node  - (char *)&((valueTemplate *)0)->node)
```

- 加地址是向后偏移，减地址是向前偏移，所以这句话意思是通过成员变量找到结构体指针
- 使用0地址的成员变量的地址偏移来计算结构体指针到成员变量的偏移量，然后用成员变量地址向前偏移去查找value
- 这个想法是真的强大

## 2. rbtree

```cpp
// include/linux/rbtree.h
struct rb_node {
	unsigned long  __rb_parent_color;
	struct rb_node *rb_right;
	struct rb_node *rb_left;
} __attribute__((aligned(sizeof(long))));
```

- 先参考 [地址对齐](/blogs/2022-06-06-computer-composition/#1-地址对齐) 了解为什么可以使用`__rb_parent_color`的低两位作为颜色

# 四、系统调用

## 1. epoll

- epoll对于fd的储存使用的是红黑树
- 使用链表保存处于就绪状态的fd

### 1.1. 接口定义

```cpp
// fs/eventpoll.c
SYSCALL_DEFINE1(epoll_create, int, size)
{
	if (size <= 0)
		return -EINVAL;

	return do_epoll_create(0);
}

/*
 * The following function implements the controller interface for
 * the eventpoll file that enables the insertion/removal/change of
 * file descriptors inside the interest set.
 */
SYSCALL_DEFINE4(epoll_ctl, int, epfd, int, op, int, fd,
		struct epoll_event __user *, event)
{
	struct epoll_event epds;

	if (ep_op_has_event(op) &&
	    copy_from_user(&epds, event, sizeof(struct epoll_event)))
		return -EFAULT;

	return do_epoll_ctl(epfd, op, fd, &epds, false);
}

SYSCALL_DEFINE4(epoll_wait, int, epfd, struct epoll_event __user *, events,
		int, maxevents, int, timeout)
{
	struct timespec64 to;

	return do_epoll_wait(epfd, events, maxevents,
			     ep_timeout_to_timespec(&to, timeout));
}
```

### 1.2. 数据结构

```cpp

```
