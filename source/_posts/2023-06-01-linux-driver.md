---
title: linux驱动开发
date: 2023-06-01 19:18:22
tags: [Linux]
categories: [Program, C/C++]
---

# 前言

# 一、环境搭建

# 二、开发注意事项

## 1. 基本模板

### 1.1. makefile编写

```makefile
ARCH := $(shell uname -r)
obj-m := hello.o
KERNEL := /lib/modules/$(ARCH)/build
PWD := $(shell pwd)
CONFIG_BBB = n

hello-obj += hello_main.o file1.o
hello-y += file2.o
hello-$(CONFIG_BBB) += file3.o

modules :
	$(MAKE) -C $(KERNEL) M=$(PWD) modules

clean :
	$(MAKE) -C $(KERNEL) M=$(PWD) clean
```

#### 1) 命令解释

- linux驱动编译最核心的命令是`make -C <kernel-source-dir> M=$(PWD) modules`
- `-C <dir>`: 指定内核源码目录，里面存放最高级的makefile
- `M=<dir>`: 代表我们模块的makefile在这里

#### 2) makefile内容解释

- `obj-m := ${MODULE_NAME}.o`: 指定要编译的ko文件，`${MODULE_NAME}.o`会生成`${MODULE_NAME}.ko`文件，写多个会生成多个
- `${MODULE_NAME}-obj += xxx.o`: 指定要编译的多个c文件，每个`xxx.o`会找`xxx.c`文件去编译，注意不能和`${MODULE_NAME}`同名，同名会循环引用错误
    - 如果编译`hello.ko`有两个源文件`hello.c`和`file1.c`，需要指定`obj-m := hello.o`，而obj需要将`hello.c`重命名为`hello_main.c`，然后设置`hello-objs := hello_main.o file1.o `
    - 如果只有一个源文件`hello.c`，那么不用指定`hello-objs`，会找`hello.c`文件
    - 经过测试，此变量不能等于变量，需要显式指定文件
- `${MODULE_NAME}-y += xxx.o`: 和`-n`对应，某些文件是否编译进去，一般设定变量来决定是`y`还是`n`

### 1.2. 源文件结构

```cpp
#include <linux/init.h>
#include <linux/module.h>

MODULE_LICENSE("GPL");
static int hello_init(void) {
    printk(KERN_ALERT "hello, world\n");
    return 0;
}

static void hello_exit(void) {
    nf_unregister_net_hook(&init_net, &nfho);
    printk(KERN_ALERT "Goodby, cruel world\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

- 上面这样的结构，需要用`module_init`和`module_exit`定义insmod和rmmod对应的时间点的操作
- 打印只能使用`printk`，会输出到`dmesg`命令的输出中
- `MODULE_LICENSE`定义开源协议

# 三、实战示例

## 1. netfilter框架

### 1.1. 在INPUT链输出包的源ip和目的ip

```cpp
#include <linux/init.h>
#include <linux/module.h>
#include <linux/netfilter.h>
#include <linux/ip.h>
#include <linux/netfilter_ipv4.h>

static struct nf_hook_ops nfho;

static unsigned int hook_func(void *priv, struct sk_buff *skb, const struct nf_hook_state *state) {
    struct iphdr *ip = ip_hdr(skb);
    struct sockaddr_in src_addr, dst_addr;
    memset(&src_addr, 0, sizeof(src_addr));
    memset(&dst_addr, 0, sizeof(dst_addr));
    src_addr.sin_family = AF_INET;
    dst_addr.sin_family = AF_INET;
    src_addr.sin_addr.s_addr = ip->saddr;
    dst_addr.sin_addr.s_addr = ip->daddr;

    printk(KERN_INFO "Packet received %pI4 to %pI4\n", &src_addr.sin_addr, &dst_addr.sin_addr);
    return NF_ACCEPT;
}

MODULE_LICENSE("GPL");
static int hello_init(void) {
    nfho.hook = hook_func;
    nfho.hooknum = NF_INET_LOCAL_IN;
    nfho.pf = PF_INET;
    nfho.priority = NF_IP_PRI_FIRST;

    nf_register_net_hook(&init_net, &nfho);
    printk(KERN_ALERT "hello, world\n");
    return 0;
}

static void hello_exit(void) {
    nf_unregister_net_hook(&init_net, &nfho);
    printk(KERN_ALERT "Goodby, cruel world\n");
}

module_init(hello_init);
module_exit(hello_exit);
```
