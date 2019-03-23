---
title: 网络编程笔记（C语言）
date: 2019-03-22 17:25:35
tags: [network, C/C++]
categories: [notes, study]
---

# 环境

- 操作系统: Linux
- 编译器: make

# 套接字

- TCP用主机的IP地址加上主机上的端口号作为TCP连接的端点，这种端点就叫做套接字（socket）或插口。
- 套接字用（IP地址：端口号）表示。
- 它是网络通信过程中端点的抽象表示，包含进行网络通信必需的五种信息：
  - 连接使用的协议
  - 本地主机的IP地址
  - 本地进程的协议端口
  - 远地主机的IP地址
  - 远地进程的协议端口。

## 创建一个套接字

```C
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

### 参数取值

__domain 指明使用的协议族

- AF_INET:Address Family，指定TCP/IP协议家族
- PF_INET:Protocol Family
  - 在windows中 AF_INET和PF_INET完全一样
  - 在某些Linux中两者会有差距（但一般也相同），理论上建立socket时是指定协议，应该用PF_XXX,设置地址时用AF_XXX，不过在两者相等的情况下混用也没啥。
- AF_UNIX：用于同一台计算机的进程间通信
- AF_INET6:ipv6网络协议

__type 指明socket类型

- SOCK_STREAM:流套接字，对应TCP协议
- SOCK_DGRAM:数据报套接字，对应UDP协议
- SOCK_RAW:原始套接字，提供原始网络协议存取
- SOCK_PACKET:直接从网络驱动获取数据，即从数据链路层开始处理（过时了）
  - 如果想获取数据链路层，可用`socket(PF_PACKET, SOCK_RAW, htons(ETH_P_IP|ETH_P_ARP|ETH_P_ALL))`

__protocol 协议类型

- 传输层：IPPROTO_TCP、IPPROTO_UDP、IPPROTO_ICMP
- 网络层：htons(ETH_P_IP|ETH_P_ARP|ETH_P_ALL)

**三者参数并不完全独立，比如type选用sock_stream,protocol就得是IPPRPTP_TCP**

## 绑定本地端口

```C
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

### 参数取值

__addr 指向要绑定给sockfd的协议地址

**赋值时需要转换为[网络字节序](#networkByte)**
**网络地址的转换可以使用[工具函数](#networkAddress)**

```C
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

## TCP连接

### TCP服务器

```C
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

### TCP客户端

```C
    #include <sys/socket.h>

    /*
     * @description 开始监听
     * @param __fd 套接字描述符
     * @param __n 等待连接的队列长度
     * @return 0，成功；-1，错误，原因存于errno
     */
    
```

# 网络编程工具函数

## <span id = "networkByte">网络字节序转换</span>

```C
    #include <netinet/in.h>

    extern uint32_t ntohl (uint32_t __netlong) __THROW __attribute__ ((__const__));
    extern uint16_t ntohs (uint16_t __netshort)
        __THROW __attribute__ ((__const__));
    extern uint32_t htonl (uint32_t __hostlong)
        __THROW __attribute__ ((__const__));
    extern uint16_t htons (uint16_t __hostshort)
        __THROW __attribute__ ((__const__));
```

## <span id = "networkAddress">网络地址转换 in_addr_t和char *</span>

```C
    #include <netinet/in.h>

    /*
     * @description 将数字-点方式(192.168.1.1)的ip字符串转成in_addr_t格式
     * @param __cp 字符串
     * @return in_addr_t类型地址
     */
    in_addr_t inet_addr (const char *__cp);

    /*
     * @description 将struct in_addr格式的ip地址转成数字-点方式(192.168.1.1)
     * @param __in ip结构体
     * @return 数字-点方式ip字符串
     */
    char *inet_ntoa (struct in_addr __in);
```
