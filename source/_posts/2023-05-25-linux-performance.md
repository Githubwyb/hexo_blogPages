---
title: linu性能观测和排查
date: 2023-05-25 11:54:54
tags: [Linux]
categories: [Program, Shell]
---

# 一、性能观测工具

## 1. pidstat 进程状态观测

### 1.1. 选项解释

**筛选**

- `-p <pid>`: 进程id号，不指定就显示所有的进程

**信息统计**

- `-r`: 显示内存使用情况和页错误
- `-u`: 显示cpu使用情况
- `-d`: 显示io使用情况
- `-s`: 展示栈使用情况，kB为单位，保留和使用的大小
- `-R`: 展示实时优先级和调度策略信息
- `-v`: 展示一些内核表的值，线程数和fd分配
- `-w`: 展示任务调度情况，每秒自愿调度的上下文切换（资源被占用导致阻塞）和非自愿调度上下文切换（在自己时间片内被强制放弃cpu）

**显示相关**

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

### 1.3. 进程io写入情况统计

```shell
# 统计进程io使用情况，5秒输出一次，一共输出1次
=> pidstat -p 1687 -d 5 1
Linux 4.19.181 (centos)         2024年07月03日  _x86_64_        (16 CPU)

10:23:20 AM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
10:23:20 AM     0      1687      0.00   1120.00      0.00     498  sftp-server
Average:        0      1687      0.00   1120.00      0.00     498  sftp-server
```

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

## 3. sar 系统活动信息收集

### 3.1. 选项

```shell
Usage: sar [ options ] [ <interval> [ <count> ] ]
Main options and reports (report name between square brackets):
        -B      Paging statistics [A_PAGE]
        -b      I/O and transfer rate statistics [A_IO]
        -d      Block devices statistics [A_DISK]
        -F [ MOUNT ]
                Filesystems statistics [A_FS]
        -H      Hugepages utilization statistics [A_HUGE]
        -I [ SUM | ALL ]
                Interrupts statistics [A_IRQ]
        -m { <keyword> [,...] | ALL }
                Power management statistics [A_PWR_...]
                Keywords are:
                BAT     Batteries capacity
                CPU     CPU instantaneous clock frequency
                FAN     Fans speed
                FREQ    CPU average clock frequency
                IN      Voltage inputs
                TEMP    Devices temperature
                USB     USB devices plugged into the system
        -n { <keyword> [,...] | ALL }
                Network statistics [A_NET_...]
                Keywords are:
                DEV     Network interfaces
                EDEV    Network interfaces (errors)
                NFS     NFS client
                NFSD    NFS server
                SOCK    Sockets (v4)
                IP      IP traffic      (v4)
                EIP     IP traffic      (v4) (errors)
                ICMP    ICMP traffic    (v4)
                EICMP   ICMP traffic    (v4) (errors)
                TCP     TCP traffic     (v4)
                ETCP    TCP traffic     (v4) (errors)
                UDP     UDP traffic     (v4)
                SOCK6   Sockets (v6)
                IP6     IP traffic      (v6)
                EIP6    IP traffic      (v6) (errors)
                ICMP6   ICMP traffic    (v6)
                EICMP6  ICMP traffic    (v6) (errors)
                UDP6    UDP traffic     (v6)
                FC      Fibre channel HBAs
                SOFT    Software-based network processing
        -q [ <keyword> [,...] | PSI | ALL ]
                System load and pressure-stall statistics
                Keywords are:
                LOAD    Queue length and load average statistics [A_QUEUE]
                CPU     Pressure-stall CPU statistics [A_PSI_CPU]
                IO      Pressure-stall I/O statistics [A_PSI_IO]
                MEM     Pressure-stall memory statistics [A_PSI_MEM]
        -r [ ALL ]
                Memory utilization statistics [A_MEMORY]
        -S      Swap space utilization statistics [A_MEMORY]
        -u [ ALL ]
                CPU utilization statistics [A_CPU]
        -v      Kernel tables statistics [A_KTABLES]
        -W      Swapping statistics [A_SWAP]
        -w      Task creation and system switching statistics [A_PCSW]
        -y      TTY devices statistics [A_SERIAL]
```

### 3.2. 网络流速统计

```shell
# 采集网卡上网络流速，5s输出一次，共输出1次
=> sar -n DEV 5 1
Linux 6.6.15-amd64 (centos)   07/03/2024      _x86_64_        (16 CPU)

10:15:49 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
10:15:54 AM        lo      3.20      3.20      0.24      0.24      0.00      0.00      0.00      0.00
10:15:54 AM      eth0    160.00     86.20     24.31   2306.26      0.00      0.00      0.00      0.00

Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
Average:           lo      3.20      3.20      0.24      0.24      0.00      0.00      0.00      0.00
Average:         eth0    160.00     86.20     24.31   2306.26      0.00      0.00      0.00      0.00

# 只采集两个网卡
=> sar -n DEV --iface=ens18,lo 5 1
Linux 6.5.0-28-generic (ubuntu-101)     2024年07月08日  _x86_64_        (16 CPU)

10:15:54 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
10:15:54 AM        lo      1.60      1.60      0.12      0.12      0.00      0.00      0.00      0.00
10:15:54 AM     ens18    130.20     31.20     23.31      3.34      0.00      0.00      0.00      0.00

Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
Average:           lo      1.60      1.60      0.12      0.12      0.00      0.00      0.00      0.00
Average:        ens18    130.20     31.20     23.31      3.34      0.00      0.00      0.00      0.00
```

# 二、内核追踪调试技术

## 2. trace-cmd

https://zhuanlan.zhihu.com/p/417204367

### 2.1. 常用的用法

```shell
# 查看可以追踪什么点
trace-cmd list
# 查看可以使用什么追踪器
trace-cmd list -t
```
