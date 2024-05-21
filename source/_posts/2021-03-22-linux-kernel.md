---
title: linux kernel源码学习记录
date: 2021-03-22 16:55:36
tags: [Linux]
categories: [Program, C/C++]
---

# 一、前言

本文为研究linux kernel源码所记录的一些笔记

- 源码下载路径: https://mirrors.edge.kernel.org/pub/linux/kernel/
- 源码git路径: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git

由于篇幅问题，后续更新放在 [linux内核源码分析记录](/bookPages/docs/linux-kernel/)

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

## 1. 公共机制

### 1.1. `container_of` 根据数据结构节点找value

[container_of](/bookPages/docs/linux-kernel/data-structures/container_of/)

## 2. rbtree

[rbtree](/bookPages/docs/linux-kernel/data-structures/rbtree/)

## 3. rcu 读拷贝更新

[rcu](/bookPages/docs/linux-kernel/data-structures/rcu/)

# 四、系统调用

## 1. 网络相关

### 1.1. epoll

[epoll](/bookPages/docs/linux-kernel/net/epoll/)

### 1.2. bind 绑定地址到socket

#### 1) 接口定义

```cpp
// net/socket.c
SYSCALL_DEFINE3(bind, int, fd, struct sockaddr __user *, umyaddr, int, addrlen)
{
	return __sys_bind(fd, umyaddr, addrlen);
}
```

### 1.3. unix套接字

源码主要看`net/unix/af_unix.c`

- unix套接字仅支持`SOCK_STREAM`、`SOCK_RAW`、`SOCK_DGRAM`、`SOCK_SEQPACKET`这几种type，源码看`unix_create()`
- 使用unix套接字，protocol参数必须为`0`，原因查看`unix_create()`的第一个判断

## 2. 文件相关

### 2.1. 监听文件变化 inotify

[inotify](/bookPages/docs/linux-kernel/fs/inotify/)

# 五、底层的几个机制

## 1. 惊群现象和处理

参考 [深入浅出 Linux 惊群：现象、原因和解决方案](https://zhuanlan.zhihu.com/p/385410196)

- 在linux的2.6.x已经解决，底层仅会唤起一个进程进行处理

# 六、编译内核

## 1. 编译过程

### 1.1. 配置

### 1.2. 编译

### 1.3. 安装

#### 1) 安装

#### 2) 安装模块

```shell
# INSTALL_MOD_PATH指定安装的根目录位置，会自动安装到此目录下的lib/modules/<arch>下
make modules_install INSTALL_MOD_PATH=/home/test/vmware/linux-5.19/fs
```

## 2. 选项解释

### 2.1. 基础知识

#### 1) 模块编译选项

- 内核编译选项中，前面是`<*>`代表编译进内核
- `<M>`代表编译成模块

#### 2) 部分配置解释

```conf
# 内核开启kasan选项，有内存异常检测到会写到dmesg中
CONFIG_KASAN=y
# 编译出/usr/lib/modules/$(uname -r)/kernel/lib/test_kasan.ko，insmod后会在dmesg中打印出来结果说明kasan生效
CONFIG_TEST_KASAN=y
```

# 七、调试内核

参考 [QEMU调试Linux内核环境搭建](https://zhuanlan.zhihu.com/p/499637419)

## 1. 编译内核

- 下载内核源码`https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.19.1.tar.gz`
- 解压后进入目录

```shell
# 创建默认配置
make x86_64_defconfig
# 进入menu配置模式
make menuconfig
```

- 修改下述配置，开启debug，关闭地址随机化

```
Kernel hacking  --->
    [*] Kernel debugging
    Compile-time checks and compiler options  --->
        [*] Compile the kernel with debug info
        [*]   Provide GDB scripts for kernel debuggin


Processor type and features ---->
    [] Randomize the address of the kernel image (KASLR)
```

- 编译

```shell
make -j 20
```

### 1.1. 想要某个函数不优化

- 给单个函数添加`__attribute__((optimize("O0")))`
- 如，不优化`static int ip_rcv_finish(struct net *net, struct sock *sk, struct sk_buff *skb)`就写成下面这样

```cpp
static int ip_rcv_finish(struct net *net, struct sock *sk, struct sk_buff *skb) __attribute__((optimize("O0")));

int ip_rcv_finish(struct net *net, struct sock *sk, struct sk_buff *skb) {
    ...
}
```

## 2. 构建根文件系统

- 这里选择ubuntu的根文件系统 `http://cdimage.ubuntu.com/ubuntu-base/releases/22.04/release/ubuntu-base-22.04-base-amd64.tar.gz`
- 创建镜像并mount

```shell
# 创建10G根镜像文件
fallocate -l 10G rootfs.img
# 格式化为ext4
mkfs.ext4 rootfs.img
# 挂载到一个目录下
sudo mount -t ext4 -o loop rootfs.img ./fs
# 解压根文件系统到目录下
sudo tar -xzvf ubuntu-base-22.04-base-amd64.tar.gz -C ./fs
# 修改resolv.conf
cp /etc/resolv.conf ./fs/etc/resolv.conf
# 修改镜像源
vim ./fs/etc/apt/sources.list
# 修改权限
sudo chown -R root:root ./fs
sudo chmod -R 777 ./fs
# 挂载dev到目录下（不然无法操作/dev/null）
sudo mount -o bind /dev fs/dev
# chroot上去
sudo chroot ./fs
# 更新软件源
apt update
# 安装必要的软件
apt install vim gcc tmux g++ make cmake openssh-server wireless-tools
apt install net-tools     # ifconfig
apt install iputils-ping  # ping
apt install ifupdown      # ip
apt install network-manager
# 开机启动NetworkManager
systemctl enable NetworkManager
# 修改root密码
passwd
# 退出后umount
exit
sudo umount ./fs/dev
sudo umount ./fs
```

## 3. 起系统

- `-s`: 相当于`-gdb tcp::1234`，在1234启用gdb调试
- `-append "root=/dev/sda rw console=ttyS0"`: root使用sda，要rw否则会只读；console设置输出到当前控制台
- 使用`spice://127.0.0.1:5900`可以看到画面

```shell
qemu-system-x86_64 -enable-kvm -m 4G -smp 1 -kernel /path/to/kernel/source/arch/x86_64/boot/bzImage -hda rootfs.img -drive format=raw -append "root=/dev/sda rw console=ttyS0" -nographic -s -spice port=5900,disable-ticketing=on
```

- 配置网络，因为没有设置网卡，所以使用的是qemu自己模拟的用户态网络，配置网络dhcp获取即可

```shell
# 查看连接
=> nmcli conn show
NAME    UUID                                  TYPE      DEVICE
enp0s3  9a364675-b60a-479a-8d4a-754bab3dfe01  ethernet  enp0s3
# 配置dhcp获取ip
=> nmcli conn add type ethernet con-name enp0s3-dhcp ifname enp0s3 ipv4.method auto
# 删除连接
=> nmcli conn delete enp0s3
# 查看连接
=> nmcli conn show
NAME          UUID                                  TYPE      DEVICE
enp0s3-dhcp   9a364675-b60a-479a-8d4a-754bab3dfe01  ethernet  --
# 启用连接
=> nmcli conn up enp0s3-dhcp
# 查看连接
=> nmcli conn show
NAME          UUID                                  TYPE      DEVICE
enp0s3-dhcp   9a364675-b60a-479a-8d4a-754bab3dfe01  ethernet  enp0s3
```

- 共享磁盘在`/dev/sdb`，自己看着挂载就好了

## 4. gdb调试

### 4.1. 命令行调试

- 到linux内核编译的目录下

```shell
=> gdb vmlinux
...
(gdb) target remote localhost:1234
Remote debugging using localhost:1234
```

### 4.2. vscode调试

- vscode打开源码目录
- 配置`launch.json`

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/vmlinux",
            "MIMode": "gdb",
            "miDebuggerServerAddress": "127.0.0.1:1234",
            "cwd": "${workspaceFolder}",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "Set Disassembly Flavor to Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

# 八、crash调试vmcore排查宕机问题

[宕机问题排查记录](/bookPages/docs/linux/kernel-troubleshooting/crash/)

## 1. crash命令使用

### 1.1. 查看系统信息

**系统信息**

```shell
crash> sys
      KERNEL: vmlinux/4.19.181/since-20220607/vmlinux
    DUMPFILE: /backup/vmcore-check/kdump/127.0.0.1-2024-02-19-18:04:10/vmcore  [PARTIAL DUMP]
        CPUS: 4
        DATE: Mon Feb 19 18:02:59 2024
      UPTIME: 104 days, 01:01:03
LOAD AVERAGE: 0.11, 0.19, 0.23
       TASKS: 1192
    NODENAME: test-kernel
     RELEASE: 4.19.181
     VERSION: #6 SMP Thu Jun 16 04:03:14 CST 2022
     MACHINE: x86_64  (2200 Mhz)
      MEMORY: 8 GB
       PANIC: ""
```

**dmesg**

```shell
crash> log | tail -n 10
[2729253.365084] R10: 00007fecfc95fb80 R11: 0000000000000293 R12: 0000000000000040
[2729253.365085] R13: 00007fecbbb56970 R14: 00007fecbbb56968 R15: 00007fecfc95fa30
[2729253.365087] Modules linked in: ip_vs_wrr ip_vs xt_TEE nf_dup_ipv6 nf_dup_ipv4 xt_TPROXY nf_tproxy_ipv6 nf_tproxy_ipv4 xt_mark xt_socket nf_socket_ipv4 nf_socket_ipv6 veth ipt_MASQUERADE nf_conntrack_netlink xt_addrtype br_netfilter bridge stp llc rfkill overlay xt_connmark xt_nat xt_comment xt_iprange ip6t_REJECT nf_reject_ipv6 ip6t_rpfilter ipt_REJECT nf_reject_ipv4 xt_conntrack nft_counter nft_chain_nat_ipv6 nf_nat_ipv6 nft_chain_route_ipv6 nft_chain_nat_ipv4 nf_nat_ipv4 nf_nat nft_chain_route_ipv4 nfnetlink_queue nf_conntrack nf_defrag_ipv6 nf_defrag_ipv4 ip6_tables nft_compat ip_set_hash_ipport ip_set_hash_ip ip_set nf_tables nfnetlink dm_crypt loop intel_rapl ext4 mbcache jbd2 x86_pkg_temp_thermal intel_powerclamp coretemp kvm_intel kvm irqbypass crct10dif_pclmul crc32_pclmul
[2729253.388462] traps: test_proxy[3608] trap invalid opcode ip:7f83806ab82e sp:7fff360d16e8 error:0 in libc-2.28.so[7f838064a000+1ba000]
[2729253.575710] traps: testfwd[27992] trap invalid opcode ip:7fe56ab0c82e sp:7ffda76b06f8 error:0 in libc-2.28.so[7fe56aaab000+1ba000]
[2729253.593401] traps: test_proxy[3609] trap invalid opcode ip:7f83806ab82e sp:7fff360d16e8 error:0 in libc-2.28.so[7f838064a000+1ba000]
[2729253.600250] test-console[16276]: segfault at 99193855eb8 ip 0000000000f7d670 sp 00007ffe7dbac1e8 error 4 in test-console[400000+26b6000]
[2729253.600264] Code: 74 35 4d 89 cb 89 c7 44 29 c7 89 fa c1 ea 1f 01 fa d1 fa 44 01 c2 44 8d 6a 01 43 8d 7c 2d 02 c1 e7 03 48 63 ff 49 8b 7c 3b ff <44> 39 57 07 72 ca 89 d0 44 39 c0 75 cb 39 d8 7f 67 8d 50 02 c1 e2
[2729253.656410]  ghash_clmulni_intel intel_cstate intel_rapl_perf sg pcspkr wdat_wdt i2c_i801 i2c_ismt ip_tables xfs libcrc32c sd_mod ahci libahci qat_c3xxx(OE) ixgbe(O) crc32c_intel libata igb i2c_algo_bit dca intel_qat(OE) uio dm_mirror dm_region_hash dm_log dm_mod fuse
[2729253.656425] CR2: 0000000000100218
```

**内存情况**

```shell
crash> kmem -i
                 PAGES        TOTAL      PERCENTAGE
    TOTAL MEM  4054860      15.5 GB         ----
         FREE   599514       2.3 GB   14% of TOTAL MEM
         USED  3455346      13.2 GB   85% of TOTAL MEM
       SHARED   204346     798.2 MB    5% of TOTAL MEM
      BUFFERS     1262       4.9 MB    0% of TOTAL MEM
       CACHED  1573638         6 GB   38% of TOTAL MEM
         SLAB   148297     579.3 MB    3% of TOTAL MEM

   TOTAL HUGE        0            0         ----
    HUGE FREE        0            0    0% of TOTAL HUGE

   TOTAL SWAP  1048575         4 GB         ----
    SWAP USED    11968      46.8 MB    1% of TOTAL SWAP
    SWAP FREE  1036607         4 GB   98% of TOTAL SWAP

 COMMIT LIMIT  3076005      11.7 GB         ----
    COMMITTED  4663520      17.8 GB  151% of TOTAL LIMIT
```

**runq 查看各个cpu正在运行的线程和时间**

```shell
crash> runq -m
 CPU 0: [31 14:07:33.656]  PID: 0      TASK: ffffffff93a12740  COMMAND: "swapper/0"
 CPU 1: [ 0 00:00:00.000]  PID: 3608   TASK: ffff8f6ba749ad00  COMMAND: "proxy"
 CPU 2: [ 0 00:00:00.063]  PID: 3609   TASK: ffff8f6ba749da00  COMMAND: "proxy"
 CPU 3: [ 0 00:00:00.875]  PID: 5102   TASK: ffff8f6cd27b0000  COMMAND: "mysqld"
```

**中断状态**

```shell
crash> irq -s
           CPU0       CPU1       CPU2       CPU3
  0:         27          0          0          0  IO-APIC-edge     timer
  4:       2251          0      15122        454  IO-APIC-edge     ttyS0
  8:          0          1          0          0  IO-APIC-edge     rtc0
  9:          0          0          0          0  IO-APIC-fasteoi  acpi
 23:          0          0          0          0  IO-APIC-fasteoi  i801_smbus
 24:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv
 25:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv,pcie-dpc
 26:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv,pcie-dpc
 27:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv,pcie-dpc
 28:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv,pcie-dpc
 29:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv,pcie-dpc
 30:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv,pcie-dpc
 31:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv,pcie-dpc
 32:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv
 33:          0          0          0          0  PCI-MSI-edge     PCIe PME,aerdrv
 34:          0          0          0          0  PCI-MSI-edge     xhci_hcd
 35:          0          0          0          0  PCI-MSI-edge     eth4
 36:     214426     285544     479106     385420  PCI-MSI-edge     eth4-rx-0
 37:     211977     287662     478597     386260  PCI-MSI-edge     eth4-rx-1
 38:     225305     282160     482065     374966  PCI-MSI-edge     eth4-tx-0
 39:     225282     282674     480828     375712  PCI-MSI-edge     eth4-tx-1
 ...
```

### 1.2. bt 查看堆栈

- `-a`: 展示每个cpu的正在运行的进程的堆栈
- `-c cpu`: 展示特定cpu的堆栈，示例 "3"、"1,8,9"、"1-23"或"1,8,9-14"
- `-l`: 显示对应的代码所在文件和行
- `pid`: 直接跟pid可以查看对应进程的栈

### 1.3. struct 查看结构体

- `-o`: 展示结构体偏移

```shell
# 对照地址查看结构体对应的值
crash> struct rq
struct rq {
    raw_spinlock_t lock;
    unsigned int nr_running;
    unsigned int nr_numa_running;
    unsigned int nr_preferred_running;
    unsigned int numa_migrate_on;
    unsigned long cpu_load[5];
    unsigned long last_load_update_tick;
    unsigned long last_blocked_load_update_tick;
    unsigned int has_blocked_load;
    unsigned int nohz_tick_stopped;
    atomic_t nohz_flags;
...

# 指定地址查看结构体的值
crash> struct rq ffff9ca577aa1ec0
struct rq {
  lock = {
    raw_lock = {
      {
        val = {
          counter = 524545
        },
        {
          locked = 1 '\001',
          pending = 1 '\001'
        },
        {
          locked_pending = 257,
          tail = 8
        }
      }
    }
  },
  nr_running = 0,
  nr_numa_running = 0,
  nr_preferred_running = 0,
  numa_migrate_on = 0,
...

# 展示结构体变量偏移
crash> struct -o rq
struct rq {
     [0] raw_spinlock_t lock;
     [4] unsigned int nr_running;
     [8] unsigned int nr_numa_running;
    [12] unsigned int nr_preferred_running;
    [16] unsigned int numa_migrate_on;
    [24] unsigned long cpu_load[5];
    [64] unsigned long last_load_update_tick;
    [72] unsigned long last_blocked_load_update_tick;
    [80] unsigned int has_blocked_load;
    [84] unsigned int nohz_tick_stopped;
    [88] atomic_t nohz_flags;
    [96] struct load_weight load;
   [112] unsigned long nr_load_updates;
   [120] u64 nr_switches;
   [128] struct cfs_rq cfs;
...
```

#### 1) 查看特定地址的结构体的某些成员变量的值

```shell
crash> struct rq.lock,nr_running,cpu_load ffff8f6d6faa1ec0
  lock = {
    raw_lock = {
      {
        val = {
          counter = 0
        },
        {
          locked = 0 '\000',
          pending = 0 '\000'
        },
        {
          locked_pending = 0,
          tail = 0
        }
      }
    }
  }
  nr_running = 1
  cpu_load = {36, 35, 129, 618, 1370}
```

#### 2) 几个常见的全局变量

**struct rq 每个cpu一个**

```shell
crash> struct rq.lock runqueues:0,1
[0]: ffff8f6d6fa21ec0
  lock = {
    raw_lock = {
      {
        val = {
          counter = 1
        },
        {
          locked = 1 '\001',
          pending = 0 '\000'
        },
        {
          locked_pending = 1,
          tail = 0
        }
      }
    }
  }
[1]: ffff8f6d6faa1ec0
  lock = {
    raw_lock = {
      {
        val = {
          counter = 0
        },
        {
          locked = 0 '\000',
          pending = 0 '\000'
        },
        {
          locked_pending = 0,
          tail = 0
        }
      }
    }
  }
```

### 1.4. set 设置进程上下文

- `-p`: 设置context到panic的task

```shell
crash> set -p
    PID: 5102
COMMAND: "mysqld"
   TASK: ffff8f6cd27b0000  [THREAD_INFO: ffff8f6cd27b0000]   # 此地址是task_struct的地址
    CPU: 3
  STATE:  (PANIC)
```

### 1.5. task 查看进程的task_struct和thread_info信息

```shell
crash> task [pid]
```
