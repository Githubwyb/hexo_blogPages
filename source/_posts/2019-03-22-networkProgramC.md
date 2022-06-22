---
title: 网络编程笔记（C语言）
date: 2019-03-22 17:25:35
tags: [网络]
categories: [Program, C/C++]
---

# 环境

- 操作系统: Linux
- 编译器: make

# 一、套接字

- TCP用主机的IP地址加上主机上的端口号作为TCP连接的端点，这种端点就叫做套接字（socket）或插口。
- 套接字用（IP地址：端口号）表示。
- 它是网络通信过程中端点的抽象表示，包含进行网络通信必需的五种信息：
  - 连接使用的协议
  - 本地主机的IP地址
  - 本地进程的协议端口
  - 远地主机的IP地址
  - 远地进程的协议端口。

## 1. 创建一个套接字

```cpp
#include <sys/socket.h>

/*
 * @description 创建一个套接字
 * @param __domain 指明使用的协议族
 * @param __type 指明socket类型
 * @param __protocol 协议类型
 * @return -1，出错；其他，描述符
 */
int socket (int __domain, int __type, int __protocol);
```

### 1.1. 参数取值

**`__domain` 指明使用的协议族**

- `AF_INET`: Address Family，指定TCP/IP协议家族
- `PF_INET`: Protocol Family
  - 在windows中 `AF_INET`和`PF_INET`完全一样
  - 在某些Linux中两者会有差距（但一般也相同），理论上建立socket时是指定协议，应该用`PF_XXX`，设置地址时用`AF_XXX`，不过在两者相等的情况下混用也没啥。
- `AF_UNIX`: 用于同一台计算机的进程间通信
- `AF_INET6`:ipv6网络协议

**`__type` 指明socket类型**

- `SOCK_STREAM`: 流套接字，对应TCP协议
- `SOCK_DGRAM`: 数据报套接字，对应UDP协议
- `SOCK_RAW`: 原始套接字，提供原始网络协议存取
- `SOCK_PACKET`: 直接从网络驱动获取数据，即从数据链路层开始处理（过时了）
  - 如果想获取数据链路层，可用`socket(PF_PACKET, SOCK_RAW, htons(ETH_P_IP|ETH_P_ARP|ETH_P_ALL))`

`__protocol` 协议类型

- 传输层: `IPPROTO_TCP`、`IPPROTO_UDP`、`IPPROTO_ICMP`
- 网络层: `htons(ETH_P_IP|ETH_P_ARP|ETH_P_ALL)`

**<font color="red">三者参数并不完全独立，比如type选用`sock_stream`，protocol就得是`IPPRPTP_TCP`</font>**

## 2. 绑定协议地址

```cpp
#include <sys/socket.h>

/*
 * @description 将套接字绑定本地端口
 * @param __fd 套接字描述符
 * @param __addr 指向要绑定给sockfd的协议地址
 * @param __len 地址的长度
 * @return 0，成功；-1，错误，原因存于errno
 */
int bind (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len)
```

### 2.1. 参数取值

`__addr` 指向要绑定给sockfd的协议地址

**赋值时需要转换为[网络字节序](#networkByte)**
**网络地址的转换可以使用[工具函数](#networkAddress)**

```cpp
#include <netinet/in.h>

# define __CONST_SOCKADDR_ARG	const struct sockaddr *

//ipv4取值
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET */
    in_port_t      sin_port;   /* port in network byte order */
    struct in_addr sin_addr;   /* internet address */
};

/* Internet address. */
struct in_addr {
    uint32_t       s_addr;     /* address in network byte order */
};

//ipv6取值
struct sockaddr_in6 {
    sa_family_t     sin6_family;   /* AF_INET6 */
    in_port_t       sin6_port;     /* port number */
    uint32_t        sin6_flowinfo; /* IPv6 flow information */
    struct in6_addr sin6_addr;     /* IPv6 address */
    uint32_t        sin6_scope_id; /* Scope ID (new in 2.4) */
};

struct in6_addr {
    unsigned char   s6_addr[16];   /* IPv6 address */
};

//Unix取值
#define UNIX_PATH_MAX    108

struct sockaddr_un {
    sa_family_t sun_family;               /* AF_UNIX */
    char        sun_path[UNIX_PATH_MAX];  /* pathname */
};
```

# 二、连接

## 1. TCP连接

### 1.1. TCP服务器

```cpp
#include <sys/socket.h>

/*
 * @description 开始监听
 * @param __fd 套接字描述符
 * @param __n 等待连接的队列长度
 * @return 0，成功；-1，错误，原因存于errno
 */
int listen (int __fd, int __n);

/*
 * @description 阻塞式接受第一个连接
 * @param __fd 套接字描述符
 * @param __addr 连接地址指针
 * @param __addr_len 地址长度指针
 * @return 连接套接字描述符
 */
int accept (int __fd, __SOCKADDR_ARG __addr, socklen_t *__restrict __addr_len);
```

### 1.2. TCP客户端

```cpp
#include <sys/socket.h>

   /*
    * @description 开始监听
    * @param __fd 套接字描述符
    * @param __n 等待连接的队列长度
    * @return 0，成功；-1，错误，原因存于errno
    */

```

## 2. UDP连接

# 三、select/poll/epoll

## 1. select

- windows和linux都存在select
- select可以同时监听多个文件描述符，但是对描述符的处理是轮询的方式，对本地fd池一个一个扫描看是否有数据
- 最大支持`FD_SETSIZE`个文件描述符，一般为`1024/2048`

## 2. poll

- 在select基础上去除了文件描述符的限制
- 轮询机制还是保留了

# 四、epoll

- epoll是linux特有的系统调用，windows没有此实现，使用mingw是模拟的一种实现
- epoll和select区别是，对于fd池的处理，epoll不会一个一个扫描，而是在内核注册了通知机制，当某个fd出现时间，内核会直接通知epoll
- epoll返回的就直接是fd的事件信息，这样防止了扫描带来的开销

## 1. epoll的调用

```cpp

```

## 2. epoll下socket是否要设置为非阻塞

[使用epoll时需要将socket设为非阻塞吗？](https://blog.csdn.net/boiled_water123/article/details/104161471)

1. 监听的服务端fd，最好使用水平触发模式，边沿触发可能导致部分客户端连接不上
2. 客户端fd，使用水平触发时，阻塞非阻塞都可以，建议设置非阻塞
3. 客户端fd，使用边沿触发时，必须使用非阻塞io，否则会卡在读取调用上。但是要求必须一次读完所有数据，否则可能存在数据没有处理。

# 五、网络编程

## 1. 头文件

### 1.1. netinet和linux下的头文件区别

- netinet是用户空间的接口头文件，一般是应用程序使用
- linux下是linux内核使用，一般是内核相关操作使用

## 2. 一些工具函数

### 2.1. <span id = "networkByte">网络字节序转换</span>

```cpp
#include <netinet/in.h>

uint32_t ntohl (uint32_t __netlong);
uint16_t ntohs (uint16_t __netshort);
uint32_t htonl (uint32_t __hostlong);
uint16_t htons (uint16_t __hostshort);
```

### 2.2. <span id = "networkAddress">网络地址转换 `in_addr_t`和`char *`</span>

```cpp
#include <arpa/inet.h>

/*
    * @description 将数字-点方式(192.168.1.1)的ip字符串转成in_addr_t格式
    * @param __cp 字符串
    * @return in_addr_t类型地址
    */
in_addr_t inet_addr (const char *__cp);

/* Convert a Internet address in binary network format for interface
   type AF in buffer starting at CP to presentation form and place
   result in buffer of length LEN astarting at BUF.  */
extern const char *inet_ntop (int __af, const void *__restrict __cp,
			      char *__restrict __buf, socklen_t __len)
     __THROW;
```

**示例**

```cpp
#include <arpa/inet.h>
#include <netinet/ip.h>

#include "log.hpp"

void handle(char *buf, int size) {
    // 转成ip结构体判断
    ipheader = (struct iphdr *)buf;

    char ipbuf1[16] = {0};  // 打印ip用
    char ipbuf2[16] = {0};  // 打印ip用
    struct in_addr addr1 = {0};
    struct in_addr addr2 = {0};
    addr1.s_addr = ipheader->saddr;  // 注意这里是网络字节序，不是本地字节序
    addr2.s_addr = ipheader->daddr;  // 注意这里是网络字节序，不是本地字节序

    LOG_INFO("src->dst: %s->%s", inet_ntop(AF_INET, &addr1, ipbuf1, sizeof(ipbuf1)),
             inet_ntop(AF_INET, &addr2, ipbuf2, sizeof(ipbuf2)));
}
```

## 3. 分片包处理

主要用到一些宏

```cpp
#include <netinet/ip.h>

void handle(char *buf, int size) {
    // 转成ip结构体判断
    ipheader = (struct iphdr *)ptr;
    frag_flag = ntohs(ipheader->frag_off);  // 注意这里用主机字节序

    if ((frag_flag & IP_DF) == 0) {
        // 分片包
        ...
        if ((frag_flag & IP_OFFMASK) == 0) {
            // 分片包首包
            ...
        }
    }
}
```

## 4. tcp头处理

```cpp
#include <netinet/tcp.h>

#include "log.hpp"

unsigned char tcpheader[] = {0x60, 0xe0, 0xef, 0x41, 0x01, 0xc5, 0x13, 0x41, 0x55, 0xce,
                             0x63, 0xb5, 0x50, 0x10, 0x25, 0x17, 0xb1, 0x0e, 0x00, 0x00};

int main(int argC, char *argV[]) {
    tcphdr *th = (tcphdr *)tcpheader;

    LOG_HEX(tcpheader, sizeof(tcphdr));
    LOG_DEBUG("th->source %u", htons(th->source));
    LOG_DEBUG("th->dest %u", htons(th->dest));
    LOG_DEBUG("th->seq %u", htonl(th->seq));
    LOG_DEBUG("th->ack_seq %u", htonl(th->ack_seq));
    LOG_DEBUG("th->doff %u", th->doff);
    LOG_DEBUG("th->res1 %04u", th->res1);
    LOG_DEBUG("th->res2 %02u", th->res2);
    LOG_DEBUG("th->urg %u", th->urg);
    LOG_DEBUG("th->ack %u", th->ack);
    LOG_DEBUG("th->psh %u", th->psh);
    LOG_DEBUG("th->rst %u", th->rst);
    LOG_DEBUG("th->syn %u", th->syn);
    LOG_DEBUG("th->fin %u", th->fin);

    LOG_DEBUG("th->th_flags %#x", th->th_flags);
    LOG_DEBUG("is ack", (th->th_flags & TH_ACK) == TH_ACK);
    return 0;
}

```

# 踩坑记

## 1. TCP连接后，一方断电，另一方是无法检测到对方断电的

- TCP连接并没有检测机制，所以一方断电，另一方无法检测到。
- 强行退出程序，操作系统会回收资源，内部执行关闭套接字，所以客户端可以检测到断开连接。

## 2. SO_KEEPALIVE 保活

TCP连接选项中有一项是心跳保活机制，但并不是规范的一部分，官方RFC罗列不适用的三个理由:

1. 在短暂的故障期间，可能使一个良好的连接被释放
2. 占用了不必要的贷款
3. 在以数据包计费的网络上花费额外的金钱
