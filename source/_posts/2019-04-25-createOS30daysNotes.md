---
title: 30天自制操作系统笔记
date: 2019-04-25 15:58:18
tags:
categories: [Knowledge, Study]
---

# 前言

本文是《30天自制操作系统》中的windows翻译为linux上的一些实践，过程中也在逐步迁移linux的源码到里面。本身作为对linux上的汇编、启动、内存管理等的一个学习实践。在30天之前都是将此工程翻译到linux，30天后将变成linux移植过程的笔记

- 参考: https://github.com/cherishsir/ubuntu230os

# 关键词解释

- 启动区: (bootsector) 软盘第一个的扇区称为启动区。那么什么是扇区呢？计算机读写软盘的时候，并不是一个字节一个字节地读写的，而是以512字节为一个单位进行读写。因此，软盘的512字节就称为一个扇区。一张软盘的空间共有1440KB, 也就是1474560字节，除以512得2880, 这也就是说一张软盘共有2880个扇区。那为什么第一个扇区称为启动区呢？那是因为计算机首先从最初一个扇区开始读软盘，然后去检查这个扇区最后2个字节的内容。
- IPL: （initial program loader）启动程序加载器。启动区只有区区512字节，实际的操作系统不像hello-os这么小，根本装不进去。所以几乎所有的操作系统，都是把加载操作系统本身的程序放在启动区里的。有鉴于此，有时也将启动区称为IPL。但hello-os没有加载程序的功能，所以HELLOIPL这个名字不太顺理成章。如果有人正义感特别强，觉得“这是撒谎造假，万万不能容忍！＂，那也可以改成其他的名字。但是必须起一个8字节的名字，如果名字长度不到8字节的话，常要在最后补上空格。

# 第1天 从计算机结构到汇编程序入门

1. 环境windows
2. 二进制编辑器 notepad++安装hexeditor插件
3. 汇编编辑器 vscode安装x86 and x86_64 Assembly
4. 编译需要使用光盘中的nask编译器

## 1. vmware启动img

- 创建系统选择other/other
- 创建好需要添加硬件，选择软盘，然后使用文件，选择img即可启动

## 2. qemu启动img

```shell
qemu-system-x86_64 -enable-kvm -m 4G -smp 1 -fda common/haribote.img
```

# 第2天 汇编语言学习与Makefile入门

## 1. 标准FAT12软盘格式

```assembly
; 以下的记述用于标准FAT12格式的软盘
    JMP     entry
    DB      0x90
    DB      "HELLOIPL"      ; 启动区的名称可以是任意的字符串
    DW      512             ; 每个扇区（sector）的大小（必须为512字节）
    DB      1               ; 簇（cluster）的大小（必须为1个扇区）
    DW      1               ; FAT的起始位置（一般从第一个扇区开始）
    DB      2               ; FAT的个数（必须为2）
    DW      224             ; 根目录的大小（一般设成224项）
    DW      2880            ; 该磁盘的大小（必须是2880扇区）
    DB      0xf0            ; 磁盘的种类
    DW      9               ; FAT的长度（必须是9扇区）
    DW      18              ; 1个磁道（track）有几个扇区（必须是18）
    DW      2               ; 磁头数（必须是2）
    DD      0               ; 不使用分区，必须是0
    DD      2880            ; 重写一次磁盘大小
    DB      0, 0, 0x29      ; 意义不明，固定
    DD      0xffffffff      ; （可能是）卷标号码
    DB      "HELLO-OS   "   ; 磁盘的名称（11字节）
    DB      "FAT12   "      ; 磁盘格式名称（8字节）
    RESB    18              ; 先空出18字节
```

## 2. 汇编代表性寄存器介绍

### 16位寄存器

```shell
AX ---- accumulator         累加寄存器
CX ---- counter             计数寄存器
DX ---- data                数据寄存器
BX ---- base                基址寄存器
SP ---- stack pointer       栈指针寄存器
BP ---- base pointer        基址指针寄存器
SI ---- source index        源变址寄存器
DI ---- destination index   目的变址寄存器
```

### 8位寄存器

8位寄存器为16位寄存器的扩展，AL和AH一起代表AX，并不是单独的寄存器

```shell
AL ---- accumulator         累加寄存器低位
CL ---- counter             计数寄存器低位
DL ---- data                数据寄存器低位
BL ---- base                基址寄存器低位
AH ---- accumulator         累加寄存器高位
CH ---- counter             计数寄存器高位
DH ---- data                数据寄存器高位
BH ---- base                基址寄存器高位
```

### 32位寄存器

- 32位系统中使用的32位寄存器，低16位和上述16位相同，高16位没有寄存器名字
- 32位寄存器加`E`代表，如`EAX, ECX, EDX, EBX, ESP, EBP, ESI, EDI`

### 段寄存器，16位

```shell
ES ---- extra segment       附加段寄存器
CS ---- code segment        代码段寄存器
SS ---- stack segment       栈段寄存器
DS ---- data segment        数据段寄存器
FS ---- segment part 2      没有名称
GS ---- segment part 3      没有名称
```

## 3. CPU和内存

- CPU寄存器很少，32位也只有44个字节的空间，所以需要内存当**外部储存器**
- 内存和CPU使用管脚连接，速度虽然光速，但是比起来内部寄存器还是慢很多
- 程序代码储存在内存中，一条一条读取出来进行运行
- `0xf0000`附近存在bios本身
- 启动区内容的装载地址为`0x00007c00 -- 0x00007dff`，为IBM和intel规定的。所以ORG指令选择此处为起始地址，也仅有512字节

## 4. Makefile编写

- `ipl.S`是启动时最先执行的代码，需要编译到软盘镜像的第一个分区

```makefile
all: img

boot:
	nasm -w-zeroing -o bootloader ipl.S

# 生成img文件
#   将ipl.S的512B内容写入到软盘镜像的C0-H0-S1的位置，用于启动扇区
#   剩余需要将软盘扩容到2880个扇区，也就是1440KB
img: boot
	dd if=bootloader of=haribote.img count=1 bs=512
	dd if=/dev/zero of=haribote.img bs=512 seek=1 skip=1 count=2879

run :
	qemu-system-x86_64 -enable-kvm -m 4G -smp 1 -fda haribote.img

clean :
	rm -f bootloader haribote.img

```

# 第3天 进入32位模式并导入C语言

## 1. 软盘构成

<img src = "2019_06_14_01.bmp" width = 60%>

- 一面80个柱面
- 磁盘有两面
- 每个柱面18个扇区
- 一个扇区512字节
- 一共80 \* 2 \* 18 \* 512 = 1474560 Byte = 1440 KB
- `C0-H0-S1`代表柱面0，磁头0，扇区1
- 扇区从1开始计数，柱面从0开始计数

### (1) 软盘保存文件

参考 [制作FAT12软盘以查看软盘的根目录条目+文件属性+文件内容](https://www.cnblogs.com/pacoson/p/4816614.html)

- 使用命令保存二进制到软盘中

```makefile
# 生成img文件
#   将ipl.S的512B内容写入到软盘镜像的C0-H0-S1的位置，用于启动扇区
#   剩余需要将软盘扩容到2880个扇区，也就是1440KB
#   将编译出来的二进制放到软盘里面，也就放到了0x4400的位置
img: bootloader kernel
	dd if=bootloader.bin of=haribote.img count=1 bs=512 &>/dev/null
	dd if=/dev/zero of=haribote.img bs=512 seek=1 skip=1 count=2879 &>/dev/null
	mkdir -p ./floppy
	mount -o loop haribote.img ./floppy -o fat=12
	sleep 1
	cp kernel ./floppy
	sleep 1
	umount ./floppy
	rm -rf ./floppy
```

```shell
=> hexdump -C haribote.img
00000000  eb 4e 90 48 45 4c 4c 4f  49 50 4c 00 02 01 01 00  |.N.HELLOIPL.....|
00000010  02 e0 00 40 0b f0 09 00  12 00 02 00 00 00 00 00  |...@............|
00000020  40 0b 00 00 00 00 29 ff  ff ff ff 48 41 52 49 42  |@.....)....HARIB|
00000030  4f 54 45 4f 53 20 46 41  54 31 32 20 20 20 00 00  |OTEOS FAT12   ..|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  b8 00 00 8e d0 bc 00 7c  8e d8 b8 20 08 8e c0 b5  |.......|... ....|
00000060  00 b6 00 b1 02 be 00 00  b4 02 b0 01 bb 00 00 b2  |................|
00000070  00 cd 13 73 10 83 c6 01  83 fe 05 73 32 b4 00 b2  |...s.......s2...|
00000080  00 cd 13 eb e3 8c c0 83  c0 20 8e c0 80 c1 01 80  |......... ......|
00000090  f9 12 76 d1 b1 01 80 c6  01 80 fe 02 72 c7 b6 00  |..v.........r...|
000000a0  80 c5 01 80 fd 0a 72 bd  88 2e f0 0f e9 51 45 be  |......r......QE.|
000000b0  c7 7c 8a 04 83 c6 01 3c  00 74 09 b4 0e bb 0f 00  |.|.....<.t......|
000000c0  cd 10 eb ee f4 eb fd 0a  0a 4c 6f 61 64 20 65 72  |.........Load er|
000000d0  72 6f 72 0a 00 00 00 00  00 00 00 00 00 00 00 00  |ror.............|
000000e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 55 aa  |..............U.|
00000200  00 00 00 00 f0 ff 00 00  00 00 00 00 00 00 00 00  |................|
00000210  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00001400  00 00 00 00 f0 ff 00 00  00 00 00 00 00 00 00 00  |................|
00001410  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00002600  41 6b 00 65 00 72 00 6e  00 65 00 0f 00 0f 6c 00  |Ak.e.r.n.e....l.|
00002610  00 00 ff ff ff ff ff ff  ff ff 00 00 ff ff ff ff  |................|
00002620  4b 45 52 4e 45 4c 20 20  20 20 20 20 00 16 05 62  |KERNEL      ...b|
00002630  23 59 23 59 00 00 05 62  23 59 03 00 39 01 00 00  |#Y#Y...b#Y..9...|
00002640  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00004400  b0 13 b4 00 cd 10 c6 06  f2 0f 08 c7 06 f4 0f 40  |...............@|
00004410  01 c7 06 f6 0f c8 00 66  c7 06 f8 0f 00 00 0a 00  |.......f........|
00004420  b4 02 cd 16 a2 f1 0f be  5a c4 e8 50 00 b0 ff e6  |........Z..P....|
00004430  21 90 e6 21 fa e8 d0 00  b0 d1 e6 64 e8 c9 00 b0  |!..!.......d....|
00004440  df e6 60 e8 c2 00 0f 01  16 30 c5 0f 20 c0 66 83  |..`......0.. .f.|
00004450  c8 01 0f 22 c0 ea 90 c4  10 00 0a 0a 6b 65 72 6e  |..."........kern|
00004460  65 6c 20 69 73 20 6c 6f  61 64 69 6e 67 00 0a 0a  |el is loading...|
00004470  74 72 79 20 69 74 20 61  67 61 69 6e 00 8a 04 83  |try it again....|
00004480  c6 01 3c 00 74 09 b4 0e  bb 0f 00 cd 10 eb ee c3  |..<.t...........|
00004490  66 b8 08 00 8e d8 8e c0  8e e0 8e e8 8e d0 bc ff  |f...............|
000044a0  ff 3f 00 be 36 c5 00 00  bf 00 00 28 00 b9 00 00  |.?..6......(....|
000044b0  02 00 e8 41 00 00 00 be  00 7c 00 00 bf 00 00 10  |...A.....|......|
000044c0  00 b9 80 00 00 00 e8 2d  00 00 00 be 00 82 00 00  |.......-........|
000044d0  bf 00 02 10 00 8a 0d f0  0f 00 00 69 c9 00 12 00  |...........i....|
000044e0  00 81 e9 80 00 00 00 e8  0c 00 00 00 ea 00 00 00  |................|
000044f0  00 18 00 eb fe f4 eb fd  8b 06 83 c6 04 89 07 83  |................|
00004500  c7 04 83 e9 01 75 f1 c3  e4 64 a8 02 75 fa c3 90  |.....u...d..u...|
00004510  00 00 00 00 00 00 00 00  ff ff 00 00 00 92 cf 00  |................|
00004520  ff ff 00 00 00 9a 47 00  ff ff 00 00 28 9a 47 00  |......G.....(.G.|
00004530  1f 00 10 c5 00 00 f4 eb  fd 00 00 00 00 00 00 00  |................|
00004540  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00168000
```

- 文件名写在`0x0002600`的地方
- 文件内容写在`0x004400`的地方，这个和书本上的`0x004200`不太一样，linux上mount后写入文件就在`0x004400`位置
- 编译生成的第三阶段启动程序代码在`0x004400`位置

### (2) 当前编译出来的软盘分布

- `C0-H0-S1`: IPL启动区位于此扇区，操作系统启动时会自动装载到`0x7c00-0x7dff`，这个是iIBM和intel规定的默认行为
- `booloader.S + header.S`编译出来的`kernel`保存在软盘的`0x004400`

## 2. 内存寻址

- `ES : BX`代表内存寻址的地址，其中BX为0-3位，ES为4-位。如`ES=0x0820，BX=0，代表0x8200地址`。总内存为12位，1M左右。
- 内存`0x7c00-0x7dff`为启动区使用，`0x7e00-0x9fbff`没有什么用，留给操作系统开发使用
- 内存寻址需要指定段寄存器DS，不然就会加上其16倍的数据，所以一般DS = 0

## 3. 汇编和C语言链接

- 使用汇编可以编译出`.obj`（`.o`）文件，这个文件和C文件编译出来的是一个效果
- 可以使用`objdump`来查看c语言生成的汇编指令代码
- 既然都是原生汇编，按照c语言生成的汇编格式来写汇编，同样可以链接到c语言中

```assembly
.code32

# 定义全局符号，这个类似于C语言的export
.global io_hlt

# 指定.text段
.section .text

# 函数实现
io_hlt:
    hlt
    ret
```

## 4. BIOS介绍

- BIOS是使用16位机器语言，32位模式不能调用BIOS函数
- VRAM（video RAM）在当前画面模式下是`0xa0000 ~ 0xaffff`，这个是在BIOS文档中`INT 0x10`说明最后写着

### 1) BIOS函数

#### (1) 0x10 设置画面模式

- 设置AH寄存器为00
- 设置AL寄存器为下面的值（省略部分模式）
  - 0x03: 16色字符模式，80 x 25
  - 0x12: VGA图形模式，640 x 480 x 4位彩色模式，独特的4面储存模式
  - 0x13: VGA图形模式，320 x 200 x 8位彩色模式，调色板模式
  - 0x12: VGA图形模式，800 x 600 x 4位彩色模式，独特的4面储存模式（部分显卡不支持此模式）
- 无返回值

```assembly
; 设置屏幕模式
MOV		AL,0x13			; VGA图形，320 x 200 x 8位颜色
MOV		AH,0x00
INT		0x10
```

## 5. cpu介绍

- 为什么写程序使用`i486p`，这个是cpu指令集
- i486p是给486cpu使用，但是如果只是用16位寄存器，也可以8086用
- intel系列cpu家谱

```
8086->80186->286->386->486->Pentium->PentiumPro->PentiumII->PentiumIII->Pentium4->...
```

- 到286为止是16位cpu，386之后为32位

## 6. 内存重新分配

- 引入C语言后，代码编译出的大小越来越大，之前的空间不足。32位模式下可用内存达到4G，重新将磁盘上的代码拷贝到内存新的地址中。
- 选择0x00100000为软盘拷贝的首地址
- 选择0x00280000为c语言代码开始位置
- 选择0x002fffff-0x003fffff为c语言栈的大小，栈设定为1MB。esp从c语言开始赋值为0x003fffff（栈顶是高地址，栈底为低地址）

```assembly
; header.S文件节选
# 栈起始位置，共1MB。0x002fffff-0x003fffff
#define STACK    0x003fffff
# C语言代码所在内存位置
#define C_CODE	0x00280000
# 软盘数据所在内存位置
#define	DISK_ADDR		0x00100000
# 软盘数据所在内存位置（真实模式）
#define DISK_ADDR_REAL	0x00008000
...
	# 拷贝c代码到内存中
	movl	$start_kernel,%esi		# 源地址
	movl	$C_CODE,%edi			# 目的地址
	movl	$(512*1024/4),%ecx		# 拷贝512K，注意后面的C语言编译超过512K就要修改这里
	call	memcpy

	# 拷贝软盘数据到内存对应位置
	# 拷贝启动扇区
	movl	$0x00007c00,%esi		# 源地址
	movl	$DISK_ADDR,%edi			# 目的地址
	movl	$(512/4),%ecx			# 拷贝512B
	call	memcpy
	# 拷贝剩余数据
	movl	$(DISK_ADDR_REAL+512),%esi		# 源地址
	movl	$(DISK_ADDR+512),%edi			# 目的地址
	movb	(CYLS),%cl						# 把cyls地址内的值赋值给cl
	# 将ecx乘上512*8*2/4，cyls是bootloader读取的软盘中柱面数，一个柱面18个磁道，一个磁道2个扇区，一个扇区512B
	imul	$(512*18*2/4),%ecx
	subl	$(512/4),%ecx					# 去除启动扇区
	call	memcpy

	# 调用c语言第一个指令，位置是第3个段的首地址
	ljmp	$(3*8),$0x0000
...
start_kernel:
```

- 由于makefile中是直接将`cobjs`加到`hader`后面的，所以在`header.S`后面的`start_kernel`就是c语言的起始位置

## 7. makefile解释

```makefile
INCLUDE = -I$(TOOLPATH)/haribote/ -I./include/ -I./arch/x86/include/ -I./arch/x86/include/uapi/ -I.
CFLAGS  = -Wall -Werror -Wno-int-to-pointer-cast -Wno-unused
# -fno-pie 生成位置相关代码，位置无关代码暂时不清楚为什么会运行有问题，访问全局变量失败导致调色板设置有问题
# -fno-stack-protector gcc默认会添加栈安全检查，但是这个会依赖glibc的库，导致undefined reference to `__stack_chk_fail'的问题
CFLAGS += -MD -Os -fno-pie -nostdinc -nostdlib -fno-builtin -fno-stack-protector -fno-omit-frame-pointer -D_SIZE_T -DCONFIG_X86_32 -m32
LDFLAGS	= -m elf_i386 -no-pie

all: img

bootloader : bootloader.S
	nasm -w-zeroing -o $@.bin $@.S

#下面四个命令通过模式匹配获取当前目录下的所有C文件
SRCDIR = ./ ./lib/ ./init/

C_SOURCES = $(foreach d,$(SRCDIR),$(wildcard $(d)*.c))
C_OBJS = $(patsubst %.c,%.o,$(C_SOURCES))
C_DEPS = $(patsubst %.c,%.d,$(C_SOURCES))

%.o : %.c
	gcc $(INCLUDE) $(CFLAGS) -c -o $*.o $*.c

# 生成img文件
#   将ipl.S的512B内容写入到软盘镜像的C0-H0-S1的位置，用于启动扇区
#   剩余需要将软盘扩容到2880个扇区，也就是1440KB
#   将编译出来的二进制放到软盘里面，也就放到了0x4400的位置
img: bootloader kernel
	dd if=bootloader.bin of=haribote.img count=1 bs=512 &>/dev/null
	dd if=/dev/zero of=haribote.img bs=512 seek=1 skip=1 count=2879 &>/dev/null
	mkdir -p ./floppy
	mount -o loop haribote.img ./floppy -o fat=12
	sleep 1
	cp kernel ./floppy
	sleep 1
	umount ./floppy
	rm -rf ./floppy

kernel: header cobjs
	cat cobjs >> header
	cp header kernel

# kernel存在软盘的0x4400，拷贝到0x8000后就是0xc400
# header.S启动从bootsect_start开始
# header.S只能保留text段，因为cobjs在header.S最后面，多了段会导致拷贝磁盘出现偏移问题
header: header.S
	gcc $(INCLUDE) $(CFLAGS) -c -o $@.bin $<
	ld $(LDFLAGS) -N -e bootsect_start -Ttext 0xc400 -o $@.out $@.bin
	objdump -S -D $@.out > $@.asm
	objcopy -S -O binary -j .text $@.out $@

asmfunc.o: asmfunc.S
	gcc $(INCLUDE) $(CFLAGS) -c -o $@ $<

# c语言起始函数为start_kernel
cobjs: $(C_OBJS) asmfunc.o
	ld $(LDFLAGS) -N -T cobjs.ld -o $@.out $^
	objdump -S -D $@.out > $@.asm
	objcopy -S -O binary $@.out $@

run :
	qemu-system-x86_64 -enable-kvm -m 4G -smp 1 -fda haribote.img

clean:
	rm -f kernel
	rm -f cobjs cobjs.out cobjs.asm
	rm -f header header.bin header.out header.asm
	rm -f bootloader.bin haribote.img
	rm -f $(C_OBJS) $(C_DEPS)
	rm -f *.o *.d *.bin *.asm *.out
```

- 我们需要让`start_kernel`在cobjs的`.text`段第一个，有两个方案
    1. 将main.o在链接的时候放到第一个位置，需要在main.c中唯一
    2. 使用`.text.first`配合ld链接配置文件实现
- 这里使用第二个方案

```cpp
void HariMain();

void __attribute__((section(".text.first"))) start_kernel(void) {
    HariMain();
}
```

- 对应的`cobjs.ld`

```
ENTRY(start_kernel)

SECTIONS {
    . = 0x00280000;
	.text :  {
		*(.text.first)
        *(.text)
	}
}
```

# 第4天 C语言与画面显示的练习

## 1. 图形化界面

参考[linux-kernel启动过程](/blogs/2021-03-22-linux-kernel)

## 2. 调色板

- 调色板是显卡的一个模块，由于颜色只有8位，也就是256色，但是正常RGB有24位
- 所以我们可以给显卡设置256种颜色，0-255分别表示一种颜色
- **<font color="red">在用的时候直接设置对应内存为一个号码，显卡就会直接将对应位置显示成对应的颜色</font>**
- 但是cpu中断和调色板的io存取需要使用汇编来实现，c语言无法实现

**设置调色板**

- 先屏蔽中断
- 将想要设置的号码（0-255）写入到`0x03c8`
- 然后按照RGB的顺序写入`0x03c9`，想要继续设定，就直接继续写就行了
- 想要读出来对应号码的RGB，将号码写入到`0x03c7`，再从`0x03c9`读取3次。同理继续读就是下一个号码
- 最后恢复中断位

```cpp
void set_palette(int start, int end, unsigned char *rgb) {
    int i, eflags;
    eflags = io_load_eflags(); /* 记录终端许可标志的值 */
    io_cli();                  /* 将许可标志置0，禁止中断 */
    io_out8(0x03c8, start);
    for (i = start; i <= end; i++) {
        io_out8(0x03c9, rgb[0] / 4);    // R
        io_out8(0x03c9, rgb[1] / 4);    // G
        io_out8(0x03c9, rgb[2] / 4);    // B
        rgb += 3;
    }
    io_store_eflags(eflags); /* 恢复许可标志 */
    return;
}
```

## 3. 速度问题

运行起来后，要显示有颜色的画面一直是黑屏。等待了几分钟后才显示预期画面，差点以为代码有bug。不过这么慢也是个bug

排查了很久发现，原始makefile编译出来的其实是64位的c程序，运行的时候特别慢，应该是不兼容导致的。修改了makefile的cflags和ldflags后就运行特别快了。之前就算导入了c语言，也只是hlt，看不出效果，所以没有发现

```makefile
CFLAGS += -m32
LDFLAGS	= -m elf_i386
```

## 4. 使用c语言遇到的几个问题

- switch不能超过5个case，否则会出问题，不清楚为什么
- while中不能使用i--，不清楚为什么

# 第5天 结构体、文字显示与GDT/IDT初始化

## 1. 字符点阵

- 假设一个字符占用像素点为8x16，那么可以使用`char[16]`表示一个字符的点阵
- 作者找其他人要了一个`hankaku.txt`里面包含了`char`里面所有可见字符的点阵，总共由256个字符，占用4096个字节
- 在对应的vram位置设置color就可以直接显示字符

```cpp
// 由hankaku.txt编译而来
extern char hankaku[4096];

/**
 * @brief 输入一个字符
 *
 * @param vram 显示内存起始位置
 * @param xsize 屏幕的宽度，因为要换行显示，需要知道宽度
 * @param x 位置x
 * @param y 位置y
 * @param color 字体颜色
 * @param c 字符
 */
static void put_font8(char *vram, int xsize, int x, int y, char color, char c) {
    int i, j;
    char *font8 = hankaku + c * 16;
    for (i = 0; i < 16; ++i) {
        // 找到对应行的最左边
        char *p = vram + (y + i) * xsize + x;
        char tmp = font8[i];
        for (j = 0; j < 8; ++j) {
            if ((tmp & (0x80 >> j)) > 0) {
                p[j] = color;
            }
        }
    }
}
```

## 2. GDT和IDT

### 2.1. 分段

- 因为操作系统可以执行多个进程，但是每个进程使用的内存是独立的，需要使用分段让每个进程使用的内存隔开

### 2.2. GDT: global segment descriptor table

- 全局段记录表
- 段寄存器是16位，一位一个字节。一个段描述结构体是8个字节占用3位，所以低三位不能用，只有13位，一共8192个段描述结构体组成表
- 段寄存器可以指示65536个字节（64KB），cpu没这么大内存存储，所以需要放到内存里面，我们可以任意指定一块内存，将首地址和个数信息放到GDTR寄存器中就好了
- 段描述结构体见下面定义，来自于cpu手册，一共8个字节

```cpp
/* 8 byte segment descriptor */
struct desc_struct {
    u16 limit0;         // 段管理的内存上限low
    u16 base0;          // 段的对应的内存实际地址low
    u16 base1 : 8;      // 段的对应的内存实际地址mid
    u16 type : 4;
    u16 s : 1;          // 系统段为1，普通段为0
    u16 dpl : 2;
    u16 p : 1;
    u16 limit1 : 4;     // 段管理的内存上限low
    u16 avl : 1;
    u16 l : 1;
    u16 d : 1;
    u16 g : 1;          // 为1就是4K为单位定义上限（上限1，管理内存4K），为0则以一个字节为单位
    u16 base2 : 8;      // 段的对应的内存实际地址high
} __attribute__((packed));
```

- 随意取内存一段地址 `0x00270000 - 0x0027ffff` 这一段存储
- 将所有段初始化成全0
- 将段号1设置位cpu管理段，在内存的地址为0，大小为4GB，为32位内管理的最大内存，可读可写
- 段号为3的设置为`kernel`程序所在的内存段，地址在0x00280000，大小512K，可读可执行
- 最后通过汇编导出的函数写入GDTR，因为c语言无法设置GDTR

```cpp
#define ADR_GDT 0x00270000       // GDT的内存位置
#define LIMIT_GDT 0x0000ffff     // GDT占用的字节数
#define ADR_BOTPAK 0x00280000    // kernel所在的地址
#define LIMIT_BOTPAK 0x0007ffff  // kernel最大为512k
#define AR_DATA32_RW 0x4092      // 数据段，可读写
#define AR_CODE32_ER 0x409a      // 代码段，可读可执行，不可写

static void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar) {
    if (limit > 0xfffff) {
        ar |= 0x8000; /* G_bit = 1 */
        limit /= 0x1000;
    }
    sd->limit_low = limit & 0xffff;
    sd->base_low = base & 0xffff;
    sd->base_mid = (base >> 16) & 0xff;
    sd->access_right = ar & 0xff;
    sd->limit_high = ((limit >> 16) & 0x0f) | ((ar >> 8) & 0xf0);
    sd->base_high = (base >> 24) & 0xff;
    return;
}

void init_gdtidt(void) {
    struct SEGMENT_DESCRIPTOR *gdt = (struct SEGMENT_DESCRIPTOR *)ADR_GDT;
    struct GATE_DESCRIPTOR *idt = (struct GATE_DESCRIPTOR *)ADR_IDT;
    int i;

    // 初始化GDT
    for (i = 0; i < LIMIT_GDT / sizeof(struct SEGMENT_DESCRIPTOR); i++) {
        set_segmdesc(gdt + i, 0, 0, 0);
    }
    // cpu管理的总内存
    set_segmdesc(gdt + 1, 0xffffffff, 0x00000000, AR_DATA32_RW);
    // c语言的段，C语言只能使用第3个段，使用第4个或第2个都会在中断里面崩溃，不知道为什么
    set_segmdesc(gdt + 3, LIMIT_BOTPAK, ADR_BOTPAK, AR_CODE32_ER);
    load_gdtr(LIMIT_GDT, ADR_GDT);
    ...
}
```

- 对应汇编里面的实现

```assembly
# 第0个段是空的，全部是0
# 第1个段是数据段，cpu管理的总内存，从0x00000000开始，0x92代表可读可写，内存上限为0xfffff，g = 1代表4K为单位，4G
# 	按照结构体和16位低位高位计算 0xffff, 0x0000, 0x9200, 0x00cf
# 第2个段是代码段，启动区所在位置，从0x00000000开始，0x9a代表可读可执行，内存上限为0x7ffff，g = 0代表字节为单位，512K
# 	按照结构体和16位低位高位计算 0xffff, 0x0000, 0x9a00, 0x0047
# 第3个段是代码段，c代码所在位置，从0x00280000开始，0x9a代表可读可执行，内存上限为0x7ffff，g = 0代表字节为单位，512K
# 	按照结构体和16位低位高位计算 0xffff, 0x0000, 0x9a28, 0x0047
.p2align	2	# 按照2^2 = 4字节对齐
gdt:
	.word	0x0000,0x0000,0x0000,0x0000		# null seg
	.word	0xffff,0x0000,0x9200,0x00cf		# 可读写的段，数据段
	.word	0xffff,0x0000,0x9a00,0x0047		# 汇编代码所在内存位置
	.word	0xffff,0x0000,0x9a28,0x0047		# 给c代码使用的段
```

### 2.3. IDT: interrupt descriptor table

- 中断记录表，结构定义在cpu手册中

<img src="2022-10-24-01.png" />

- 代码定义的结构

```cpp
struct idt_bits {
    u16 ist : 3;
    u16 zero : 5;
    u16 type : 5;
    u16 dpl : 2;    // Descriptor Privilege Level
    u16 p : 1;
} __attribute__((packed));

struct gate_struct {
    u16 offset_low;         // 函数在段内的偏移地址low
    u16 segment;            // 段配置对应的偏移，比如第2号段，就是 2 * 8，一个段配置8个字节
    struct idt_bits bits;
    u16 offset_middle;      // 函数在段内的偏移地址mid
#ifdef CONFIG_X86_64
    u32 offset_high;
    u32 reserved;
#endif
} __attribute__((packed));
```

### 2.4. GDT、IDT、LDT和TSS的关系

- GDT，IDT都是全局的。LDT是局部的（在GDT中有它的描述符）
- GDT用来存储描述符（门或非门）；系统中几个CPU,就有几个GDT
- IDT整个系统只有一个
- 系统启动时候需要初始化GDT和IDT。LDT和进程相关，并不一定必有
- TSS: Task-State Segment，任务状态段，保存任务状态信息的系统段
- TSS只能存在于GDT中
- Task-Gate Descriptor，任务门描述符，用来保存和恢复任务的上下文信息。可以放到GDT、LDT、IDT中，里面的TSS段选择指向GDT的TSS描述符
- 下图为32位TSS结构

<img src="2022-10-24-03.png" />

- 下图为64位TSS结构

<img src="2022-10-24-02.png" />

# 第6天 中断处理

## 1. PIC: Programmable interrupt controller

- 可编程中断控制器
- 就是一个芯片，将8个中断信号合成一个中断信号输出给cpu
- 当前电脑上不止8个外部设备，所以使用两个pic合并成15个中断信号（主PIC的IRQ2被从PIC占据）

<img src="2022-10-23-01.png" />

- PIC是外部设备，不能直接使用C语言的等于赋值，需要使用`io_out8`
- 主从PIC的寄存器赋值需要使用端口进行，具体端口定义如下

```cpp
#define PIC0_ICW1 0x0020    // initial controlword，初始化控制数据。用于设置中断的模式，0x11为边沿触发
#define PIC0_OCW2 0x0020    // 用于通知PIC已经收到某中断，0x60+IRQ号即可，IRQ1就是0x61
#define PIC0_IMR 0x0021     // interrupt maskregister，中断屏蔽寄存器，8位对应8路中断信号，为1就屏蔽
#define PIC0_ICW2 0x0021    // 中断号的起始，设置为0x20，0-7号IRQ触发的中断号为0x20-0x27
#define PIC0_ICW3 0x0021    // 控制主从设定，第几位为1就代表几号IRQ和从PIC相连，当前cpu写死使用0x00000100，也就是IRQ2
#define PIC0_ICW4 0x0021    // 用于设置中断模式，0x01为无缓冲区模式

#define PIC1_ICW1 0x00a0    // 用于设置中断的模式，0x11为边沿触发
#define PIC1_OCW2 0x00a0    // 用于通知PIC已经收到某中断，设置这个同时也要设置PIC0_OCW2，0x62
#define PIC1_IMR 0x00a1     // interrupt maskregister，中断屏蔽寄存器，8位对应8路中断信号，为1就屏蔽
#define PIC1_ICW2 0x00a1    // 中断号的起始，设置为0x28，0-7号IRQ（对应主PIC的8-15号IRQ）触发的中断号为0x28-0x2f
#define PIC1_ICW3 0x00a1    // 仅用低3位表示和主设备的几号IRQ相连，当前写死为2，为主PIC的IRQ2
#define PIC1_ICW4 0x00a1    // 用于设置中断模式，0x01为无缓冲区模式
```

- `0x00-0x1f`中断号不能使用，是cpu内部用于产生错误的中断号，所以从0x20开始
- 设置PIC的代码如下

```cpp
void init_pic() {
    io_out8(PIC0_IMR, 0xff);  // 禁止所有中断
    io_out8(PIC1_IMR, 0xff);  // 禁止所有中断

    io_out8(PIC0_ICW1, 0x11);    // 边沿触发模式
    io_out8(PIC0_ICW2, 0x20);    // IRQ0-7由INT20-27接收
    io_out8(PIC0_ICW3, 1 << 2);  // PIC1由IRQ2连接
    io_out8(PIC0_ICW4, 0x01);    // 无缓冲区模式

    io_out8(PIC1_ICW1, 0x11);    // 边沿触发模式
    io_out8(PIC1_ICW2, 0x28);    // IRQ8-15由INT28-2f接收
    io_out8(PIC1_ICW3, 1 << 1);  // PIC1由IRQ2连接
    io_out8(PIC1_ICW4, 0x01);    // 无缓冲区模式

    io_out8(PIC0_IMR, 0xfb);  // 11111011 PIC1以外全部禁止
    io_out8(PIC1_IMR, 0xff);  // 禁止所有中断
}
```

## 2. 中断号对应的中断类型

| 硬件中断号 | 系统中断号 | 用途                    |
| ---------- | ---------- | ----------------------- |
| IRQ0       | INT20      |                         |
| IRQ1       | INT21      | PS/2键盘                |
| IRQ2       | INT22      | PIC1的中断              |
| IRQ3       | INT23      |                         |
| IRQ4       | INT24      |                         |
| IRQ5       | INT25      |                         |
| IRQ6       | INT26      |                         |
| IRQ7       | INT27      | 初始化PIC可能引发的中断 |
| IRQ8       | INT28      |                         |
| IRQ9       | INT29      |                         |
| IRQ10      | INT2a      |                         |
| IRQ11      | INT2b      |                         |
| IRQ12      | INT2c      | PS/2鼠标                |
| IRQ13      | INT2d      |                         |
| IRQ14      | INT2e      |                         |
| IRQ15      | INT2f      |                         |

## 3. 注册中断函数

- 也就是将中断函数地址写入idt中
- 找到对应中断号对应的idt地址，将函数地址和对应的段号放进去，设置标志位即可
- 由于中断最终要调用IRETD汇编指令退出，所以使用汇编函数调用c函数的方式来进行，并在里面存储了中断打断的进程的上下文信息
- gcc链接的时候，已经指定了地址在0x00280000，所有函数在汇编代码中就是真实地址（0x00280000后面），在汇编启动时拷贝到0x00280000，其他c代码实际使用就是这个地址，但是对于cpu来说，不关注是否拷贝到内存的哪个位置，只关注段内偏移，所以idt中设置的是在段中的偏移，这里需要减去ADR_BOTPAK

```cpp
// 注册中断处理函数
set_gatedesc(idt + 0x21, (int)asm_inthandler21-ADR_BOTPAK, 3 * 8, AR_INTGATE32);
set_gatedesc(idt + 0x27, (int)asm_inthandler27-ADR_BOTPAK, 3 * 8, AR_INTGATE32);
set_gatedesc(idt + 0x2c, (int)asm_inthandler2c-ADR_BOTPAK, 3 * 8, AR_INTGATE32);
```

## 4. 中断处理

- 中断中尽可能少执行代码，所以将中断数据放到全局变量中，在外部进程上下文中读取变量进行处理

# 第7天 FIFO与鼠标控制

## 1. 鼠标键盘数据读取

- 鼠标键盘都属于键盘控制电路，数据获取都在端口`0x0060`
- 只能通过中断号来判断此端口数据是属于鼠标还是键盘

## 2. fifo

- 由于中断数据可能很多，所以需要使用fifo进行存取，防止数据丢失
- fifo自己参考linux的实现，没有使用书上的实现，具体原理查看 [linux源码分析-kfifo](/bookPages/docs/linux-kernel/data-structures/kfifo/)

## 3. 鼠标初始化

- 由于一开始鼠标并不是必须品，后来鼠标才加入
- 鼠标加入后，当时使用者并不怎么使用，为了防止频繁中断，将鼠标控制默认关闭了
- 所以想要使用鼠标，需要先通过键盘控制电路设置模式启用鼠标，再进行特定操作进行激活鼠标

```cpp
#define PORT_KEYDAT 0x0060
#define PORT_KEYSTA 0x0064
#define PORT_KEYCMD 0x0064
#define KEYSTA_SEND_NOTREADY 0x02
#define KEYCMD_WRITE_MODE 0x60
#define KBC_MODE 0x47   // 鼠标模式

/**
 * @brief 等待键盘控制器可以发送数据
 *
 */
void wait_KBC_sendready() {
    for (;;) {
        if ((io_in8(PORT_KEYSTA) & KEYSTA_SEND_NOTREADY) == 0) {
            break;
        }
    }
}

/**
 * @brief 键盘控制器的初始化
 *
 */
void init_keyboard() {
    wait_KBC_sendready();
    io_out8(PORT_KEYCMD, KEYCMD_WRITE_MODE);
    wait_KBC_sendready();
    // 设置模式使用鼠标
    io_out8(PORT_KEYDAT, KBC_MODE);
    return;
}

#define KEYCMD_SENDTO_MOUSE 0xd4
#define MOUSECMD_ENABLE 0xf4

/**
 * @brief 使能鼠标
 *
 */
void enable_mouse() {
    wait_KBC_sendready();
    io_out8(PORT_KEYCMD, KEYCMD_SENDTO_MOUSE);
    wait_KBC_sendready();
    io_out8(PORT_KEYDAT, MOUSECMD_ENABLE);
    // 如果成功，就会发送ACK(0xfa)
}
```

## 4. 使用linux的idt和gdt的实现

```cpp
#include <asm/desc.h>
#include <asm/desc_defs.h>
#include <linux/kernel.h>

#include "asmfunc.h"

#define ADR_IDT 0x0026f800       // IDT的内存位置
#define LIMIT_IDT 0x000007ff     // IDT占用的字节数
#define ADR_GDT 0x00270000       // GDT的内存位置
#define LIMIT_GDT 0x0000ffff     // GDT占用的字节数
#define ADR_BOTPAK 0x00280000    // kernel所在的地址
#define LIMIT_BOTPAK 0x0000007f  // kernel最大为4K x 128 = 512KB

// 全局gdt表
static const struct gdt_page gdt_page = {.gdt = {
#ifdef CONFIG_X86_64
#else
                                             // kernel的代码段，也就是bootpack所在位置
                                             [GDT_ENTRY_KERNEL_CS] = GDT_ENTRY_INIT(0xc09a, ADR_BOTPAK, LIMIT_BOTPAK),
                                             // kernel控制的总内存大小
                                             [GDT_ENTRY_KERNEL_DS] = GDT_ENTRY_INIT(0xc092, 0, 0xfffff),
#endif
                                         }};

#define DPL0 0x0
#define DPL3 0x3

#define DEFAULT_STACK 0

#define G(_vector, _addr, _ist, _type, _dpl, _segment) \
    {                                                  \
        .vector = _vector,                             \
        .bits.ist = _ist,                              \
        .bits.type = _type,                            \
        .bits.dpl = _dpl,                              \
        .bits.p = 1,                                   \
        .addr = _addr,                                 \
        .segment = _segment,                           \
    }

/* Interrupt gate */
#define INTG(_vector, _addr) G(_vector, _addr, DEFAULT_STACK, GATE_INTERRUPT, DPL0, __KERNEL_CS)

/* System interrupt gate */
#define SYSG(_vector, _addr) G(_vector, _addr, DEFAULT_STACK, GATE_INTERRUPT, DPL3, __KERNEL_CS)

// idt表
static const struct idt_data idt_table[] = {
    // gcc链接的时候，已经指定了地址在0x00280000，所有函数在汇编代码中就是真实地址（0x00280000后面）
    // 在汇编启动时拷贝到0x00280000，其他c代码实际使用就是这个地址
    // 但是对于cpu来说，不关注是否拷贝到内存的哪个位置，只关注段内偏移，所以idt中设置的是在段中的偏移
    // 这里需要减去ADR_BOTPAK

    // PS/2键盘中断
    INTG(0x21, (int)asm_inthandler21-ADR_BOTPAK),
    // 处理27中断
    INTG(0x27, (int)asm_inthandler27-ADR_BOTPAK),
    // PS/2鼠标中断
    INTG(0x2c, (int)asm_inthandler2c-ADR_BOTPAK),
};

void init_gdtidt(void) {
    int i;

    // 初始化GDT
    struct desc_struct *gdt = (struct desc_struct *)ADR_GDT;
    // C语言的段
    write_gdt_entry(gdt, GDT_ENTRY_KERNEL_CS, &gdt_page.gdt[GDT_ENTRY_KERNEL_CS], DESCTYPE_S);
    // cpu管理的总内存
    write_gdt_entry(gdt, GDT_ENTRY_KERNEL_DS, &gdt_page.gdt[GDT_ENTRY_KERNEL_DS], DESCTYPE_S);
    // 设置gdtr
    load_gdtr(LIMIT_GDT, ADR_GDT);

    // 初始化IDT
    gate_desc *idt = (gate_desc *)ADR_IDT;
    gate_desc desc = {0};
    // 所有中断位置设置为空
    for (i = 0; i < LIMIT_IDT / sizeof(gate_desc); i++) {
        write_idt_entry(idt, i, &desc);
    }
    // 初始化要使用的中断
    const struct idt_data *t = idt_table;
    for (i = 0; i < ARRAY_SIZE(idt_table); ++i, ++t) {
        idt_init_desc(&desc, t);
        write_idt_entry(idt, t->vector, &desc);
    }

    load_idtr(LIMIT_IDT, ADR_IDT);
    return;
}
```

- 不过发现了一个问题，gcc编译不能使用`-O1`，只能使用`-Os`，否则中断会崩溃

# 第8天 鼠标控制与32位模式切换

## 1. 鼠标数据解读

- 鼠标使能后会先发送`0xfa`数据，然后会连续三个中断发送三个字节数据

## 2. 当前操作系统内存分布图

```
0x00000000 - 0x000fffff: 启动中多次使用。（1MB）
  - 0x00007c00 - 0x00007dff: 启动区装载地址，默认行为（512B）
  - 0x00008000 - 0x000081ff: 预留磁盘的C0-H0-S1位置，好计算偏移量（512B）
  - 0x00008200 - 0x00034fff: 磁盘上C0-H0-S2到C9-H1-S18读取位置（180KB）
    - 0x0000c400 -         : 磁盘的0x4400位置的内容，asmhead.nas
  - 0x000a0000 - 0x000affff: vram内存地址
  - 0x000f0000 附近存在bios本身
0x00100000 - 0x00267fff: 用于保存软盘内容。（1440KB）
0x00268000 - 0x0026f7ff: 空（30KB）
0x0026f800 - 0x0026ffff: IDT（2KB）
0x00270000 - 0x0027ffff: GDT（64KB）
0x00280000 - 0x002fffff: kernel（512KB）
0x00300000 - 0x003fffff: 栈及其他（1MB）
0x00400000 -           : 空
```

# 第9天 内存管理

## 1. 禁用高速缓存

- 486之后的cpu存在高速缓存，如果代码上设置一个内存立刻使用或修改，cpu会先用在缓存中，并不会直接写到内存中
- 内存检查就是写入内存一个值操作后读出来是否正常，这个时候就需要禁用cpu的高速缓存

```cpp
// asmfunc.h
static inline uint32_t io_load_eflags(void) {
    uint32_t eflags;
    asm volatile("pushfl; popl %0" : "=r"(eflags));
    return eflags;
}

static inline void io_store_eflags(uint32_t eflags) { asm volatile("pushl %0; popfl" : : "r"(eflags)); }

static inline uint32_t load_cr0(void) {
    uint32_t cr0;
    asm volatile("movl %%cr0, %0" : "=r"(cr0));
    return cr0;
}

static inline void store_cr0(uint32_t cr0) { asm volatile("movl %0, %%cr0" : : "r"(cr0)); }

// bootpack.c
unsigned int memtest(unsigned int start, unsigned int end) {
    bool before_386;
    unsigned int eflg, cr0, i;

    // 386及之前的cpu，没有AC这个标记位，所以设置之后还是返回0，使用此方式判断是否为386及以前
    eflg = io_load_eflags();
    eflg |= EFLAGS_AC_BIT;
    io_store_eflags(eflg);
    eflg = io_load_eflags();
    before_386 = (eflg & EFLAGS_AC_BIT) == 0;
    // 还原AC标记位
    eflg &= ~EFLAGS_AC_BIT;
    io_store_eflags(eflg);

    if (!before_386) {
        cr0 = load_cr0();
        cr0 |= CR0_CACHE_DISABLE; /* 禁止缓存 */
        store_cr0(cr0);
    }

    i = memtest_sub(start, end);

    if (!before_386) {
        cr0 = load_cr0();
        cr0 &= ~CR0_CACHE_DISABLE; /* 高速缓存许可 */
        store_cr0(cr0);
    }

    return i;
}
```

## 2. 内存检查和编译器优化

如果使用c语言实现内存检查

```cpp
static unsigned int memtest_sub(unsigned int start, unsigned int end) {
    unsigned int i, old, pat0 = 0xaa55aa55, pat1 = 0x55aa55aa;
    unsigned int *p;
    for (i = start; i <= end; i += 0x1000) {
        p = (unsigned int *)(i + 0xffc);
        old = *p;         /* 记录以前的值 */
        *p = pat0;        /* 尝试写 */
        *p ^= 0xffffffff; /* 反转 */
        if (*p != pat1) { /* 检查反转结果 */
        not_memory:
            *p = old;
            break;
        }
        *p ^= 0xffffffff; /* 再次反转 */
        if (*p != pat0) { /* 检查反转结果 */
            goto not_memory;
        }
        *p = old; /* 恢复原来的值 */
    }
    return i;
}
```

编译器会进行优化，发现第一个if判断没有意义，`pat0^0xffffffff`本来就是等于`pat1`啊，就算赋值给了`*p`不是一样吗

```cpp
static unsigned int memtest_sub(unsigned int start, unsigned int end) {
    unsigned int i, old, pat0 = 0xaa55aa55, pat1 = 0x55aa55aa;
    unsigned int *p;
    for (i = start; i <= end; i += 0x1000) {
        p = (unsigned int *)(i + 0xffc);
        old = *p;         /* 记录以前的值 */
        *p = pat0;        /* 尝试写 */
        *p ^= 0xffffffff; /* 反转 */
        *p ^= 0xffffffff; /* 再次反转 */
        if (*p != pat0) { /* 检查反转结果 */
            goto not_memory;
        }
        *p = old; /* 恢复原来的值 */
    }
    return i;
}
```

两次`*p^0xffffffff`数据保持原样，那就没有必要了

```cpp
static unsigned int memtest_sub(unsigned int start, unsigned int end) {
    unsigned int i, old, pat0 = 0xaa55aa55, pat1 = 0x55aa55aa;
    unsigned int *p;
    for (i = start; i <= end; i += 0x1000) {
        p = (unsigned int *)(i + 0xffc);
        old = *p;         /* 记录以前的值 */
        *p = pat0;        /* 尝试写 */
        if (*p != pat0) { /* 检查反转结果 */
            goto not_memory;
        }
        *p = old; /* 恢复原来的值 */
    }
    return i;
}
```

`*p = pat0`后比较两个的值，肯定一样啊

```cpp
static unsigned int memtest_sub(unsigned int start, unsigned int end) {
    unsigned int i, old, pat0 = 0xaa55aa55, pat1 = 0x55aa55aa;
    unsigned int *p;
    for (i = start; i <= end; i += 0x1000) {
        p = (unsigned int *)(i + 0xffc);
        old = *p;         /* 记录以前的值 */
        *p = old; /* 恢复原来的值 */
    }
    return i;
}
```

复制old后又赋值回去，等于没变

```cpp
static unsigned int memtest_sub(unsigned int start, unsigned int end) {
    unsigned int i, old, pat0 = 0xaa55aa55, pat1 = 0x55aa55aa;
    unsigned int *p;
    for (i = start; i <= end; i += 0x1000) {
        p = (unsigned int *)(i + 0xffc);
    }
    return i;
}
```

p就赋了一个值，全局都没有使用

```cpp
static unsigned int memtest_sub(unsigned int start, unsigned int end) {
    unsigned int i, old, pat0 = 0xaa55aa55, pat1 = 0x55aa55aa;
    unsigned int *p;
    for (i = start; i <= end; i += 0x1000) {}
    return i;
}
```

整个函数就剩下for循环了，所以一定返回输入什么就怎么样。解决这样的问题只需要在p前面修饰上`volatile`即可，编译器认为这个p是要实时变化的，就不会进行优化。

```cpp
static unsigned int memtest_sub(unsigned int start, unsigned int end) {
    unsigned int i, old, pat0 = 0xaa55aa55, pat1 = 0x55aa55aa;
    volatile unsigned int *p;       // 这里加上volatile，防止编译器优化
    // unsigned int *p = 0;
    for (i = start; i <= end; i += 0x1000) {
        p = (unsigned int *)(i + 0xffc);
        old = *p;         /* 记录以前的值 */
        *p = pat0;        /* 尝试写 */
        *p ^= 0xffffffff; /* 反转 */
        if (*p != pat1) { /* 检查反转结果 */
        not_memory:
            *p = old;
            break;
        }
        *p ^= 0xffffffff; /* 再次反转 */
        if (*p != pat0) { /* 检查反转结果 */
            goto not_memory;
        }
        *p = old; /* 恢复原来的值 */
    }
    return i;
}
```

# 第10天和第11天 窗口处理

主要处理图层、刷新等方式，使用图层添加了一个窗口，鼠标放到最上层。在刷新过程中发现了闪烁的问题，将刷新的方式优化了几板后没有了闪烁。主要结构如下

```cpp
#define SHEET_USE 1
struct SHTCTL;
struct SHEET {
    struct SHTCTL *ctl;
    unsigned char *buf;
    int bxsize;   // 宽度
    int bysize;   // 高度
    int vx0;      // 左上角坐标
    int vy0;      // 左上角坐标
    int col_inv;  // 透明标识，buf中和此值相同的值会被透明化
    int height;   // 高度，越大越在上面0-max
    int flags;
};

#define MAX_SHEETS 256

struct SHTCTL {
    unsigned char *vram;
    unsigned char *map;  // 标识哪里被哪一层sheet覆盖
    int xsize, ysize;
    int top;                           // 最高的sheet的索引+1
    struct SHEET *sheets[MAX_SHEETS];  // 按照高度顺序排列的已使用sheet的数组，最低的sheet在sheets[0]
    struct SHEET sheets0[MAX_SHEETS];  // 全部sheet包含使用和未使用
};
```

刷新只刷新某一个区域，对区域内的点，根据map取哪一层要写入则写入vram。

# 第12天 定时器（1）

## PIT（Programmable Interval Timer） 可编程的间隔型定时器

cpu有一个旁路芯片，8254芯片（或替代品），定时向cpu的IRQ0发起中断。如果我们接收此中断，设置中断周期就可以用于做定时器中断。

- 此芯片频率大概为1.19318MHz，我们设置一个计数来让计数到某个值产生一次中断
    - 1：1.19318MHz
    - 1000：1.19318KHz
    - 10000：119.318Hz
    - 11932：100Hz

对应的设置指令为

- OUT(0x43, 0x34): 0x43为pit_ctrl寄存器，0x34为mode
- OUT(0x40, 中断周期低8位):
- OUT(0x40, 中断周期高8位):

基于linux的实现，定义如下宏进行处理

```cpp
// include/linux/timex.h
/* The clock frequency of the i8253/i8254 PIT */
#define PIT_TICK_RATE 1193182ul

// include/asm-generic/param.h
# define HZ		CONFIG_HZ	/* Internal kernel timer frequency */

// include/linux/i8253.h
#define PIT_LATCH	((PIT_TICK_RATE + HZ/2) / HZ)
```

在makefile中定义`-DCONFIG_HZ=1000`即可实现，直接使用`PIT_LATCH`设置到中断周期里面

## 定时器中断优化

可以一步到位，使用链表管理定时器，设置定时器插入到链表里面按照超时时间从小到大排序。中断中仅关注第一个定时器是否到期即可。这里使用内核的hlist处理定时器，不需要timer_alloc也可以使用定时器。

```cpp
// bookpack.h
extern volatile unsigned long jiffies;
void init_pit(void);
#define MAX_TIMER 500
typedef STRUCT_KFIFO(unsigned char, 8) TimerBufType;
struct TIMER {
    struct hlist_node entry;
    unsigned long expires;  // 超时时间，取绝对时间，基于jiffies
    unsigned int flags;
    TimerBufType *fifo;
    unsigned char data;
};

void timer_free(struct TIMER *timer);
void timer_init(struct TIMER *timer, TimerBufType *fifo, unsigned char data);
void timer_settime(struct TIMER *timer, unsigned long timeout);

// timer.c
#include <linux/list.h>
volatile unsigned long jiffies = 0;

static HLIST_HEAD(s_timer_list);  // 按照到期时间排序

void init_pit(void) {
    // 芯片主频为PIT_TICK_RATE，想要1ms触发1次，HZ为1000，设置PIT_LATCH = PIT_TICK_RATE / HZ
    io_out8(PIT_MODE, 0x34);
    io_out8(PIT_CH0, PIT_LATCH & 0xff);
    io_out8(PIT_CH0, PIT_LATCH >> 8);
}

void timer_free(struct TIMER *timer) {
    struct TIMER *node;
    timer->flags = 0;
    hlist_for_each_entry(node, &s_timer_list, entry) {
        if (node == timer) {
            hlist_del_init(&node->entry);
            break;
        }
    }
}

void timer_init(struct TIMER *timer, TimerBufType *fifo, unsigned char data) {
    timer->fifo = fifo;
    timer->data = data;
    INIT_HLIST_NODE(&timer->entry);
}

void timer_settime(struct TIMER *timer, unsigned long timeout) {
    int e;
    timer->expires = timeout;

    // 操作s_timer_list需要关中断，防止并行操作
    e = io_load_eflags();
    io_cli();
    // 按照到期时间排序插入
    struct TIMER *node;
    struct TIMER *prev = NULL;  // 记录要插入的节点前一个的节点

    hlist_for_each_entry(node, &s_timer_list, entry) {
        if (node->expires > timer->expires) {
            break;
        }
        prev = node;
    }
    if (prev == NULL) {
        // 没有前一个节点，说明要插入到第一个
        // 1. 有节点，但是都不满足条件，插入最前面
        // 2. 没有节点，直接插入最前面
        hlist_add_head(&timer->entry, &s_timer_list);
    } else {
        // 有前一个节点，插入到前一个节点后面
        hlist_add_behind(&timer->entry, &prev->entry);
    }

    // 恢复中断
    io_store_eflags(e);
    io_sti();
}

void inthandler20(int *esp) {
    io_out8(PIC0_OCW2, 0x60);  // 通知PIC已经处理完毕
    ++jiffies;
    struct TIMER *node;
    struct hlist_node *n;
    hlist_for_each_entry_safe(node, n, &s_timer_list, entry) {
        if (node->expires <= jiffies) {
            hlist_del_init(&node->entry);
            kfifo_put(node->fifo, node->data);
        } else {
            break;
        }
    }
}
```

# 第13天 定时器（2）

## 串口输出日志

搞了这么久，发现出现一些问题没有办法进行排查，只能通过比较搓的方式直接在屏幕上输出某个字符串代表跑到了某个逻辑。而linux启动整个过程中可以有很多输出到串口或控制台上，参考linux的实现将日志加到操作系统中。

- 对于串口，直接将字符串写到`0x3f8`即可输出到串口中

```cpp
// log.c
#include <linux/jiffies.h>
#include <linux/kernel.h>
#include <linux/string.h>

#include "asmfunc.h"

/* Serial functions loosely based on a similar package from Klaus P. Gerlicher */

static unsigned long early_serial_base = 0x3f8; /* ttyS0 */

#define XMTRDY 0x20

#define DLAB 0x80

#define TXR 0 /*  Transmit register (WRITE) */
#define RXR 0 /*  Receive register  (READ)  */
#define IER 1 /*  Interrupt Enable          */
#define IIR 2 /*  Interrupt ID              */
#define FCR 2 /*  FIFO control              */
#define LCR 3 /*  Line control              */
#define MCR 4 /*  Modem control             */
#define LSR 5 /*  Line Status               */
#define MSR 6 /*  Modem Status              */
#define DLL 0 /*  Divisor Latch Low         */
#define DLH 1 /*  Divisor latch High        */

static inline unsigned int io_serial_in(unsigned long addr, int offset) { return inb(addr + offset); }

static inline void io_serial_out(unsigned long addr, int offset, int value) { outb(value, addr + offset); }

static inline int serial_putc(unsigned char ch) {
    unsigned timeout = 0xffff;

    while ((io_serial_in(early_serial_base, LSR) & XMTRDY) == 0) {
        --timeout;
    }
    io_serial_out(early_serial_base, TXR, ch);
    return timeout ? 0 : -1;
}

static inline void serial_write(const char *s, unsigned n) {
    while (*s && n-- > 0) {
        if (*s == '\n') serial_putc('\r');
        serial_putc(*s);
        s++;
    }
}

void serial_printf(const char *fmt, ...) {
    va_list ap;
    char buf[512];
    int n;

    va_start(ap, fmt);
    n = vscnprintf(buf, sizeof(buf), fmt, ap);
    va_end(ap);

    serial_write(buf, n);
}

void _print_current_time(void) {
    unsigned long msecs = jiffies_to_msecs(jiffies);
    unsigned long secs = msecs / 1000;
    unsigned long msec = msecs % 1000;
    serial_printf("[%5lu.%03lu]", secs, msec);
}
```

```cpp
// log.h
#ifndef __LOG_H__
#define __LOG_H__

void serial_printf(const char *fmt, ...);

void _print_current_time(void);

#define LOG_INFO(fmt, ...) \
    _print_current_time(); \
    serial_printf("[%s:%d %s] " fmt "\r\n", __FILE__, __LINE__, __FUNCTION__, ##__VA_ARGS__)

#endif  // __LOG_H__
```

makefile中修改一下qemu的参数，添加`-serial stdio`，然后在控制台就可以看到输出，代码中使用`LOG_INFO`打印即可。输出效果

```
[    0.000][bootpack.c:21 HariMain] HariMain start
[    0.000][bootpack.c:30 HariMain] init gdtidt done
[    0.002][bootpack.c:44 HariMain] create timer done
[    0.002][bootpack.c:50 HariMain] init keyboard and mouse done
[    0.004][bootpack.c:54 HariMain] init palette done
[    0.314][bootpack.c:64 HariMain] init memory done, cost 310ms, memory 2048MB, free: 2093688KB
[    0.324][bootpack.c:106 HariMain] init windows done
```

## 中断处理优化

其实就是搞了个数组实现链表的方式，将timer管理起来。之前使用了hlist来管理timer，天然就是链表，后面两个优化就不做了。

# 第14天 提高分辨率及键盘输入

## VBE（VESA BIOS Extensions）

参考：https://en.wikipedia.org/wiki/VESA_BIOS_Extensions

最开始电脑规格是IBM公司决定的，也制定了相关的规格，各家显卡公司只能迎合IBM规格制作显卡。一段时间后，显卡公司图像处理技术超过了IBM公司，就出现了各式各样的画面模式，造成了显卡公司竞争和设定方法使用方法各不相同。由于开发人员不想记忆那么多情况，所以很多高性能显卡也只使用最原始的320x200来使用。为了解决这个问题，各个显卡公司经过协商，成立了VESA协会（Video Electronics Standards Association，视频电子标准协会）。协会制定了一个通用设定方法，虽然不能完全兼容，但是可以通用。这个BIOS操作称为“VESA BIOS Extension”简称VBE。

在qemu上测试支持下面几个模式

- 0x0100: 640x400
- 0x0101: 640x480
- 0x0105: 1024x768
- 0x0107: 1280x1024
- 0x011c: 1600x1200

不过设置之后的vram的地址不是`0xe0000000`而是`0xfd000000`。设置方法为

```assembly
#define VBEMODE         0x0105

	mov 	$(VBEMODE+0x4000),%bx
	mov 	$0x4f02,%ax
	int 	$0x10
	movb 	$8,(VMODE)
```

需要ax设置`0x4f02`，bx设置`vbemode+0x4000`，然后调用`int 10`

## VBE检查

部分显卡可能真不支持VBE的情况，我们的代码就有问题了，无法显示屏幕，所以需要做一个自适应检查。

```assembly
	# 确认VBE是否存在
	mov 	$0x9000,%ax
	mov 	%ax,%es
	mov 	$0,%di
	mov 	$0x4f00,%ax
	int 	$0x10
	cmp 	$0x004f,%ax
	jne 	scrn320

	# 检查vbe版本需要是0x200，即VBE 2.0
	mov 	%es:(%di,4),%ax
	cmp 	$0x0200,%ax
	jb  	scrn320

	# 测试画面模式是否能使用，需要ax为0x004f
	mov 	$VBEMODE,%cx
	mov 	$0x4f01,%ax
	int 	$0x10
	cmp 	$0x004f,%ax
	jne  	scrn320

	# 画面的模式 包含了非常多的重要信息，如模式属性
    # WORD  [es:di+0x00]: 模式属性，第7位需要是1，这样才能加上0x4000
    # WORD  [es:di+0x12]: X的分辨率
    # WORD  [es:di+0x14]: Y的分辨率
    # BYTE  [es:di+0x19]: 颜色数，必须为8，我们代码用的就是8
    # BYTE  [es:di+0x1b]: 颜色的指定方法，必须为4，即调色板模式
    # DWORD [es:di+0x28]: VRAM的地址

    # 确认颜色数
	movb 	%es:0x19(%di),%al
	cmp 	$8,%al
	jne 	scrn320
    # 确认颜色指定方法
	movb 	%es:0x1b(%di),%al
	cmp 	$4,%al
	jne 	scrn320

    # 重新调用设置画面模式，需要加上0x4000
    mov 	$(VBEMODE+0x4000),%bx
	mov 	$0x4f02,%ax
	int 	$0x10

	# 记录对应信息给到c语言
	movb 	$8,(VMODE)
	mov 	%es:0x12(%di),%ax
	mov 	%ax,(SCRNX)
	mov 	%es:0x14(%di),%ax
	mov 	%ax,(SCRNY)
	movl 	%es:0x28(%di),%eax
	movl 	%eax,(VRAM)
```
