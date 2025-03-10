---
title: 汇编语言学习笔记
date: 2019-04-25 15:56:41
tags:
categories: [Program, C/C++]
---

# 一、汇编通用

## 1. 语法

```assembly
; 注释
```

# 二、nask汇编

汇编指令来源于《30天自制操作系统》一书中，编译环境为nask，似乎是作者自己写的编译，并不清楚和一般的汇编的区别

## 1. 指令介绍

```assembly
DB      ; 添加1个字节长度数据
DW      ; 添加2个字节长度数据
DD      ; 添加4个字节长度数据
RESB    ; 用0x00填充x字节
$       ; 变量，可指当前位置的字节数
ORG     ; 指定运行时机器语言指令装载内存地址，有这条指令后，$代表内存地址
JMP     ; 相当于C语言goto
MOV     ; 赋值，"MOV AV, 0"代表"AV = 0"，拷贝赋值而非移动，位数必须相同，不然编译出错
[]      ; 代表内存地址
CMP     ; 比较指令
JE      ; "Jump if equal"，比较跳转指令，相等跳转，否则继续执行，和CMP配合使用
JA      ; "Jump if above"，大于跳转，否则继续执行，和CMP配合使用
JB      ; "Jump if below"，小于跳转，否则继续执行，和CMP配合使用
JAE     ; "Jump if above or equal"，大于等于跳转，否则继续执行，和CMP配合使用
JBE     ; "Jump if below or equal"，小于等于跳转，否则继续执行，和CMP配合使用
JC      ; "Jump if carry"，捕获"进位标志"，如果为1则跳转
JNC     ; "Jump if not carry"，捕获"进位标志"，如果为0则跳转
EQU     ; 定义常数，相当于define
HLT     ; CPU停止动作，外部事件唤醒，可能时键盘、鼠标等外部事件
PUSH    ; 将寄存器内容入栈，PUSH EBP
POP     ; 将栈顶内容设置到寄存器内，POP EBP
CLI     ; 禁止CPU中断
STI     ; 使能CPU中断
IN      ; 从外设读取数据
OUT     ; 将数据写入外设
```

**示例**

```assembly
    DB      0xeb, 0x4e, 0x90
    DB      2               ; FAT的个数（必须为2）
    DB      "HELLOIPL"      ; 启动区的名称可以是任意的字符串
    DW      512             ;
    DD      2880            ; 重写一次磁盘大小
    DD      0xffffffff      ; （可能是）卷标号码
    RESB    18              ; 先空出18字节
    RESB    0x1fe-$         ; 填写0x00，直到0x001fe

    ORG     0x7c00          ; 指明程序的装载地址
    JMP     entry

entry:
    MOV     AV, 0           ; 初始化寄存器
    MOV     SS, AX
    MOV     AL, [SI]        ; AL赋值为SI值的地址的值，SI储存为678，则AL赋值为内存中678号地址指向的值
    MOV     BYTE [678], 123 ; 将123按照Byte（8位）储存在内存的678地址中

; MOV使用寄存器赋值只有BX、BP、SI、DI，其他寄存器赋值可以使用下面指令进行
    MOV     BX, DX
    MOV     AL, BYTE [BX]

    CMP     AL, 0
    JE      entry

    CMP     AL, 0
    JA      entry

    CMP     AL, 0
    JB      entry

    CMP     AL, 0
    JAE     entry

    CMP     AL, 0
    JBE     entry

    JC      error           ; FLAGS.CF进位标志为1则跳转
    JNC     fin             ; FLAGS.CF进位标志为0则跳转

CYLS    EQU     10          ; 定义CYLS为10
```

### 1.1. MOV 指令详解

```assembly
MOV         AL, 0           ; AL寄存器值设置成0
MOV         EDX, [ESP+4]    ; 将ESP内部存的地址偏移4，作为地址找到值赋值给EDX
MOV		    [ECX], AL       ; 将AL内部的值，赋值给ECX值作为地址的地方
```

## 2. 实现C语言中的 void write_mem8(char *addr, char data)

```cpp
void write_mem8(char *addr, char data) {
    *addr = data;
}
```

- 手写

```assembly
_write_mem8:	; void write_mem8(char *addr, char data);
    MOV		ECX, [ESP+4]		; [ESP+4]中存放的是addr，读到ECX中
    MOV		AL, [ESP+8]		    ; [ESP+8]存放的是data，读到AL中
    MOV		[ECX], AL
    RET
```

- nask编译出来的

```assembly
	GLOBAL	_write_mem8
_write_mem8:
	PUSH	EBP
	MOV	    EBP, ESP
	MOV	    EAX, DWORD [8+EBP]
	MOV	    EDX, DWORD [12+EBP]
	MOV	    BYTE [EAX],DL
	POP	    EBP
	RET
```

**差异解释**

- ESP、EBP见[寄存器解释](#3-寄存器解释)
- `PUSH EBP`将EBP入栈，保存系统栈状态，但是由于PUSH，导致ESP向上移动了4位
- 对第一种汇编，ESP指向栈顶，`ESP+4`是第一个参数位置
- 对第二种汇编，EBP等于了ESP，但是ESP向上移动了4位，所以`EBP+8`是第一个参数的位置

## 3. 寄存器解释

- ESP: 栈指针寄存器(extended stack pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的栈顶。
- EBP: 基址指针寄存器(extended base pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的底部。
- EAX: 32位的寄存器，累加器(accumulator), 它是很多加法乘法指令的缺省寄存器。
- EBX: 32位的寄存器，基地址(base)寄存器, 在内存寻址时存放基地址。
- ECX: 32位的寄存器，计数器(counter), 是重复(REP)前缀指令和LOOP指令的内定计数器。
- EDX: 32位的寄存器，总是被用来放整数除法产生的余数。
- AX: 16位寄存器，同样也是EAX的低16位，同样有BX, CX, DX
- AH: 8位寄存器，同样是AX的高8位，同样有BH, CH, DH
- AL: 8位寄存器，同样是AX的低8位，同样有BL, CL, DL

# 三、bios内置汇编指令

## 1. 调用显卡（INT 0x10）

### 1.1. 显示字符

- INT为软件中断指令，用于调用BIOS内置函数
- BIOS（Basic input output system）为厂家开发的提供给操作系统开发人员使用的程序，其中含有许多基本程序，使用INT调用
- 0x10函数使用INT调用为显示一个字符函数

```nas
; 显示一个字符，无返回值
    MOV     AH, 0x0e
    MOV     AL, 'a'         ; 显示字符'a'
    MOV     BH, 0
    MOV     BX, 15          ; 指定字符颜色
    INT     0x10            ; 调用显卡BIOS
```

### 1.2. 设置显卡显示模式

- AH=0x00
- AL=模式
  - 0x03: 16色字符模式，80x25
  - 0x12: VGA图形模式，640x480x4位彩色模式，独特的4面储存模式
  - 0x13: VGA图形模式，320x200x8位彩色模式，调色板模式
  - 0x6a: 扩展VGA图形模式，800x600x4位彩色模式，独特的4面储存模式（部分显卡不支持）
- 返回值: 无


## 2. 磁盘读写，扇区校验，寻道函数（INT 0x13)

- AH = 0x02（读盘） / 0x04（写盘） / 0x0c（寻道）
- AL = 处理几个扇区（只能处理连续的扇区）
- CH = 柱面号（0-1位）
- CL = 扇区号（0-5位）|（柱面号&0x300）>> 2
- DH = 磁头号
- DL = 驱动器号
- ES:BX = 缓冲地址（校验和寻道不使用）
- 返回值
  - FLAGS.CF == 0，没有错误，AH == 0
  - FLAGS.CF == 1，有错误，错误号码存于AH（与重置`reset`功能一样）

### 2.1. 复位磁盘

- AH = 0x00
- DL = 0x00
- INT 0x13

# 四、gcc汇编（AT&T汇编）

## 1. 生成汇编代码

```shell
# -S 生成.s结尾的汇编代码文件
# -fverbose-asm 将变量名称作为注释
gcc -S -fverbose-asm xxx.c
```

## 2. 语法

### 2.1. 汇编符号定义

汇编中使用`.`作为指令的前缀，编译器不会将其翻译成及其指令

| 指令 | 含义                                                           |
| ---- | -------------------------------------------------------------- |
| jnz  | jump not zero，上一步结果不为0则跳转，可以配合sub、testb等使用 |
| jne  | jump not equals，配合testb使用                                 |
| je   | jump when equal，配合testb使用                                 |
| jge  | jump when greter or equal，配合cmp使用                         |

#### 1) `.section`

将代码分成多个段，使用`.section`进行划分。程序被操作系统加在执行时，每个段被加载到不同的地址。对不同的地址有不同的读、写、执行权限。

```assembly
.section .data
# 此段用于保存程序的数据，可读可写，相当于全局变量

.section .text
# 此段用于保存代码，只读可执行
```

#### 2) `.global`

汇编程序被编译器处理后，所有符号都变成对应的地址。`.global`是告诉编译器，这个符号要被链接器识别，是一个全局符号。

```assembly
.global _start

_start:
```

`_start`符号代表汇编程序的入口函数，类似与c程序中的main。整个程序中需要有一个被`.global`声明的`_start`符号，会作为整个程序的入口地址

### 2.2. 变量和寄存器

|            | 表示        | 示例                               | 说明                                                            |
| ---------- | ----------- | ---------------------------------- | --------------------------------------------------------------- |
| 寄存器     | `%al`       | `movb $0x13,%al`<br>`movb %al,%ah` | 寄存器需要使用`%`标记，放左边为内部的值，放右边为要写入此寄存器 |
| 内存操作数 | `(%eax)`    | `movb (%esi),%eax`                 | 取地址中的值使用括号                                            |
| 立即数     | `0x200b96`  | `movb $0x13,0x200b96`              | 什么都不加，代表一个内存地址                                    |
| 常量       | `$0x001234` | `movb $0x13,%al`                   | 值常量需要使用`$`开头                                           |

### 2.3. 定义常量宏

```assembly
#define CYLS  0x0ff0
```

### 2.4. movx 赋值函数

movx的操作是将左边赋值给右边

- movb: 赋值1字节
- movw: 按照2字节赋值
- movl: 按照4字节赋值

```assembly
movb 0x13,AL    # 将0x13赋值给寄存器AL
```

### 2.5. inx/outx 输出数值到端口

```assembly
# 将0xd1输出到0x64端口
movb	$0xd1,%al
outb	%al,0x64

# 查看0x64端口是否是0x02
inb     $0x64,%al
testb   $0x2,%al
```

### 2.6. addx 加法

```assembly
addl    $4,%eax     # 将eax加4
```

### 2.7. ljmp 段间跳转

```assembly
# 跳转到1号段的首地址执行
ljmp    $(1*8),0x0000
```

## 3. C语言嵌入汇编

- `volatile`让编译器不优化此段代码

### 3.1. 语法解释

#### 1) 变量修饰符

内联汇编传入函数变量需要对变量添加修饰符，很多指令会要求特定的类型

- `i`: 立即数，表示变量是一个常量，ljmp指令就需要传入常量，这里传入的变量不能是可变的
- `r`: 寄存器，表示变量会放到一个通用寄存器来存储传入，在不同架构代表不同的寄存器
- `q`: 同r，但是只在x86架构下使用，表示eax、ebx、ecx、edx几个寄存器都可以使用
- `a`: 放入eax使用
- `b`: 放入ebx使用
- `c`: 放入ecx使用
- `=r`: 输出寄存器，代表操作会存入寄存器赋值给变量
- `m`: 内存，编译器会生成适当的内存访问指令来访问这个操作数

### 3.2. 几个示例

#### 1) io_load_eflags

```cpp
static inline uint32_t io_load_eflags(void) {
    uint32_t eflags;
    asm volatile("pushfl; popl %0" : "=r" (eflags));
    return eflags;
}
```

```assembly
  280023:	9c                   	pushf
  280024:	5b                   	pop    %ebx
```

#### 2) io_store_eflags

```cpp
static inline void io_store_eflags(uint32_t eflags) {
    asm volatile("pushl %0; popfl" : : "r" (eflags));
}
```

```assembly
  280057:	53                   	push   %ebx
  280058:	9d                   	popf
```

#### 3) 单句指令

```cpp
static inline void io_cli(void) {
    asm volatile("cli");
}
```

# 五、nasm汇编

nasm是x86的最常用的汇编

## 1. 指令介绍

```assembly
DB      ; 添加1个字节长度数据
DW      ; 添加2个字节长度数据
DD      ; 添加4个字节长度数据
RESB    ; 用0x00填充x字节
$       ; 变量，可指当前行的位置
$$      ; 指当前节开始处被汇编后的位置
ORG     ; 指定运行时机器语言指令装载内存地址，有这条指令后，$代表内存地址
JMP     ; 相当于C语言goto
MOV     ; 赋值，"MOV AV, 0"代表"AV = 0"，拷贝赋值而非移动，位数必须相同，不然编译出错
[]      ; 代表内存地址
CMP     ; 比较指令
JE      ; "Jump if equal"，比较跳转指令，相等跳转，否则继续执行，和CMP配合使用
JA      ; "Jump if above"，大于跳转，否则继续执行，和CMP配合使用
JB      ; "Jump if below"，小于跳转，否则继续执行，和CMP配合使用
JAE     ; "Jump if above or equal"，大于等于跳转，否则继续执行，和CMP配合使用
JBE     ; "Jump if below or equal"，小于等于跳转，否则继续执行，和CMP配合使用
JC      ; "Jump if carry"，捕获"进位标志"，如果为1则跳转
JNC     ; "Jump if not carry"，捕获"进位标志"，如果为0则跳转
EQU     ; 定义常数，相当于define
HLT     ; CPU停止动作，外部事件唤醒，可能时键盘、鼠标等外部事件
PUSH    ; 将寄存器内容入栈，PUSH EBP
POP     ; 将栈顶内容设置到寄存器内，POP EBP
CLI     ; 禁止CPU中断
STI     ; 使能CPU中断
IN      ; 从外设读取数据
OUT     ; 将数据写入外设
```
