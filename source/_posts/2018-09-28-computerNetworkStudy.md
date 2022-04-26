---
title: 计算机网络学习
date: 2018-09-28 11:30:07
tags: [网络]
categories: [Knowledge, Study]
top: 94
---

# 一、网络OSI七层模型

<img src = "2018_09_28_01.jpg" width = "80%" />

## 分层介绍

### 应用层

- 网络服务与最终用户的一个接口。
- 协议有: HTTP FTP TFTP SMTP SNMP DNS TELNET HTTPS POP3 DHCP

### 表示层

- 数据的表示、安全、压缩。（在五层模型里面已经合并到了应用层）
- 格式有，JPEG、ASCll、DECOIC、加密格式等

### 会话层

- 建立、管理、终止会话。（在五层模型里面已经合并到了应用层）
- 对应主机进程，指本地主机与远程主机正在进行的会话

### 传输层

- 定义传输数据的协议端口号，以及流控和差错校验。
- 协议有: TCP UDP，数据包一旦离开网卡即进入网络传输层

### 网络层

- 进行逻辑地址寻址，实现不同网络之间的路径选择。ip包传输，不带端口。
- 协议有: ICMP IGMP IP（IPV4 IPV6） ARP RARP

### 数据链路层

- 建立逻辑连接、进行硬件地址寻址、差错校验 [2]  等功能。（由底层网络定义协议）
- 将比特组合成字节进而组合成帧，用MAC地址访问介质，错误发现但不能纠正。

### 物理层

- 建立、维护、断开物理连接。（由底层网络定义协议）

## 备注

- TCP/IP 层级模型结构，应用层之间的协议通过逐级调用传输层（Transport layer）、网络层（Network Layer）和物理数据链路层（Physical Data Link）而可以实现应用层的应用程序通信互联。
- 应用层需要关心应用程序的逻辑细节，而不是数据在网络中的传输活动。应用层其下三层则处理真正的通信细节。在 Internet 整个发展过程中的所有思想和着重点都以一种称为 RFC（Request For Comments）的文档格式存在。针对每一种特定的 TCP/IP 应用，有相应的 RFC [3]  文档。
- 一些典型的 TCP/IP 应用有 FTP、Telnet、SMTP、SNTP、REXEC、TFTP、LPD、SNMP、NFS、INETD 等。RFC 使一些基本相同的 TCP/IP 应用程序实现了标准化，从而使得不同厂家开发的应用程序可以互相通信

# 二、传输层

- tcp和udp都是ip包的数据段，分片包协议属于ip网络层协议，非传输层

## 1. TCP

### 1.1. 建立TCP连接: 三次握手协议

- 客户端: 我要对你讲话，你能听到吗；
- 服务端: 我能听到；而且我也要对你讲话，你能听到吗；
- 客户端: 我也能听到。
...
互相开始通话
...


### 1.2. 关闭TCP连接: 四次握手协议

- 客户端: 我说完了，我要闭嘴了；
- 服务端: 我收到请求，我要闭耳朵了；
（客户端收到这个确认，于是安心地闭嘴了。）
...
服务端还没倾诉完自己的故事，于是继续唠唠叨叨向客户端说了半天，直到说完为止
...
- 服务端: 我说完了，我也要闭嘴了；
- 客户端: 我收到请求，我要闭耳朵了；（事实上，客户端为了保证这个确认包成功送达，等待了两个最大报文生命周期后，才闭上耳朵。）
（服务端收到这个确认，于是安心地闭嘴了。）

#### 1) 问题

1. 客户端收到请求包后，为什么要等待两个最大报文生命周期后，才闭上耳朵呢？
    - 为了以防万一，因为最后一个发往服务端B的确认包有可能丢失。若丢失，服务端这里过了响应超时时间timeOut，会再次往客户端A发送关闭连接请求，这时候客户端得保证自己还没闭上耳朵，还能接收请求才行。
    - 服务端B再次发送的请求包到达客户端A时间，绝不会超过最大报文生命周期。
    - 这里的问题是，到底上面服务端的是如何判断超时的（我不是很清楚），假如是以自己发送请求时刻开始计时，半天未应答，为超时，那么：
    - 从服务端B发送请求包的时刻开始算，经过( TimeOut + 最大报文生命周期 )后，A必须还能接收数据包。
    - 那么A需要等待的时间是: ( TimeOut + 最大报文生命周期 ) - （上一个关闭l请求包从B发送到A的时长）。
    - 网上这块儿都讲得很模糊，一般就是说到A需要等待( TimeOut + 最大报文生命周期 ) < 2 * 最大报文生命周期，所以等待2 * 最大报文生命周期可以确保万无一失。
    - 事实上，这里关键需要搞清楚服务端的是如何判断超时的，我不是很清楚。但是假如是以自己发送请求的时刻开始计时，那么TimeOut应该是一个往返的最大时间吧，你们确定一个“请求-应答”往返的最大时间小于最大报文生命周期。
    - 当然，所有地方都是说要等待 2 * 最大生命周期，虽然没具体搞明白，但是我也同样相信。只是，网上的各种解释，都解析的模模糊糊，而且好多地方从逻辑上都不能完全说通诶，对那些解释，我没法完全相信。

### 1.3. tcp头部分析

**结构图**

<img src="2022-04-19-06.png" />

**wireshark抓包**

<img src="2022-04-19-07.png" />

**协议头文件定义**

```cpp
// <netinet/tcp.h>

/*
 * TCP header.
 * Per RFC 793, September, 1981.
 */
struct tcphdr
  {
    __extension__ union
    {
      struct
      {
	uint16_t th_sport;	/* source port */
	uint16_t th_dport;	/* destination port */
	tcp_seq th_seq;		/* sequence number */
	tcp_seq th_ack;		/* acknowledgement number */
# if __BYTE_ORDER == __LITTLE_ENDIAN
	uint8_t th_x2:4;	/* (unused) */
	uint8_t th_off:4;	/* data offset */
# endif
# if __BYTE_ORDER == __BIG_ENDIAN
	uint8_t th_off:4;	/* data offset */
	uint8_t th_x2:4;	/* (unused) */
# endif
	uint8_t th_flags;
# define TH_FIN	0x01
# define TH_SYN	0x02
# define TH_RST	0x04
# define TH_PUSH	0x08
# define TH_ACK	0x10
# define TH_URG	0x20
	uint16_t th_win;	/* window */
	uint16_t th_sum;	/* checksum */
	uint16_t th_urp;	/* urgent pointer */
      };
      struct
      {
	uint16_t source;
	uint16_t dest;
	uint32_t seq;
	uint32_t ack_seq;
# if __BYTE_ORDER == __LITTLE_ENDIAN
	uint16_t res1:4;    // uint16低8bit在前（0x50），对应低8bit的低4bit（0000）
	uint16_t doff:4;    // uint16低8bit在前（0x50），对应低8bit的高4bit（0101）
	uint16_t fin:1;     // uint16高8bit在后（0x10），对应高8bit的倒数第1位（0）
	uint16_t syn:1;     // uint16高8bit在后（0x10），对应高8bit的倒数第2位（0）
	uint16_t rst:1;     // uint16高8bit在后（0x10），对应高8bit的倒数第3位（0）
	uint16_t psh:1;     // uint16高8bit在后（0x10），对应高8bit的倒数第4位（0）
	uint16_t ack:1;     // uint16高8bit在后（0x10），对应高8bit的倒数第5位（1）
	uint16_t urg:1;     // uint16高8bit在后（0x10），对应高8bit的倒数第6位（0）
	uint16_t res2:2;    // uint16高8bit在后（0x10），对应高8bit的倒数第7,8位（0）
# elif __BYTE_ORDER == __BIG_ENDIAN
	uint16_t doff:4;
	uint16_t res1:4;
	uint16_t res2:2;
	uint16_t urg:1;
	uint16_t ack:1;
	uint16_t psh:1;
	uint16_t rst:1;
	uint16_t syn:1;
	uint16_t fin:1;
# else
#  error "Adjust your <bits/endian.h> defines"
# endif
	uint16_t window;
	uint16_t check;
	uint16_t urg_ptr;
      };
    };
};
```

## 2. UDP

### 2.1. UDP头部分析

**结构图**

<img src="2022-04-20-08.png" />

**wireshark抓包**

<img src="2022-04-20-09.png" />

**协议头文件定义**

```cpp
// netinet/udp.h

/* UDP header as specified by RFC 768, August 1980. */

struct udphdr
{
  __extension__ union
  {
    struct
    {
      uint16_t uh_sport;	/* source port */
      uint16_t uh_dport;	/* destination port */
      uint16_t uh_ulen;		/* udp length */
      uint16_t uh_sum;		/* udp checksum */
    };
    struct
    {
      uint16_t source;
      uint16_t dest;
      uint16_t len;
      uint16_t check;
    };
  };
};
```

# 三、网络层

## 1. ip地址分类

<img src="2019_10_14_02.png" width="80%" />

IP地址根据网络号和主机号来分，分为A、B、C三类及特殊地址D、E。全0和全1的都保留不用。

- A类: (1.0.0.0-126.0.0.0)（默认子网掩码: 255.0.0.0或 0xFF000000）第一个字节为网络号，后三个字节为主机号。该类IP地址的最前面为“0”，所以地址的网络号取值于1~126之间。一般用于大型网络。
- B类: (128.0.0.0-191.255.0.0)（默认子网掩码: 255.255.0.0或0xFFFF0000）前两个字节为网络号，后两个字节为主机号。该类IP地址的最前面为“10”，所以地址的网络号取值于128~191之间。一般用于中等规模网络。
- C类: (192.0.0.0-223.255.255.0)（子网掩码: 255.255.255.0或 0xFFFFFF00）前三个字节为网络号，最后一个字节为主机号。该类IP地址的最前面为“110”，所以地址的网络号取值于192~223之间。一般用于小型网络。
- D类: 是多播地址。该类IP地址的最前面为“1110”，所以地址的网络号取值于224~239之间。一般用于多路广播用户[1]  。
- E类: 是保留地址。该类IP地址的最前面为“1111”，所以地址的网络号取值于240~255之间。

## 2. ip包分析

### 2.1. ipv4包

**结构图**

<img src="2022-03-15-03.png" />

**协议头文件定义**

```cpp
// netinet/ip.h

struct iphdr
  {
// 主机字节序是小端，就在第一个字节后4位
#if __BYTE_ORDER == __LITTLE_ENDIAN
    unsigned int ihl:4;         // 0:0-3
    unsigned int version:4;     // 0:4-7
#elif __BYTE_ORDER == __BIG_ENDIAN
    unsigned int version:4;
    unsigned int ihl:4;
#else
# error	"Please fix <bits/endian.h>"
#endif
    uint8_t tos;        // 1
    uint16_t tot_len;   // 2-3
    uint16_t id;        // 4-5
    uint16_t frag_off;  // 6-7
    uint8_t ttl;        // 8
    uint8_t protocol;   // 9
    uint16_t check;     // 10-11
    uint32_t saddr;     // 12-15
    uint32_t daddr;     // 16-19
    /*The options start here. */
  };

// 下面是__USE_MISC定义的
struct ip
  {
#if __BYTE_ORDER == __LITTLE_ENDIAN
    unsigned int ip_hl:4;		/* header length */
    unsigned int ip_v:4;		/* version */
#endif
#if __BYTE_ORDER == __BIG_ENDIAN
    unsigned int ip_v:4;		/* version */
    unsigned int ip_hl:4;		/* header length */
#endif
    uint8_t ip_tos;			/* type of service */
    unsigned short ip_len;		/* total length */
    unsigned short ip_id;		/* identification */
    unsigned short ip_off;		/* fragment offset field */
#define	IP_RF 0x8000			/* reserved fragment flag */
#define	IP_DF 0x4000			/* dont fragment flag */
#define	IP_MF 0x2000			/* more fragments flag */
#define	IP_OFFMASK 0x1fff		/* mask for fragmenting bits */
    uint8_t ip_ttl;			/* time to live */
    uint8_t ip_p;			/* protocol */
    unsigned short ip_sum;		/* checksum */
    struct in_addr ip_src, ip_dst;	/* source and dest address */
  };
```

- 正好20个字节

**详细介绍**

- ihl: ip包头部长度，包括拓展字段长度
- version: 标识ipv4还是ipv6
- tos: 服务类型，只有在区分服务的时候才会用
- tot_len: ip包总长度
- id: 标识数据包的计数，每一个包，计数加一；分片包此数字一样
- frag_off: 分片包相关标记
  - `0x8000`: 保留未使用
  - `0x4000`: 不分片的flag，分片就是0，不分片为1
  - `0x2000`: 是否后面还有分片包的标志位
  - `0x1fff`: 标识分片包顺序的标志位
- ttl: 生存时间，经过每个路由器，TTL减去消耗的时间，当TTL为0，丢掉此数据包
- protocol: 标识协议类型，TCP、UDP、ICMP等，具体定义在`netinet/in.h`里面
- check: ip头校验和
- saddr: 源地址
- daddr: 目标地址

### 2.2. ipv6包

**结构图**

<img src="2022-03-29-04.png" />

**wireshark抓包**

<img src="2022-03-29-05.png" />

**协议头文件定义**

```cpp
// netinet/ip6.h

struct ip6_hdr
  {
    union
      {
    struct ip6_hdrctl
      {
        uint32_t ip6_un1_flow;   /* 4 bits version, 8 bits TC,
                    20 bits flow-ID */
        uint16_t ip6_un1_plen;   /* payload length */
        uint8_t  ip6_un1_nxt;    /* next header */
        uint8_t  ip6_un1_hlim;   /* hop limit */
      } ip6_un1;
    uint8_t ip6_un2_vfc;       /* 4 bits version, top 4 bits tclass */
      } ip6_ctlun;
    struct in6_addr ip6_src;      /* source address */
    struct in6_addr ip6_dst;      /* destination address */
  };
```

**详细介绍**

- `version`: 前4bit，ipv6就只有6
- `Traffic Class`: 紧跟的8bit
- `ip6_src`: 64bit源地址
- `ip6_dst`: 64bit目的地址

# 四、应用层

## 1. tls握手流程

参考自 [图解 HTTPS：RSA 握手过程](https://zhuanlan.zhihu.com/p/344086342)

# 问题

## 1. 局域网和外网？

### 一个局域网内共用一个外网ip，数据如何定位到自己的电脑上的？

- 通过端口定位
- 电脑通过路由器连接到外网时，会在路由器上映射一个nat表，nat运行在传输层，因为要解析端口
- 表中映射为`ip:port`，会将路由器上随机生成一个端口，并且将局域网ip加端口和外网ip加端口相互映射。
- 本地电脑通过这个映射表连上外网

如:
- 路由器外网ip: 115.156.207.252
- 电脑的局域网ip: 192.168.11.109

电脑连外网时，通过本机的`80`端口访问网页。路由器会随机生成一个端口号，比如`1234`。路由器的nat映射表就会有一条记录：`115.156.207.252:1234 <-> 192.168.11.109:80`，外网的数据就通过返回到`115.156.207.252:1234`这个地址来给到电脑的`80`端口上。
