---
title: 汇编语言学习笔记
date: 2019-04-25 15:56:41
tags:
categories: [Program, C/C++]
---

# nask汇编

汇编指令来源于《30天自制操作系统》一书中，编译环境为nask，似乎是作者自己写的编译，并不清楚和一般的汇编的区别

## 指令介绍

```nas
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
    HLT     ; CPU停止动作，外部事件唤醒
```

示例

```nas
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
