---
title: linux网络编程（C语言）
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

调用参考文档 [socket() — Create a socket](https://www.ibm.com/docs/en/zos/2.3.0?topic=functions-socket-create-socket)

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
- `AF_UNIX`: 域套接字，用于同一台计算机的进程间通信
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

**<font color="red">三者参数并不完全独立，比如type选用`SOCK_STREAM`，protocol就得是`IPPRPTP_TCP`</font>**

## 2. 绑定协议地址

- 绑定地址后，可以使用地址进行读写和监听，这里会判断是否已被占用
- 如果端口想要复用，可以使用setsockopt设置socket为可以重复使用地址

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
```

### 1.3. TCP断开直接发送rst包

linker机制

| l_onoff | l_linger | closesocket行为                  | 发送队列                         | 底层行为                                    |
| ------- | -------- | -------------------------------- | -------------------------------- | ------------------------------------------- |
| 0       | x        | 立即返回                         | 保持直至发送完成                 | 系统接管套接字，数据发送完成                |
| x       | 0        | 立即返回                         | 立即放弃                         | 直接发送rst包，自身立即复位，不经过2msl状态 |
| x       | x        | 阻塞到l_linger超时或数据发送完成 | 超时时间内尝试发送，超时立即放弃 | 超时则发送rst包，发送完成就是fin            |

```cpp
// 设置方式
struct linger ling;
ling.l_onoff = 1;
ling.l_linger = 0;
setsockopt(sockfd, SOL_SOCKET, SO_LINGER, &ling, sizeof(ling));
close(sockfd);  // 这里直接发送rst
```

## 2. UDP连接

## 3. close()和shutdown()的区别

**close()**

- 头文件`unistd.h`
- `close()`是关闭文件句柄，如果文件句柄没有引用，会找到对应的socket进行关闭清理
- 如果存在多个进程共享一个文件句柄，`close()`一个不会断开连接，多个进程都`close()`才会断开连接

**shutdown()**

- `shutdown()`是关闭socket连接，一个进程关闭连接，另一个进程也无法使用
- `shutdown()`针对的是socket不是文件，关闭socket之后，文件还在，还需要调用`close()`才能关闭文件
- `shutdown()`存在第二个参数，控制关闭的方向
    - `SHUT_RD`: 只关闭读端，来的数据会丢掉，对方不知道读端关闭
    - `SHUT_WR`: 只关闭写端，发送FIN包，对方知道此事情
    - `SHUT_RDWR`: 关闭读写端，相当于调用两次，一次指定读，一次是写

```cpp
// include/linux/net.h
/**
 * enum sock_shutdown_cmd - Shutdown types
 * @SHUT_RD: shutdown receptions
 * @SHUT_WR: shutdown transmissions
 * @SHUT_RDWR: shutdown receptions/transmissions
 */
enum sock_shutdown_cmd {
	SHUT_RD,
	SHUT_WR,
	SHUT_RDWR,
};
```

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

1. 服务端用于监听的fd，最好使用水平触发模式，边沿触发可能导致部分客户端连接不上
2. 和客户端交互的fd，使用水平触发时，阻塞非阻塞都可以，建议设置非阻塞
3. 和客户端交互fd，使用边沿触发时，必须使用非阻塞io，否则会卡在读取调用上。但是要求必须一次读完所有数据，否则可能存在数据没有处理。

# 五、网络编程

## 1. 头文件

### 1.1. netinet和linux下的头文件区别

- netinet是用户空间的接口头文件，一般是应用程序使用
- linux下是linux内核使用，一般是内核相关操作使用

## 2. 一些工具函数

### 2.1. <span id = "networkByte">网络字节序转换</span>

```cpp
// #include <netinet/in.h>

uint32_t ntohl (uint32_t __netlong);
uint16_t ntohs (uint16_t __netshort);
uint32_t htonl (uint32_t __hostlong);
uint16_t htons (uint16_t __hostshort);
```

### 2.2. <span id = "networkAddress">网络地址转换 `in_addr_t`和`char *`</span>

```cpp
// #include <arpa/inet.h>

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

## 5. 获取系统dns服务器

- 下面函数是glibc提供的从`/etc/resolv.conf`文件中读取的dns服务器列表

```cpp
#include <netinet/in.h>
#include <resolv.h>
#include <sys/socket.h>

void getSystemNSList() {
    errno = 0;
    struct __res_state res;
    if (res_ninit(&res) == 0) {
        for (int i = 0; i < res.nscount; i++) {
            if (res.nsaddr_list[i].sin_family == AF_INET) {
                // ipv4 dns server
                struct sockaddr_in *addr = res.nsaddr_list + i;
            } else if (res.nsaddr_list[i].sin_family == AF_INET6) {
                // ipv6 dns server
                struct sockaddr_in6 *addr = reinterpret_cast<sockaddr_in6 *>(res.nsaddr_list + i);
            }
        }
        res_nclose(&res);
    }
}
```

## 6. 忽略路由绑定网卡发包

- 方案一

```cpp
#include <arpa/inet.h>
#include <net/if.h>
#include <netinet/in.h>
#include <stdio.h>
#include <string.h>
#include <sys/ioctl.h>
#include <sys/socket.h>
#include <unistd.h>

static const uint8_t udpData[] = {
    0xe2, 0x60, 0x01, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02,
    0x63, 0x73, 0x06, 0x64, 0x65, 0x76, 0x6f, 0x70, 0x73, 0x07, 0x73, 0x61, 0x6e,
    0x67, 0x66, 0x6f, 0x72, 0x03, 0x6f, 0x72, 0x67, 0x00, 0x00, 0x01, 0x00, 0x01,
};

int main() {
    ssize_t ret = 0;
    char buf[1024] = {0};
    auto fd = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (fd < 0) {
        printf("socket error\n");
        return -1;
    }
    struct sockaddr_in addr = {0};
    socklen_t len = sizeof(addr);
    addr.sin_family = AF_INET;                              // ipv4
    addr.sin_port = htons(53);                              // 端口号转换为网络字节序
    addr.sin_addr.s_addr = inet_addr("114.114.114.114");    // 连接本机的服务器

    // 定义网卡接口
    struct ifreq ifr = {0};
    strncpy(ifr.ifr_ifrn.ifrn_name, "ens18", sizeof(ifr.ifr_ifrn.ifrn_name));

    // 设置socket属性，使用绑定设备的方式
    ret = setsockopt(fd, SOL_SOCKET, SO_BINDTODEVICE, &ifr, sizeof(ifr));
    if (ret < 0) {
        printf("setsockopt error\n");
        return -1;
    }

    ret = sendto(fd, udpData, sizeof(udpData), 0, (sockaddr *)&addr, sizeof(addr));
    if (ret < 0) {
        printf("send error\n");
        goto err;
    }

    ret = recvfrom(fd, buf, sizeof(buf), 0, (sockaddr *)&addr, &len);
    if (ret < 0) {
        printf("recvfrom error\n");
        goto err;
    }
    log_hex(buf, ret);
    close(fd);
    return 0;
err:
    close(fd);
    return 1;
}
```

- 方案二

```cpp
#include <arpa/inet.h>
#include <net/if.h>
#include <netinet/in.h>
#include <stdio.h>
#include <string.h>
#include <sys/ioctl.h>
#include <sys/socket.h>
#include <unistd.h>

static const uint8_t udpData[] = {
    0xe2, 0x60, 0x01, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02,
    0x63, 0x73, 0x06, 0x64, 0x65, 0x76, 0x6f, 0x70, 0x73, 0x07, 0x73, 0x61, 0x6e,
    0x67, 0x66, 0x6f, 0x72, 0x03, 0x6f, 0x72, 0x67, 0x00, 0x00, 0x01, 0x00, 0x01,
};

int main() {
    ssize_t ret = 0;
    char buf[1024] = {0};
    auto fd = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (fd < 0) {
        printf("socket error\n");
        return -1;
    }
    struct sockaddr_in addr = {0};
    socklen_t len = sizeof(addr);
    addr.sin_family = AF_INET;                              // ipv4
    addr.sin_port = htons(53);                              // 端口号转换为网络字节序
    addr.sin_addr.s_addr = inet_addr("114.114.114.114");    // 连接本机的服务器

    // 定义网卡接口
    struct ifreq ifr = {0};
    strncpy(ifr.ifr_ifrn.ifrn_name, "ens18", sizeof(ifr.ifr_ifrn.ifrn_name));
    // 获取网卡的index
    ioctl(fd, SIOCGIFINDEX, &ifr);

    uint32_t ifindex_opt = htonl(ifr.ifr_ifindex);
    // 设置socket属性，IP_UNICAST_IF为单播，传入网卡index
    ret = setsockopt(fd, IPPROTO_IP, IP_UNICAST_IF, &ifindex_opt, sizeof(ifindex_opt));
    if (ret < 0) {
        printf("setsockopt error\n");
        return -1;
    }

    ret = sendto(fd, udpData, sizeof(udpData), 0, (sockaddr *)&addr, sizeof(addr));
    if (ret < 0) {
        printf("send error\n");
        goto err;
    }

    ret = recvfrom(fd, buf, sizeof(buf), 0, (sockaddr *)&addr, &len);
    if (ret < 0) {
        printf("recvfrom error\n");
        goto err;
    }
    log_hex(buf, ret);
    close(fd);
    return 0;
err:
    close(fd);
    return 1;
}
```

## 7. 使用非本机ip发送数据包

- 主要使用的是socket的一个选项，`IP_TRANSPARENT`

```cpp
// 设置fd选项为允许透明转发
int value = 1;
setsockopt(fd, SOL_IP, IP_TRANSPARENT, &value, sizeof(value));

// 绑定一个非本机ip
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = inet_addr("2.0.1.1");
int ret = bind(fd, &addr, sizeof(addr));
```

- 发送数据包后，此socket会监听一个非本机ip的地址，而linux本身会在反向路由里面发现非本机地址的数据包会直接丢弃，所以还需要添加策略路由来允许本机接收数据包

```shell
# 对此ip回包添加mark来匹配策略路由，这样防止发出去的包也被路由处理了
=> iptables -I PREROUTING -t mangle -d 80.0.0.0/8 -j MARK --set-mark 4567
# 对此mark的数据包匹配策略路由，默认到lo本地网卡
=> ip rule add fwmark 4567 lookup 4567
# 必须要加local，代表包是走本机处理，默认的unicast是不走本机处理的
=> ip route add local 0.0.0.0/0 dev lo table 4567
```

# 六、实战示例

## 1. tcp传输

- socket确定是TCP协议后，recv和send拿到的数据是tcp数据，不包含tcp头部

### 1.1. 端口模式

- 服务端

```cpp
#include <arpa/inet.h>
#include <errno.h>
#include <netinet/in.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

int serverInitPort(int port) {
    /********** 1. 创建套接字 **********/
    auto fd = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    if (fd < 0) {
        printf("socket error, %d:%s\n", errno, strerror(errno));
        return -1;
    }

    /********** 2. 绑定端口 **********/
    struct sockaddr_in addr = {0};
    addr.sin_family = AF_INET;                 // ipv4
    addr.sin_port = htons(port);               // 端口号转换为网络字节序
    addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 监听所有地址 0.0.0.0
    auto ret = bind(fd, (struct sockaddr *)&addr, sizeof(addr));
    if (ret < 0) {
        printf("bind error, %d:%s\n", errno, strerror(errno));
        return -1;
    }

    /********** 3. 监听端口，这里可以通过netstat查看到 **********/
    ret = listen(fd, SOMAXCONN);  // 队列长度为SOMAXCONN，即最大排队连接数
    if (ret < 0) {
        printf("listen error, %d:%s\n", errno, strerror(errno));
        return -1;
    }

    return fd;
}

void serverRun() {
    auto serverFd = serverInitPort(serverPort);
    if (serverFd < 0) {
        LOG_ERROR("serverInitPort error");
        return;
    }

    while (true) {
        /********** 1. 接受客户端连接 **********/
        sockaddr_in clientAddr = {0};
        socklen_t clientAddrLen = sizeof(clientAddr);
        // 下面函数会直接阻塞，直到有客户端连接进来。后两个参数用于获取客户端的地址信息
        auto clientFd = accept(serverFd, (sockaddr *)&clientAddr, &clientAddrLen);
        if (clientFd < 0) {
            LOG_ERROR("accept error");
            continue;
        }
        LOG_DEBUG("accept client {}, addr {}:{}", clientFd, inet_ntoa(clientAddr.sin_addr), ntohs(clientAddr.sin_port));

        /********** 2. 处理客户端请求 **********/
        do {
            // 接收客户端发送消息，调用recv会直接阻塞，直到接收到消息
            char buf[1024] = {0};
            auto ret = recv(clientFd, buf, sizeof(buf), 0);
            if (ret < 0) {
                LOG_ERROR("recv error");
                break;
            }
            LOG_DEBUG("recv {} from client", buf);

            // 发送消息给客户端
            ret = send(clientFd, buf, strlen(buf), 0);
            if (ret < 0) {
                LOG_ERROR("send error");
                break;
            }
            LOG_DEBUG("send {} to client", buf);
        } while (false);

        /********** 3. 关闭客户端套接字 **********/
        close(clientFd);
    }
}
```

- 客户端

```cpp
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <errno.h>
#include <string.h>

int clientInitPort(const char *ip, int port) {
    /********** 1. 创建套接字 **********/
    auto fd = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    if (fd < 0) {
        printf("socket error, %d:%s\n", errno, strerror(errno));
        return -1;
    }

    /********** 2. 连接服务器 **********/
    struct sockaddr_in addr = {0};
    addr.sin_family = AF_INET;             // ipv4
    addr.sin_port = htons(port);           // 端口号转换为网络字节序
    addr.sin_addr.s_addr = inet_addr(ip);  // 连接本机的服务器
    // 连接服务端
    auto ret = connect(fd, (struct sockaddr *)&addr, sizeof(addr));
    if (ret < 0) {
        printf("connect error, %d:%s\n", errno, strerror(errno));
        return -1;
    }

    return fd;
}

void clientRunPort() {
    /********** 1. 创建套接字 **********/
    auto fd = clientInitPort("127.0.0.1", serverPort);
    if (fd < 0) {
        LOG_ERROR("clientInitPort error");
        return;
    }

    /********** 2. 发送消息 **********/
    auto msg = "hello";
    // recv和send拿到的都是不包含tcp头部的数据
    auto ret = send(fd, msg, strlen(msg), 0);
    if (ret < 0) {
        printf("send error, %d:%s\n", errno, strerror(errno));
        return;
    }

    /********** 3. 接收服务端的消息 **********/
    char buf[1024] = {0};
    // recv函数会直接阻塞，直到服务端发送消息过来
    ret = recv(fd, buf, sizeof(buf), 0);
    if (ret < 0) {
        LOG_ERROR("recv error");
        return;
    }
    LOG_DEBUG("recv msg {}", buf);

    /********** 4. 关闭客户端套接字 **********/
    close(fd);
}
```

### 1.2. unix套接字

- 源码分析查看[unix套接字](/blogs/2021-03-22-linux-kernel/#1-3-unix套接字)
- 服务端，区别仅在init阶段
- 建立`AF_UNIX`套接字的情况下，protocol必须设置为`0`

```cpp
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <sys/un.h>

int serverInitUnix(const char *unixPath) {
    /********** 1. 创建套接字 **********/
    auto fd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (fd < 0) {
        LOG_ERROR("socket error, unixPath {}, error {}:{}", unixPath, errno, strerror(errno));
        return -1;
    }

    /********** 2. 绑定域套接字，不存在会创建 **********/
    struct sockaddr_un addr = {0};
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, unixPath, sizeof(addr.sun_path) - 1);
    auto ret = bind(fd, (struct sockaddr *)&addr, sizeof(addr));
    if (ret < 0) {
        LOG_ERROR("bind error, unixPath {}, error {}:{}", unixPath, errno, strerror(errno));
        return -1;
    }

    /********** 3. 监听域套接字，这里可以通过netstat查看到 **********/
    ret = listen(fd, SOMAXCONN);  // 队列长度为SOMAXCONN，即最大排队连接数
    if (ret < 0) {
        LOG_ERROR("listen error, unixPath {}, error {}:{}", unixPath, errno, strerror(errno));
        return -1;
    }

    return fd;
}
```

- 客户端区别也在init阶段

```cpp
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <sys/un.h>

int clientInitUnix(const char *unixPath) {
    /********** 1. 创建套接字 **********/
    auto fd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (fd < 0) {
        LOG_ERROR("socket error");
        return -1;
    }

    /********** 2. 连接服务器 **********/
    sockaddr_un addr;
    addr.sun_family = AF_UNIX;
    strcpy(addr.sun_path, unixPath);
    auto ret = connect(fd, (struct sockaddr *)&addr, sizeof(addr));
    if (ret < 0) {
        LOG_ERROR("connect error");
        return -1;
    }
    return fd;
}
```

## 2. udp传输

- 和tcp不同点在于，服务端不需要listen，调用完bind就可以直接recv
- 客户端不需要connect，创建完socket就可以直接调用`sendto`
- 如果对端是一个服务器，但是端口没开放，会回复icmp端口不可达，这个需要通过设置sockopt才能在recvfrom时返回错误，不然就会阻塞
- 如果对端地址不可达，那就无法得知，不会回复icmp不可达的信息

### 2.2. 客户端代码

```cpp
int main(int argc, char *argv[]) {
    /********** 1. 创建套接字 **********/
    auto fd = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (fd < 0) {
        LOG_ERROR("socket error");
        return -1;
    }
    // 想要收到icmp端口不可达的信息，需要设置下面的选项
    int val = 1;
    setsockopt(fd, IPPROTO_IP, IP_RECVERR, &val, sizeof(val));

    /********** 2. 发送消息 **********/
    LOGI(WHAT("Begin send"));
    auto msg = "hello";
    struct sockaddr_in addr = {0};
    addr.sin_family = AF_INET;                    // ipv4
    addr.sin_port = htons(5555);                  // 端口号转换为网络字节序
    addr.sin_addr.s_addr = inet_addr("10.240.17.101");  // 连接本机的服务器
    auto ret = sendto(fd, msg, strlen(msg), 0, reinterpret_cast<sockaddr *>(&addr), sizeof(addr));
    if (ret < 0) {
        LOG_ERROR("send error {}", std::to_string(std::error_code(errno, std::system_category())));
        return 1;
    }

    /********** 3. 接收服务端的消息 **********/
    printf("Begin recv\n");
    char buf[1024] = {0};
    // recv函数会直接阻塞，直到服务端发送消息过来
    socklen_t len = sizeof(addr);
    ret = recvfrom(fd, buf, sizeof(buf), 0, reinterpret_cast<sockaddr *>(&addr), &len);
    if (ret < 0) {
        printf("recv error, %d:%s\n", errno, strerror(errno));
        return 1;
    }
    LOG_DEBUG("recv msg {}", buf);

    /********** 4. 关闭客户端套接字 **********/
    close(fd);
}
```

## 3. 转移文件句柄到另一个进程

- 主要使用`sendmsg/recvmsg`连个系统调用实现
- 必须使用unix套接字才能发送和接受成功，但是使用tcp协议和udp都可以，域套接字也不会丢包
- 将套接字发送出去后，本进程内的文件句柄还有效可以读写数据，但是关闭套接字不会引起客户端断开连接，即使接收进程没有进行`recvmsg`接受套接字
- 文件描述符发送时，内核会对文件描述符结构体进行拷贝到另一个进程，但是文件描述符句柄的值会根据当前进程的最小未打开的文件描述符进行计算，不会和发送端一致
  - 例如发送端发送的fd值是4，接受端受到的会变成其他值，可以是3、4、5等

##### 示例代码

- 发送端

```cpp
#include <sys/socket.h>
#define SEND_FD_NUM 1  // 可以一次发送多个文件描述符
void sendFd(int clientFd) {
    /********** 1. 和待接受的进程建立连接 **********/
    // 必须使用unix套接字才能进行文件描述符发送
    auto fd = clientInitUnix(serverPath);
    if (fd < 0) {
        LOG_ERROR("clientInitUnix error");
        return;
    }

    /********** 2. 构建msg进行发送套接字 **********/
    struct msghdr msg = {0};
    // 使用union定义可以使用宏设置字节对齐的结构体
    union {
        struct cmsghdr cm;
        char control[CMSG_SPACE(sizeof(clientFd) *
                                SEND_FD_NUM)];  // 包含cmsghdr大小和fd大小的空间，对两个都进行了8字节对齐
    } control_un;
    control_un.cm.cmsg_len =
        CMSG_LEN(sizeof(clientFd) * SEND_FD_NUM);  // 此宏代表cmsghdr大小和clientFd大小的空间，对cmsghdr进行了8字节对齐
    control_un.cm.cmsg_level = SOL_SOCKET;          // 发送文件描述符必须设置的level
    control_un.cm.cmsg_type = SCM_RIGHTS;           // 发送文件描述符
    int *fdPtr = (int *)CMSG_DATA(&control_un.cm);  // 获取文件描述符的指针
    fdPtr[0] = clientFd;                            // 设置文件描述符
    msg.msg_control = control_un.control;
    msg.msg_controllen = sizeof(control_un.control);
    // 下面的设置是必须的，用于标识是否发送成功，data可以随意设置大小
    struct iovec vec = {0};
    unsigned char data = 0;
    vec.iov_base = &data;
    vec.iov_len = sizeof(data);
    msg.msg_iov = &vec;
    msg.msg_iovlen = 1;
    LOG_DEBUG("msg_controllen {}, control_un.cm.cmsg_len {}", msg.msg_controllen, control_un.cm.cmsg_len);

    /********** 3. 发送文件描述符 **********/
    auto ret = sendmsg(fd, &msg, 0);
    if (ret < 0) {
        LOG_ERROR("sendmsg error");
        return;
    }

    /********** 4. 关闭连接套接字，发送的文件描述符是否关闭外部处理 **********/
    close(fd);
}
```

- 接收端

```cpp
int recvFd(int clientFd) {
    /********** 1. 构建msg进行接收文件描述符 **********/
    struct msghdr msg = {0};
    // 这里留下可接收的空间即可，其他不用设置
    union {
        struct cmsghdr cm;
        char control[CMSG_SPACE(sizeof(clientFd))];  // 只接收一个文件描述符，所以只需要一个空间
    } control_un;
    msg.msg_control = control_un.control;
    msg.msg_controllen = sizeof(control_un.control);
    // 这里使用同样的或者更大的内存进行接收都可以，但是必须分配内存进行接收
    struct iovec vec = {0};
    unsigned char data = 0;
    vec.iov_base = &data;
    vec.iov_len = sizeof(data);
    msg.msg_iov = &vec;
    msg.msg_iovlen = 1;
    LOG_DEBUG("msg_controllen {}, control_un.cm.cmsg_len {}", msg.msg_controllen, control_un.cm.cmsg_len);

    /********** 3. 接收文件描述符 **********/
    auto ret = recvmsg(clientFd, &msg, 0);
    if (ret < 0) {
        LOG_ERROR("recvmsg error");
        return -1;
    }

    /********** 4. 判断接收是否合法 **********/
    cmsghdr *cmptr = CMSG_FIRSTHDR(&msg);
    if (cmptr == nullptr                                  // 要存在地址
        || cmptr->cmsg_len != CMSG_LEN(sizeof(clientFd))  // 长度要够
        || cmptr->cmsg_level != SOL_SOCKET                // level要正确
        || cmptr->cmsg_type != SCM_RIGHTS) {              // type要正确
        LOG_ERROR("CMSG_FIRSTHDR error");
        return -1;
    }
    int *fdPtr = (int *)CMSG_DATA(cmptr);

    /********** 5. 返回fd **********/
    return fdPtr[0];
}
```

## 4. 原始socket发包

### 4.1. 使用原始socket实现一个tcp syn扫描器

**带ip头的处理**

```cpp
#define MAX_PACKET_SIZE 65536
#define TH_SYN 0x02
#define TH_RST 0x04
#define TH_ACK 0x10

struct pseudo_header {
    u_int32_t source_address;
    u_int32_t dest_address;
    u_int8_t placeholder;
    u_int8_t protocol;
    u_int16_t tcp_length;
    struct tcphdr tcp;
};

unsigned short csum(unsigned short *ptr, int nbytes) {
    register long sum;
    unsigned short oddbyte;
    register short answer;

    sum = 0;
    while (nbytes > 1) {
        sum += *ptr++;
        nbytes -= 2;
    }
    if (nbytes == 1) {
        oddbyte = 0;
        *((u_char *) &oddbyte) = *(u_char *) ptr;
        sum += oddbyte;
    }

    sum = (sum >> 16) + (sum & 0xffff);
    sum = sum + (sum >> 16);
    answer = (short) ~sum;

    return (answer);
}

int main(int argc, char *argv[]) {
    if (argc < 4) {
        cout << "Usage: " << argv[0] << " <source IP> <target IP> <target port>" << endl;
        return 1;
    }

    char *src_ip = argv[1];
    char *dst_ip = argv[2];
    int dst_port = atoi(argv[3]);

    int sock_raw = socket(AF_INET, SOCK_RAW, IPPROTO_TCP);
    if (sock_raw < 0) {
        cout << "Error creating socket: " << strerror(errno) << endl;
        return 1;
    }
    // 发的包里面有ip头部
    int on = 1;
    if (setsockopt(sock_raw, IPPROTO_IP, IP_HDRINCL, &on, sizeof(on)) < 0) {
        cout << "Error setting socket options: " << strerror(errno) << endl;
        return 1;
    }

    char packet[MAX_PACKET_SIZE];
    memset(packet, 0, MAX_PACKET_SIZE);

    struct iphdr *ip = (struct iphdr *) packet;
    struct tcphdr *tcp = (struct tcphdr *) (packet + sizeof(struct iphdr));
    struct sockaddr_in sin;

    sin.sin_family = AF_INET;
    sin.sin_port = htons(dst_port);
    sin.sin_addr.s_addr = inet_addr(dst_ip);

    ip->ihl = 5;
    ip->version = 4;
    ip->tos = 0;
    ip->tot_len = sizeof(struct iphdr) + sizeof(struct tcphdr);
    ip->id = htons(54321);
    ip->frag_off = 0;
    ip->ttl = 255;
    ip->protocol = IPPROTO_TCP;
    ip->check = 0;  // 这里留空，系统会帮忙计算
    ip->saddr = inet_addr(src_ip);
    ip->daddr = sin.sin_addr.s_addr;

    tcp->source = htons(1234);
    tcp->dest = htons(dst_port);
    tcp->seq = htonl(1105024978);
    tcp->ack_seq = 0;
    tcp->doff = 5;
    tcp->fin = 0;
    tcp->syn = 1;
    tcp->rst = 0;
    tcp->psh = 0;
    tcp->ack = 0;
    tcp->urg = 0;
    tcp->window = htons(14600);
    tcp->urg_ptr = 0;

    struct pseudo_header psh;
    psh.source_address = inet_addr(src_ip);
    psh.dest_address = sin.sin_addr.s_addr;
    psh.placeholder = 0;
    psh.protocol = IPPROTO_TCP;
    psh.tcp_length = htons(sizeof(struct tcphdr));

    memcpy(&psh.tcp, tcp, sizeof(struct tcphdr));
    // 计算checksum需要携带目的地址和源地址，否则发出去对面不认就不会回包
    tcp->check = csum((unsigned short *) &psh, sizeof(struct pseudo_header));

    int sent = sendto(sock_raw, packet, ip->tot_len, 0, (struct sockaddr *) &sin, sizeof(sin));
    if (sent < 0) {
        cout << "Error sending packet: " << strerror(errno) << endl;
        return 1;
    }

    char buffer[MAX_PACKET_SIZE];
    struct sockaddr_in saddr;
    int saddr_size = sizeof(saddr);
    int recv_len = 0;

    while (recv_len == 0) {
        recv_len = recvfrom(sock_raw, buffer, MAX_PACKET_SIZE, 0, (struct sockaddr *) &saddr, (socklen_t *) &saddr_size);
        if (recv_len < 0) {
            cout << "Error receiving packet: " << strerror(errno) << endl;
            return 1;
        }

        struct iphdr *ip = (struct iphdr *) buffer;
        if (ip->protocol == IPPROTO_TCP) {
            struct tcphdr *tcp = (struct tcphdr *) (buffer + ip->ihl * 4);
            if (tcp->source == htons(dst_port) && tcp->rst == 1) {
                cout << "Port " << dst_port << " is closed" << endl;
            } else if (tcp->source == htons(dst_port) && tcp->syn == 1 && tcp->ack == 1) {
                cout << "Port " << dst_port << " is open" << endl;
            }
        }
    }

    close(sock_raw);

    return 0;
}
```

**不带ip头的处理**

```cpp
int main(int argc, char *argv[]) {
    if (argc < 4) {
        cout << "Usage: " << argv[0] << " <source IP> <target IP> <target port>" << endl;
        return 1;
    }

    char *src_ip = argv[1];
    char *dst_ip = argv[2];
    int dst_port = atoi(argv[3]);

    int sock_raw = socket(AF_INET, SOCK_RAW, IPPROTO_TCP);
    if (sock_raw < 0) {
        cout << "Error creating socket: " << strerror(errno) << endl;
        return 1;
    }

    char packet[MAX_PACKET_SIZE];
    memset(packet, 0, MAX_PACKET_SIZE);

    struct tcphdr *tcp = (struct tcphdr *)packet;
    struct sockaddr_in sin;

    sin.sin_family = AF_INET;
    sin.sin_port = htons(dst_port);
    sin.sin_addr.s_addr = inet_addr(dst_ip);

    tcp->source = htons(1234);
    tcp->dest = htons(dst_port);
    tcp->seq = htonl(1105024978);
    tcp->ack_seq = 0;
    tcp->doff = 5;
    tcp->fin = 0;
    tcp->syn = 1;
    tcp->rst = 0;
    tcp->psh = 0;
    tcp->ack = 0;
    tcp->urg = 0;
    tcp->window = htons(14600);
    tcp->check = 0;
    tcp->urg_ptr = 0;

    struct pseudo_header psh;
    psh.source_address = inet_addr(src_ip);
    psh.dest_address = sin.sin_addr.s_addr;
    psh.placeholder = 0;
    psh.protocol = IPPROTO_TCP;
    psh.tcp_length = htons(sizeof(struct tcphdr));

    memcpy(&psh.tcp, tcp, sizeof(struct tcphdr));
    // 计算checksum需要携带目的地址和源地址，否则发出去对面不认就不会回包
    tcp->check = csum((unsigned short *)&psh, sizeof(struct pseudo_header));

    // 直接发送携带tcp头的数据即可，操作系统会自动拼接ip头，但是tcp的checksum必须对应上，不然对面不认
    int sent = sendto(sock_raw, tcp, sizeof(struct tcphdr), 0, (struct sockaddr *)&sin, sizeof(sin));
    if (sent < 0) {
        cout << "Error sending packet: " << strerror(errno) << endl;
        return 1;
    }

    char buffer[MAX_PACKET_SIZE];
    struct sockaddr_in saddr;
    int saddr_size = sizeof(saddr);
    int recv_len = 0;

    while (recv_len == 0) {
        recv_len = recvfrom(sock_raw, buffer, MAX_PACKET_SIZE, 0, (struct sockaddr *)&saddr, (socklen_t *)&saddr_size);
        if (recv_len < 0) {
            cout << "Error receiving packet: " << strerror(errno) << endl;
            return 1;
        }

        struct iphdr *ip = (struct iphdr *)buffer;
        if (ip->protocol == IPPROTO_TCP) {
            struct tcphdr *tcp = (struct tcphdr *)(buffer + ip->ihl * 4);
            if (tcp->source == htons(dst_port) && tcp->rst == 1) {
                cout << "Port " << dst_port << " is closed" << endl;
            } else if (tcp->source == htons(dst_port) && tcp->syn == 1 && tcp->ack == 1) {
                cout << "Port " << dst_port << " is open" << endl;
            }
        }
    }

    close(sock_raw);

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
2. 占用了不必要的资源
3. 在以数据包计费的网络上消耗额外的流量
