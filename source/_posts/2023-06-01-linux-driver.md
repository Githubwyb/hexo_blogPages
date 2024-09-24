---
title: linux驱动开发
date: 2023-06-01 19:18:22
tags: [Linux]
categories: [Program, C/C++]
---

# 前言

# 一、环境搭建

# 二、开发指导

## 1. 基本模板

### 1.1. Makefile编写

- Makefile第一个M要大写，不然也会报错

```makefile
NAME := hello

obj-m := $(NAME).o

ifeq ($(KERNDIR), )
KDIR := /lib/modules/$(shell uname -r)/build
else
KDIR := $(KERNDIR)
endif
PWD := $(shell pwd)

$(NAME)-obj += hello_main.o

# $(NAME)-y +=
# $(NAME)-n +=

ifeq ($(DEBUG), 1)
ccflags-y += -g -O0
endif

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean :
	$(MAKE) -C $(KDIR) M=$(PWD) modules clean
```

#### 1) 命令解释

- linux驱动编译最核心的命令是`make -C <kernel-source-dir> M=$(PWD) modules`
- `-C <dir>`: 指定内核源码目录，里面存放最高级的makefile
- `M=<dir>`: 代表我们模块的makefile在这里

#### 2) Makefile内容解释

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

## 2. 传参

- 源文件中这样写可以传参

```cpp
static int mode = 0;                    // 这里定义默认值
module_param(mode, int, S_IRUGO);       // 第三个参数是权限，定义在include/linux/stat.h，这里是0444
```

**传参示例**

```shell
# 驱动传参必须指定参数名称
insmod ./aaa.ko mode=1
```

**权限的作用**

- 所有挂载的驱动，在`/sys/module/<mod_name>/parameters/`下以参数名命名

```shell
=> ls -l /sys/module/test/parameters/
total 0
-r--r--r-- 1 admin root 4096 Mar 26 11:21 mode
=> cat /sys/module/test/parameters/mode
1
```

- 权限设置为可写的话，其实可以使用vi进行修改此选项。但是内核没有方式通知驱动此参数值发生了变化，所以要么自己写机制保障，要么就权限设置为只读

# 三、小工具代码

## 1. 日志

```cpp
#include <linux/printk.h>

#define tag "xxx"

#define xxx_log_info(fmt, ...) \
    pr_info("[%s] "fmt, tag, ##__VA_ARGS__)
```

## 2. 编译的内核版本输出

- 这个只输出针对编译内核的版本

```cpp
#include <linux/version.h>

static char *get_kernel_version(void) {
    static char buf[16] = {0};
    short a, b, c;

    if (buf[0] != 0) return buf;

    a = LINUX_VERSION_CODE >> 16;
    b = (LINUX_VERSION_CODE >> 8) & 0xff;
    c = LINUX_VERSION_CODE & 0xff;
    sprintf(buf, "%d.%d.%d", a, b, c);
    return buf;
}
```

## 3. 添加git commitid

makefile中对于编译添加选项

```makefile
ifeq ($(GIT_COMMITID), )
GIT_COMMITID := $(shell git rev-parse HEAD)
endif

ccflags-y += -DGIT_COMMITID=\"$(GIT_COMMITID)\"
```

代码中定义到comment里面

```cpp
#ifndef GIT_COMMITID
#define GIT_COMMITID "unknown"
#endif

// 显示在modinfo xxx.ko中
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("xxxx, Git: "GIT_COMMITID);
```

# 四、实战示例

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

### 1.2. 添加setsockopt选项

```cpp
/* toa insert socket options */
enum {
    SOCKET_OPT_BASE = 0x248,
    /* set */
    SOCKET_OPT_TOA_SET_IP = SOCKET_OPT_BASE,
    SOCKET_OPT_MAX,
};

static int toa_set_ip_options(struct sock *sk, int cmd, void __user *user, unsigned int len) {
    int ret = 0;
    struct sockaddr_storage address;
    static const char *tag = __FUNCTION__;

    switch (cmd) {
        case SOCKET_OPT_TOA_SET_IP:
            if (!sk || NULL == user || len < 2 || len > sizeof(struct sockaddr_storage)) {
                LOG_ERR("%s, param is invalid", tag);
                return -EINVAL;
            }

            if (sk->sk_user_data) {
                LOG_ERR("%s, sk_user_data is not NULL", tag);
                return -EPERM;
            }

            ret = copy_from_user(&address, user, len);
            if (ret < 0) {
                LOG_ERR("%s, copy addr from userspace failed, ret %d", tag, ret);
                return ret;
            }

            switch (address.ss_family) {
                case PF_INET: {
                    struct sockaddr_in *vip_addr = (struct sockaddr_in *)&address;
                    sock_set_flag(sk, SOCK_TOAV4_SET);
                    // 存放高32位，低2位被用于当标记位使用
                    sk->sk_user_data = (void *)((u64)(vip_addr->sin_addr.s_addr) << 32);
                    INC_ATOMIC(s_setsockopt_count);
                } break;
                case PF_INET6: {
                    toa_ipv6_node_t *node = NULL;
                    struct sockaddr_in6 *vip_addr = (struct sockaddr_in6 *)&address;

                    node = kmem_cache_zalloc(toa_ipv6_node_slab, GFP_KERNEL);
                    if (!node) {
                        LOG_ERR("%s, kmalloc toa_ipv6_node_t failed", tag);
                        return -ENOMEM;
                    }

                    node->sk = sk;
                    node->created_at = jiffies;
                    memcpy(node->ipv6, vip_addr->sin6_addr.s6_addr, sizeof(node->ipv6));
                    toa_ipv6_insert_node(node);

                    sock_set_flag(sk, SOCK_TOAV6_SET);
                    INC_ATOMIC(s_setsockoptv6_count);
                } break;
                default:
                    LOG_ERR("%s, family %d is not support", tag, address.ss_family);
                    return -EOPNOTSUPP;
            }
            break;
        default:
            LOG_ERR("%s cmd %d not support", tag, cmd);
            return -EOPNOTSUPP;
    }

    return 0;
}

// 应用层设置toa标记位，使用setsockopt接口进行设置
static struct nf_sockopt_ops toa_insert_sockopts = {
    .pf = PF_INET,
    .owner = THIS_MODULE,
    /* set */
    .set_optmin = TOA_INSERT_BASE_CTL,          // 接受的最小命令
    .set_optmax = TOA_INSERT_SO_SET_MAX + 1,    // 接受的最大命令减一
    .set = toa_set_ip_options,                  // 处理函数
    /* Nothing to do in get */
};

int driver_init(void) {
    int ret;
    // 注册setsockopt接口，用于设置是否插入toa的标记位
    ret = nf_register_sockopt(&toa_insert_sockopts);
    if (ret) {
        pr_err("register toa_insert_sockopts failed\n");
    }
    return ret;
}

void driver_uninit(void) {
    nf_unregister_sockopt(&toa_insert_sockopts);
}
```

- 用户层使用

```cpp
#define TOA_SET_IP 0x248

int main() {
    ...
    // 使用IP_PROTO
    ret = setsockopt(sockfd, IPPROTO_IP, TOA_SET_IP, &data, sizeof(data));
    ...
}
```

### 1.3. 改包

#### 1) OUTPUT链添加toa选项

```cpp
#include "toa_netfilter.h"

#include <linux/netfilter.h>
#include <net/sock.h>
#include <net/tcp.h>
#include <uapi/linux/netfilter_ipv4.h>

// 记录当前是否启用hook功能，0表示未启用，1表示启用
static atomic_t g_hook_enable = ATOMIC_INIT(0);
static void enable_netfilter_hook(void) { atomic_set(&g_hook_enable, 1); }

static void disable_netfilter_hook(void) { atomic_set(&g_hook_enable, 0); }

bool is_toa_netfilter_enable(void) { return atomic_read(&g_hook_enable) == 1; }

static bool insert_ipv4_toa(struct sock *sk, struct sk_buff *skb) {
    struct tcphdr *th;
    unsigned int tcp_options_size, tcp_header_size;
    int length, ip_offset, tcp_offset;
    int mac_offset = 0;
    __be32 *ptr;
    __be32 i;
    u16 port = 0;

    // 计算当前tcp头部长度和剩余空间
    tcp_options_size = tcp_optlen(skb);
    if (MAX_TCP_OPTION_SPACE - tcp_options_size < TCPOLEN_TOAV4_ALIGNED) {
        // 剩余空间不足
        LOG_WARN("there is not enough space left to add toa v4, remain size %d",
                     MAX_TCP_OPTION_SPACE - tcp_options_size);
        return false;
    }

    // 增加头部长度并将固定长度拷贝到新位置，toa插入到option的第一段
    // 先计算一下ip头到tcp头的偏移量，再进行设置transport_header位置
    tcp_offset = skb_transport_offset(skb);
    ip_offset = skb_network_offset(skb);
    if (skb_mac_header_was_set(skb)) {
        // 正常来讲是进到output链是没有mac头，这里算是防御性编程，防止mac头被覆盖
        mac_offset = skb_mac_offset(skb);
    }
    ptr = skb_push(skb, TCPOLEN_TOAV4_ALIGNED);
    skb_set_network_header(skb, ip_offset);
    skb_set_transport_header(skb, tcp_offset);
    if (mac_offset) {
        skb_set_mac_header(skb, mac_offset);
    }
    // 按照4字节的方式计算整体偏移到tcp固定长度头的长度，进行内存移位计算
    length = (skb_transport_offset(skb) + sizeof(struct tcphdr)) >> 2;
    for (i = 0; i < length; i++) {
        *ptr = *(ptr + TCPOLEN_TOAV4_ALIGNED / 4);
        ptr++;
    }

    /*
    正好4字节对齐
    |-------------+------------+-------------+-----------------------+
    | KIND(1byte) | LEN(1byte) | Port(2byte) | IPV4/IPV6(4/16byte)   |
    |-------------+------------+-------------+-----------------------+
    |    200      |    8/20    |  Port       |   IP                  |
    |-------------+------------+-------------+-----------------------+
     */
    // 设置toa
    *ptr++ = htonl((TCPOPT_TOA << 24) | (TCPOLEN_TOAV4 << 16) | (htons(port)));
    // 存放于sk_user_data的高32位中，从s_addr存入的，所以不需要做htonl
    *ptr++ = (u32)((u64)(sk->sk_user_data) >> 32);
    // 清理sk_user_data，防止影响到后面的流程
    sk->sk_user_data = NULL;

    // 修改tcp头部长度后，需要重新计算伪首部的校验和
    th = tcp_hdr(skb);
    tcp_header_size = tcp_hdrlen(skb) + TCPOLEN_TOAV4_ALIGNED;
    th->doff = tcp_header_size >> 2;
    return true;
}

static bool insert_ipv6_toa(struct sock *sk, struct sk_buff *skb) {
    static const char *tag = __FUNCTION__;
    toa_ipv6_node_t *node;
    struct tcphdr *th;
    unsigned int tcp_options_size, tcp_header_size;
    int length, ip_offset, tcp_offset;
    int mac_offset = 0;
    __be32 *ptr;
    __be32 i;
    u16 port = 0;

    // 计算当前tcp头部长度和剩余空间
    tcp_options_size = tcp_optlen(skb);
    if (MAX_TCP_OPTION_SPACE - tcp_options_size < TCPOLEN_TOAV6_ALIGNED) {
        // 剩余空间不足
        LOG_WARN("%s, there is not enough space left to add toa v6, remain size %d", tag,
                     MAX_TCP_OPTION_SPACE - tcp_options_size);
        return NF_ACCEPT;
    }

    // 看能不能找到，找到同时删除节点
    node = toa_ipv6_lookup_node(sk, true);
    if (!node) {
        LOG_WARN("%s, can not find ipv6 node, maybe cleaned", tag);
        return NF_ACCEPT;
    }

    // 增加头部长度并将固定长度拷贝到新位置，toa插入到option的第一段
    // 先计算一下ip头到tcp头的偏移量，再进行设置transport_header位置
    tcp_offset = skb_transport_offset(skb);
    ip_offset = skb_network_offset(skb);
    if (skb_mac_header_was_set(skb)) {
        // 正常来讲是进到output链是没有mac头，这里算是防御性编程，防止mac头被覆盖
        mac_offset = skb_mac_offset(skb);
    }
    ptr = skb_push(skb, TCPOLEN_TOAV6_ALIGNED);
    skb_set_network_header(skb, ip_offset);
    skb_set_transport_header(skb, tcp_offset);
    if (mac_offset) {
        skb_set_mac_header(skb, mac_offset);
    }
    // 按照4字节的方式计算整体偏移到tcp固定长度头的长度，进行内存移位计算
    length = (skb_transport_offset(skb) + sizeof(struct tcphdr)) >> 2;
    for (i = 0; i < length; i++) {
        *ptr = *(ptr + TCPOLEN_TOAV6_ALIGNED / 4);
        ptr++;
    }

    /*
    正好4字节对齐
    |-------------+------------+-------------+-----------------------+
    | KIND(1byte) | LEN(1byte) | Port(2byte) | IPV6(16byte)          |
    |-------------+------------+-------------+-----------------------+
    |    200      |      20    |  Port       |   IP                  |
    |-------------+------------+-------------+-----------------------+
     */
    // 设置toa
    *ptr++ = htonl((TCPOPT_TOA << 24) | (TCPOLEN_TOAV6 << 16) | (htons(port)));
    memcpy(ptr, &node->ipv6, sizeof(node->ipv6));
    kmem_cache_free(toa_ipv6_node_slab, node);
    node = NULL;

    // 清理sk_user_data，防止影响到后面的流程
    sk->sk_user_data = NULL;

    // 修改tcp头部长度后，需要重新计算伪首部的校验和
    th = tcp_hdr(skb);
    tcp_header_size = tcp_hdrlen(skb) + TCPOLEN_TOAV6_ALIGNED;
    th->doff = tcp_header_size >> 2;
    return NF_ACCEPT;
}

// local out hook
static unsigned int toa_insert_hook(void *priv, struct sk_buff *skb,
                                    const struct nf_hook_state *state) {
    struct tcphdr *th;
    struct sock *sk = skb_to_full_sk(skb);

    if (!is_toa_netfilter_enable()) return NF_ACCEPT;

    // 这里需要判断sock是否是完整的，只有完整的sock中flag才是真的flag
    // 非完整的request_sock中flag是skc_listener
    // 非完整的inet_timewait_sock中flag是skc_tw_dr
    if (!sk || !sk_fullsock(sk)) {
        return NF_ACCEPT;
    }

    switch (sk->sk_family) {
        case PF_INET: {
            // output链进来的是ip包，这里用ip头判断只处理tcp协议包
            struct iphdr *iph = ip_hdr(skb);
            if (likely(iph->protocol != IPPROTO_TCP)) {
                return NF_ACCEPT;
            }
        } break;
        case PF_INET6: {
            // output链进来的是ip包，这里用ip头判断只处理tcp协议包
            // 这里仅处理sdp-tunnel代理的短隧道访问的握手包，由于是我们自己的代码，所以不考虑ack或syn带数据的场景
            // 也就不存在分片包场景，这里就仅处理nexthdr为tcp的，分片的直接放过
            struct ipv6hdr *iph = ipv6_hdr(skb);
            if (likely(iph->nexthdr != NEXTHDR_TCP)) {
                return NF_ACCEPT;
            }
        } break;
        default:
            // 其他协议不处理，直接放通
            return NF_ACCEPT;
    }

    // 只处理syn包
    th = tcp_hdr(skb);
    switch (s_toa_mode) {
        case TOA_MODE_SYN:
            // 只处理syn包
            if (th->syn && !th->ack) {
                break;
            }
            return NF_ACCEPT;
        case TOA_MODE_ACK:
            // 只处理ack包
            if (th->ack && !th->syn && !th->fin) {
                break;
            }
            return NF_ACCEPT;
        default:
            LOG_WARN("unsupport toa_mode %d", s_toa_mode);
            return NF_ACCEPT;
    }

    // 只有完整的sock中flag才是真的flag，前面处理了，这里一定是完整的sk
    // 非完整的request_sock中flag是skc_listener
    // 非完整的inet_timewait_sock中flag是skc_tw_dr
    if (sock_flag(sk, SOCK_TOAV4_SET)) {
        if (!insert_ipv4_toa(sk, skb)) {
            return NF_ACCEPT;
        }
        // 仅设置一次，设置后清理flag
        sock_reset_flag(sk, SOCK_TOAV4_SET);
        INC_ATOMIC(s_settcpopt_count);
    } else if (sock_flag(sk, SOCK_TOAV6_SET)) {
        if (!insert_ipv6_toa(sk, skb)) {
            return NF_ACCEPT;
        }
        // 仅设置一次，设置后清理flag
        sock_reset_flag(sk, SOCK_TOAV6_SET);
        INC_ATOMIC(s_settcpoptv6_count);
    } else {
        // 没有设置过，直接放通
        return NF_ACCEPT;
    }

    // 处理完tcp层的toa插入后，需要重新处理一下checksum和头部长度
    if (sk->sk_family == PF_INET) {
        struct iphdr *iph = ip_hdr(skb);  // 由于skb指针偏移，需要重新获取iph的地址
        // checksum计算的是整个tcp长度，所以计算长度取的是整体长度减去头部长度
        th = tcp_hdr(skb);  // 由于skb指针偏移，需要重新获取tcp的地址
        th->check = ~tcp_v4_check(skb->len - skb_transport_offset(skb), iph->saddr, iph->daddr, 0);
        skb->csum_start = skb_transport_header(skb) - skb->head;
        skb->csum_offset = offsetof(struct tcphdr, check);

        // 修改ip头部长度和checksum
        iph->tot_len = htons(skb->len);
        ip_send_check(iph);
    } else {
        struct ipv6hdr *iph = ipv6_hdr(skb);  // 由于skb指针偏移，需要重新获取iph的地址
        // checksum计算的是整个tcp长度，所以计算长度取的是整体长度减去头部长度
        th = tcp_hdr(skb);  // 由于skb指针偏移，需要重新获取tcp的地址
        th->check =
            ~tcp_v6_check(skb->len - skb_transport_offset(skb), &iph->saddr, &iph->daddr, 0);
        skb->csum_start = skb_transport_header(skb) - skb->head;
        skb->csum_offset = offsetof(struct tcphdr, check);

        // 修改ip头部长度
        iph->payload_len = htons(skb->len - sizeof(struct ipv6hdr));
    }
    return NF_ACCEPT;
}

static struct nf_hook_ops s_hook_local_out = {
    .hook = toa_insert_hook,
    .hooknum = NF_INET_LOCAL_OUT,
    .pf = NFPROTO_INET,
    .priority = NF_IP_PRI_FIRST,
};

void unregister_toa_netfilter_hook(void) { nf_unregister_net_hook(&init_net, &s_hook_local_out); }

int register_toa_netfilter_hook(void) { return nf_register_net_hook(&init_net, &s_hook_local_out); }
```

## 2. ftrace hook

参见 [ftrace](/bookPages/docs/linux/performance/ftrace/)

## 3. proc目录

参考 [在Linux驱动中使用proc子系统](https://www.cnblogs.com/schips/p/using_proc_ss_in_linux_driver.html)

```cpp
#include "toa_proc.h"

#include <linux/fs.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>

static inline bool is_writable(unsigned flags) {
    return (flags & O_WRONLY) || (flags & O_RDWR) || (flags & O_CREAT) || (flags & O_TRUNC) || (flags & O_APPEND);
}

static int toa_proc_enable_show(struct seq_file *m, void *v) {
    // seq_printf会输出到控制台，也就是使用cat xxx可以看到
    seq_printf(m, "%d\n", is_ftrace_hook_enabled());
    return 0;
}

static int toa_proc_enable_open(struct inode *inode, struct file *filp) {
    // 使用single_open将要实现的show函数穿进去就好了
    return single_open(filp, toa_proc_enable_show, inode->i_private);
}

// 定义写函数，对proc文件写入的数据会放到buffer中
static ssize_t toa_proc_enable_write(struct file *file, const char __user *buffer, size_t count, loff_t *pos) {
    char mode;

    if (count > 0) {
        // 从用户空间的buffer读取一个字符
        if (get_user(mode, buffer)) return -EFAULT;

        if (mode == '1') {
            // do something
        } else if (mode == '0') {
            // do something
        }
    }

    return count;
}

static int toa_proc_debug_info_show(struct seq_file *m, void *v) {
    seq_printf(m, "ftrace hook\n");
    return 0;
}

static int toa_proc_debug_info_open(struct inode *inode, struct file *filp) {
    return single_open(filp, toa_proc_debug_info_show, inode->i_private);
}

struct toa_proc_t {
    struct proc_dir_entry *main_dir;         // 主目录entry
    struct proc_dir_entry *enable_file;      // enable文件的entry
    struct file_operations enable_fops;      // enable文件的fops
    struct proc_dir_entry *debug_info_file;  // debug_info文件的entry
    struct file_operations debug_info_fops;  // enable文件的fops
};

static struct toa_proc_t s_toa_proc = {
    // 只读文件，只需要实现open函数，read、llseek、release使用内核自带的
    .debug_info_file = NULL,
    .debug_info_fops =
        {
            .open = toa_proc_debug_info_open,
            .read = seq_read,
            .llseek = seq_lseek,
            .release = single_release,
        },
    // 可读可写文件，需要实现open和write函数，read、llseek、release使用内核自带的
    .enable_file = NULL,
    .enable_fops =
        {
            .open = toa_proc_enable_open,
            .read = seq_read,
            .write = toa_proc_enable_write,
            .llseek = seq_lseek,
            .release = single_release,
        },
    .main_dir = NULL,
};

/**
 * @brief 初始化proc相关，仅允许调用一次，需要外部保证
 *
 * @return int 0代表成功，失败返回-errno
 *
 * /proc/sdp-toa/enable 控制hook的开启和关闭
 * /proc/sdp-toa/debug_info 打印内部调试信息
 */
int toa_proc_init(void) {
    int ret = 0;

    if (s_toa_proc.main_dir) {
        return -EEXIST;
    }
    // 创建proc目录
    s_toa_proc.main_dir = proc_mkdir("sdp-toa", NULL);
    if (!s_toa_proc.main_dir) {
        return -EBUSY;
    }

    // 创建proc文件enable，可写可读
    s_toa_proc.enable_file = proc_create("enable", 0644, s_toa_proc.main_dir, &s_toa_proc.enable_fops);
    if (!s_toa_proc.enable_file) {
        ret = -EBUSY;
        goto err_clean;
    }

    // 创建文件debug_info，只读
    s_toa_proc.debug_info_file = proc_create("debug_info", 0444, s_toa_proc.main_dir, &s_toa_proc.debug_info_fops);
    if (!s_toa_proc.debug_info_file) {
        ret = -EBUSY;
        goto err_clean;
    }

    return 0;
err_clean:
    toa_proc_exit();
    return ret;
}

/**
 * @brief 销毁proc目录
 *
 */
void toa_proc_exit(void) {
    if (s_toa_proc.debug_info_file) {
        proc_remove(s_toa_proc.debug_info_file);
        s_toa_proc.debug_info_file = NULL;
    }

    if (s_toa_proc.enable_file) {
        proc_remove(s_toa_proc.enable_file);
        s_toa_proc.enable_file = NULL;
    }

    if (s_toa_proc.main_dir) {
        proc_remove(s_toa_proc.main_dir);
        s_toa_proc.main_dir = NULL;
    }
}
```

## 4. 定时器

```cpp
#include <linux/jiffies.h>
#include <linux/timer.h>

static int toav6_expire_time = 5 * 60;  // 默认过期时间5分钟
module_param(toav6_expire_time, int, S_IRUGO);
#define EXPIRE_TIME_JIFFIES (msecs_to_jiffies(toav6_expire_time * 1000))
#define TIMER_INTERVAL EXPIRE_TIME_JIFFIES  // 使用过期时间作为定时器间隔

static struct timer_list toa_timer;

static void toa_timer_callback(struct timer_list *timer) {
    // do something

    // Re-arm the timer
    mod_timer(timer, jiffies + TIMER_INTERVAL);
}

bool toa_timer_init(void) {
    static const char *tag = __FUNCTION__;
    LOG_INFO("%s, init timer, intervel %d s", tag, toav6_expire_time);
    // Initialize the timer
    timer_setup(&toa_timer, toa_timer_callback, 0);

    // Set the timer to fire in 5 minutes
    return mod_timer(&toa_timer, jiffies + TIMER_INTERVAL);
}

void toa_timer_exit(void) {
    del_timer(&toa_timer);
}
```
