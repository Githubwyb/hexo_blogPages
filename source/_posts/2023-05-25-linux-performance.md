---
title: linu性能观测和排查
date: 2023-05-25 11:54:54
tags: [Linux]
categories: [Program, Shell]
---

# 一、性能观测工具

## 1. pidstat 进程状态观测

### 1.1. 选项解释

- `-p <pid>`: 进程id号，不指定就显示所有的进程
- `-r`: 显示内存使用情况和页错误
- `-u`: 显示cpu使用情况
- `-d`: 显示io使用情况
- `-h`: 指定多个统计后，将统计结果显示在一行里面
- `-H`: 使用时间戳显示时间，默认时间显示的是`03:22:38 PM`

### 1.2. 内存观测

```shell
# 观测pid 1584925的内存情况，每秒输出一次
=> pidstat -p 1584925 -r 1
Linux 6.3.1-arch1-1 (arch-work-pc)      05/25/2023      _x86_64_        (16 CPU)

03:25:26 PM   UID       PID  minflt/s  majflt/s     VSZ     RSS   %MEM  Command
03:25:27 PM     0   1584925     11.00      0.00 6572528 3098804   9.54  wireshark
03:25:28 PM     0   1584925      9.00      0.00 6572528 3098932   9.54  wireshark
03:25:29 PM     0   1584925     11.00      0.00 6572528 3098932   9.54  wireshark
03:25:30 PM     0   1584925     19.00      0.00 6572704 3098932   9.54  wireshark
03:25:31 PM     0   1584925     17.00      0.00 6572704 3099060   9.54  wireshark
```

- `minflt/s`: 任务每秒发生的小故障总数，这些小故障不需要从磁盘加载内存页。
- `majflt/s`: 任务每秒发生的主要故障总数，这些故障需要从磁盘加载内存页。
- `VSZ`: 虚拟大小（Virtual Size），整个任务的虚拟内存使用情况，单位为KByte。
- `RSS`: 常驻集大小（Resident Set Size），任务使用的未交换的物理内存，单位为KByte。

#### RSS/PSS/VSZ

一个进程有500K的代码并且链接了2500K的共享库，然后有200K的堆栈分配。其中有400K自身的代码、1000K的共享库以100K的堆栈内存被加载在实际内存（RAM）中，并且系统中一共有两个进程用了同样的共享库。那么：

VSZ：500K + 2500K + 200K = 3200K
RSS：400K + 1000K + 100K = 1500K
PSS：400K + (1000K / 2) + 100K = 1000K

## 2. top 类似任务管理器

### 2.1. 选项解释

- `-p <pid>`: 只显示某个进程，不指定就显示所有进程

### 2.2. 几个快捷键

- `Shift + E`: 调整内存单位
- `1`: 切换cpu统计模式，所有/详细
- `Shift + P`: cpu占用排序
- `Shift + M`: 内存占用排序
- `m`: 切换内存显示样式
- `c`: 显示进程详细命令

### 2.3. 显示内容解释

```shell
top - 16:01:03 up 1 day, 16:45, 13 users,  load average: 2.45, 2.09, 2.23
Tasks: 499 total,   2 running, 496 sleeping,   0 stopped,   1 zombie
%Cpu(s):  5.7 us,  1.4 sy,  0.0 ni, 92.4 id,  0.0 wa,  0.3 hi,  0.2 si,  0.0 st
GiB Mem :     31.0 total,      2.0 free,     14.8 used,     14.1 buff/cache
GiB Swap:     12.0 total,     12.0 free,      0.0 used.     12.6 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    569 root      20   0 2770000 247324  98132 S  45.0   0.8 169:11.50 /usr/lib/Xorg -nolisten tcp -background none -seat seat0 vt1 -auth /var/run/sddm/{9049d9af-27e6-413f-ac66-bba929904f67} -noreset -displayfd 17
```

#### 1) 参数含义

- `PR`: 进程的优先级，数值越小表示优先级越高。
- `NI`: 进程的nice值，数值越小表示优先级越高。
- `VIRT`: 进程使用的虚拟内存大小，包括代码、数据和共享库等。单位KB
- `RES`: 进程使用的物理内存大小，不包括共享库等。单位KB
- `SHR`: 进程使用的共享内存大小。单位KB或MB，取决于数值大小
- `TIME+`: 进程使用的CPU时间，包括用户态和内核态的时间。

#### 2) S 进程状态

- `R`: Runnable（运行）: 正在运行或在运行队列中等待
- `S`: sleeping（中断）: 休眠中，受阻，在等待某个条件的形成或接收到信号
- `D`: uninterruptible sleep(不可中断): 收到信号不唤醒和不可运行，进程必须等待直到有中断发生
- `Z`: zombie（僵死）: 进程已终止，但进程描述还在，直到父进程调用wait4()系统调用后释放
- `T`: traced or stoppd(停止): 进程收到SIGSTOP,SIGSTP,SIGTOU信号后停止运行

**后缀表示**

- `<`: 优先级高的进程
- `N`: 优先级低的进程
- `L`: 有些页被锁进内存
- `s`: 进程的领导者（在它之下有子进程）
- `l`: ismulti-threaded (using CLONE_THREAD, like NPTL pthreads do)
- `+`: 位于后台的进程组

# 二、内核追踪调试技术



## 2. trace-cmd

### 2.1. 常用的用法

```shell
# 查看可以追踪什么点
trace-cmd list
# 查看可以使用什么追踪器
trace-cmd list -t
```
