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

# 二、物理层

# 三、数据链路层

# 四、网络层

## 1. ip地址分类

<img src="2019_10_14_02.png" width="80%" />

IP地址根据网络号和主机号来分，分为A、B、C三类及特殊地址D、E。全0和全1的都保留不用。

- A类: (1.0.0.0-126.0.0.0)（默认子网掩码: 255.0.0.0或 0xFF000000）第一个字节为网络号，后三个字节为主机号。该类IP地址的最前面为“0”，所以地址的网络号取值于1~126之间。一般用于大型网络。
- B类: (128.0.0.0-191.255.0.0)（默认子网掩码: 255.255.0.0或0xFFFF0000）前两个字节为网络号，后两个字节为主机号。该类IP地址的最前面为“10”，所以地址的网络号取值于128~191之间。一般用于中等规模网络。
- C类: (192.0.0.0-223.255.255.0)（子网掩码: 255.255.255.0或 0xFFFFFF00）前三个字节为网络号，最后一个字节为主机号。该类IP地址的最前面为“110”，所以地址的网络号取值于192~223之间。一般用于小型网络。
- D类: 是多播地址。该类IP地址的最前面为“1110”，所以地址的网络号取值于224~239之间。一般用于多路广播用户[1]  。
- E类: 是保留地址。该类IP地址的最前面为“1111”，所以地址的网络号取值于240~255之间。

## 2. ipv4

### 2.1. 包结构分析

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

## 3. ipv6

### 3.1. 包结构分析

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
- `ip6_un1_plen`: 应用数据包大小（不算ip头）
- `ip6_un1_nxt`: 协议（UDP、TCP等）
- `ip6_src`: 64bit源地址
- `ip6_dst`: 64bit目的地址

#### 1) ipv6分片包处理

- ipv6分片包和ipv4不一样，ipv4的是中间三层代理设备需要合包再根据mtu发送出去，重新分片，可能存在性能和安全性问题
- ipv4的ip头部是最少20个字节，里面有flags并且可能拓展一些数据，但是ipv6是固定长度40个字节，没有拓展数据，没有flags，ipv6的分片是通过拓展头的方式做分片

<img src="2024-08-09-01.png" />

- 解析ipv6分片包就是检查`next_header`和所有拓展头中的`next_header`是否有44，一般分片包里面的`next_header`就是传输层协议，找到对应传输层协议的`next_header`的拓展头，下一个payload就是传输层数据

### 3.2. ipv6的地址分类

<img src="2024-08-16-01.png" />

参考: [ipv6地址、单薄、多播、组播介绍](https://blog.csdn.net/m0_55708805/article/details/115798614)

#### 1) 单播地址 unicast

单播就是单点访问，此ip对应一个设备，不会发给其他设备

##### (1) Global Unicast Address 全球单播地址

可以理解为公网地址，全球唯一，不会重复，一般为`2000::/3`

##### (2) Link-Local Unicast

局域网内的单播地址，类似ipv4的私网ip一般取`192.168.0.1/24`等，ipv6的局域网单播地址为`fe80::/10`

#### 2) 多播地址 multicast


# 五、传输层

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

### 1.4. tcp校验和的计算

- 硬件有checksum offload能力时，协议栈只需要计算一个伪头部校验和，放到tcp->check中，然后设置好csum_start（硬件需要计算checksum的起始位置即tcp头部起始位置），csum_offset（计算完成后存放的位置），硬件就会自动计算
- 协议栈里面处理tcp校验和的代码

```cpp
// net/ipv4/tcp_ipv4.c
void __tcp_v4_send_check(struct sk_buff *skb, __be32 saddr, __be32 daddr)
{
	struct tcphdr *th = tcp_hdr(skb);

  // 伪头部校验和包括tcp包整个长度，源ip和目的ip
	th->check = ~tcp_v4_check(skb->len, saddr, daddr, 0);
	skb->csum_start = skb_transport_header(skb) - skb->head;  // tcp头部起始位置，基于head的偏移
	skb->csum_offset = offsetof(struct tcphdr, check);  // check基于tcp头部起始位置的偏移
}

/* This routine computes an IPv4 TCP checksum. */
void tcp_v4_send_check(struct sock *sk, struct sk_buff *skb)
{
	const struct inet_sock *inet = inet_sk(sk);

	__tcp_v4_send_check(skb, inet->inet_saddr, inet->inet_daddr);
}
EXPORT_SYMBOL(tcp_v4_send_check);
```

- 伪头部包含源ip和目的ip以及tcp整个包长度，目的是让接收端进行校验，确定tcp构造头部时的源地址和目的地址和数据包是一致的

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

# 六、应用层

## 1. tls握手流程

参考自 [图解 HTTPS：RSA 握手过程](https://zhuanlan.zhihu.com/p/344086342)

## 2. http

### 2.1. http header

#### 1) General部分

| Header          | 解释                                     | 示例                                                                 |
| --------------- | ---------------------------------------- | -------------------------------------------------------------------- |
| Request URL     | 请求的url                                | https://www.baidu.com/home/msg/data/personalcontent?callback=jQuery1 |
| Request Method  | 请求的方法，可以是GET、POST              | GET                                                                  |
| Status Code     | [HTTP 状态码](#statusCode)，表示请求成功 | 200 OK                                                               |
| Remote Address  | 远程IP地址                               | 119.75.213.61:443                                                    |
| Referrer Policy | 见下文                                   | unsafe-url                                                           |

##### Referrer Policy States

新的Referrer规定了五种策略：
- No Referrer：任何情况下都不发送Referrer信息
- No Referrer When Downgrade：仅当协议降级（如HTTPS页面引入HTTP资源）时不发送Referrer信息。是大部分浏览器默认策略。
- Origin Only：发送只包含host部分的referrer.
- Origin When Cross-origin：仅在发生跨域访问时发送只包含host的Referer，同域下还是完整的。与Origin Only的区别是多判断了是否Cross-origin。协议、域名和端口都一致，浏览器才认为是同域。
- Unsafe URL：全部都发送Referrer信息。最宽松最不安全的策略。

#### 2) Requests部分

| Header              | 解释                                                                                      | 示例                                                    |
| ------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| Accept              | 指定客户端能够接收的内容类型                                                              | Accept: text/plain, text/html                           |
| Accept-Charset      | 浏览器可以接受的字符编码集。                                                              | Accept-Charset: iso-8859-5                              |
| Accept-Encoding     | 指定浏览器可以支持的web服务器返回内容压缩编码类型。                                       | Accept-Encoding: compress, gzip                         |
| Accept-Language     | 浏览器可接受的语言                                                                        | Accept-Language: en,zh                                  |
| Accept-Ranges       | 可以请求网页实体的一个或者多个子范围字段                                                  | Accept-Ranges: bytes                                    |
| Authorization       | HTTP授权的授权证书                                                                        | Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==       |
| Cache-Control       | 指定请求和响应遵循的缓存机制                                                              | Cache-Control: no-cache                                 |
| Connection          | 表示是否需要持久连接。（HTTP 1.1默认进行持久连接）                                        | Connection: close                                       |
| Cookie              | HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。                 | Cookie: $Version=1; Skin=new;                           |
| Content-Length      | 请求的内容长度                                                                            | Content-Length: 348                                     |
| Content-Type        | 请求的与实体对应的MIME信息                                                                | Content-Type: application/x-www-form-urlencoded         |
| Date                | 请求发送的日期和时间                                                                      | Date: Tue, 15 Nov 2010 08:12:31 GMT                     |
| Expect              | 请求的特定的服务器行为                                                                    | Expect: 100-continue                                    |
| From                | 发出请求的用户的Email                                                                     | From: user@email.com                                    |
| Host                | 指定请求的服务器的域名和端口号                                                            | Host: www.zcmhi.com                                     |
| If-Match            | 只有请求内容与实体相匹配才有效                                                            | If-Match: “737060cd8c284d8af7ad3082f209582d”            |
| If-Modified-Since   | 如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码                       | If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT        |
| If-None-Match       | 如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变 | If-None-Match: “737060cd8c284d8af7ad3082f209582d”       |
| If-Range            | 如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag                | If-Range: “737060cd8c284d8af7ad3082f209582d”            |
| If-Unmodified-Since | 只在实体在指定时间之后未被修改才请求成功                                                  | If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT      |
| Max-Forwards        | 限制信息通过代理和网关传送的时间                                                          | Max-Forwards: 10                                        |
| Pragma              | 用来包含实现特定的指令                                                                    | Pragma: no-cache                                        |
| Proxy-Authorization | 连接到代理的授权证书                                                                      | Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ== |
| Range               | 只请求实体的一部分，指定范围                                                              | Range: bytes=500-999                                    |
| Referer             | 先前网页的地址，当前请求网页紧随其后,即来路                                               | Referer: http://www.zcmhi.com/archives/71.html          |
| TE                  | 客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息                                  | TE: trailers,deflate;q=0.5                              |
| Upgrade             | 向服务器指定某种传输协议以便服务器进行转换（如果支持）                                    | Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11          |
| User-Agent          | User-Agent的内容包含发出请求的用户信息                                                    | User-Agent: Mozilla/5.0 (Linux; X11)                    |
| Via                 | 通知中间网关或代理服务器地址，通信协议                                                    | Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)             |
| Warning             | 关于消息实体的警告信息                                                                    | Warn: 199 Miscellaneous warning                         |

#### 3) Responses 部分

| Header             | 解释                                                                                | 示例                                                  |
| ------------------ | ----------------------------------------------------------------------------------- | ----------------------------------------------------- |
| Accept-Ranges      | 表明服务器是否支持指定范围请求及哪种类型的分段请求                                  | Accept-Ranges: bytes                                  |
| Age                | 从原始服务器到代理缓存形成的估算时间（以秒计，非负）                                | Age: 12                                               |
| Allow              | 对某网络资源的有效的请求行为，不允许则返回405                                       | Allow: GET, HEAD                                      |
| Cache-Control      | 告诉所有的缓存机制是否可以缓存及哪种类型                                            | Cache-Control: no-cache                               |
| Content-Encoding   | web服务器支持的返回内容压缩编码类型。                                               | Content-Encoding: gzip                                |
| Content-Language   | 响应体的语言                                                                        | Content-Language: en,zh                               |
| Content-Length     | 响应体的长度                                                                        | Content-Length: 348                                   |
| Content-Location   | 请求资源可替代的备用的另一地址                                                      | Content-Location: /index.htm                          |
| Content-MD5        | 返回资源的MD5校验值                                                                 | Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==                 |
| Content-Range      | 在整个返回体中本部分的字节位置                                                      | Content-Range: bytes 21010-47021/47022                |
| Content-Type       | 返回内容的MIME类型                                                                  | Content-Type: text/html; charset=utf-8                |
| Date               | 原始服务器消息发出的时间                                                            | Date: Tue, 15 Nov 2010 08:12:31 GMT                   |
| ETag               | 请求变量的实体标签的当前值                                                          | ETag: “737060cd8c284d8af7ad3082f209582d”              |
| Expires            | 响应过期的日期和时间                                                                | Expires: Thu, 01 Dec 2010 16:00:00 GMT                |
| Last-Modified      | 请求资源的最后修改时间                                                              | Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT          |
| Location           | 用来重定向接收方到非请求URL的位置来完成请求或标识新的资源                           | Location: http://www.zcmhi.com/archives/94.html       |
| Pragma             | 包括实现特定的指令，它可应用到响应链上的任何接收方                                  | Pragma: no-cache                                      |
| Proxy-Authenticate | 它指出认证方案和可应用到代理的该URL上的参数                                         | Proxy-Authenticate: Basic                             |
| refresh            | 应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持） | Refresh: 5; url=http://www.zcmhi.com/archives/94.html |
| Retry-After        | 如果实体暂时不可取，通知客户端在指定时间之后再次尝试                                | Retry-After: 120                                      |
| Server             | web服务器软件名称                                                                   | Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)          |
| Set-Cookie         | 设置Http Cookie                                                                     | Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1   |
| Trailer            | 指出头域在分块传输编码的尾部存在                                                    | Trailer: Max-Forwards                                 |
| Transfer-Encoding  | 文件传输编码                                                                        | Transfer-Encoding:chunked                             |
| Vary               | 告诉下游代理是使用缓存响应还是从原始服务器请求                                      | Vary: *                                               |
| Via                | 告知代理客户端响应是通过哪里发送的                                                  | Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)           |
| Warning            | 警告实体可能存在的问题                                                              | Warning: 199 Miscellaneous warning                    |
| WWW-Authenticate   | 表明客户端请求实体应该使用的授权方案                                                | WWW-Authenticate: Basic                               |

### 2.2. <span id = "statusCode">http状态码记录</span>

#### 1xx 消息

这一类型的状态码，代表请求已被接受，需要继续处理。这类响应是临时响应，只包含状态行和某些可选的响应头信息，并以空行结束。由于 HTTP/1.0 协议中没有定义任何 1xx 状态码，所以除非在某些试验条件下，服务器禁止向此类客户端发送 1xx 响应。

##### 100 Continue

客户端应当继续发送请求。这个临时响应是用来通知客户端它的部分请求已经被服务器接收，且仍未被拒绝。客户端应当继续发送请求的剩余部分，或者如果请求已经完成，忽略这个响应。服务器必须在请求完成后向客户端发送一个最终响应。

##### 101 Switching Protocols

服务器已经理解了客户端的请求，并将通过Upgrade 消息头通知客户端采用不同的协议来完成这个请求。在发送完这个响应最后的空行后，服务器将会切换到在Upgrade 消息头中定义的那些协议。

只有在切换新的协议更有好处的时候才应该采取类似措施。例如，切换到新的HTTP 版本比旧版本更有优势，或者切换到一个实时且同步的协议以传送利用此类特性的资源。

##### 102 Processing

由WebDAV（RFC 2518）扩展的状态码，代表处理将被继续执行。

#### 2xx 成功

这一类型的状态码，代表请求已成功被服务器接收、理解、并接受。

##### 200 OK

请求已成功，请求所希望的响应头或数据体将随此响应返回。出现此状态码是表示正常状态。

##### 201 Created

请求已经被实现，而且有一个新的资源已经依据请求的需要而建立，且其 URI 已经随Location 头信息返回。假如需要的资源无法及时建立的话，应当返回 '202 Accepted'。

##### 202 Accepted

服务器已接受请求，但尚未处理。正如它可能被拒绝一样，最终该请求可能会也可能不会被执行。在异步操作的场合下，没有比发送这个状态码更方便的做法了。

返回202状态码的响应的目的是允许服务器接受其他过程的请求（例如某个每天只执行一次的基于批处理的操作），而不必让客户端一直保持与服务器的连接直到批处理操作全部完成。在接受请求处理并返回202状态码的响应应当在返回的实体中包含一些指示处理当前状态的信息，以及指向处理状态监视器或状态预测的指针，以便用户能够估计操作是否已经完成。

##### 203 Non-Authoritative Information

服务器已成功处理了请求，但返回的实体头部元信息不是在原始服务器上有效的确定集合，而是来自本地或者第三方的拷贝。当前的信息可能是原始版本的子集或者超集。例如，包含资源的元数据可能导致原始服务器知道元信息的超集。使用此状态码不是必须的，而且只有在响应不使用此状态码便会返回200 OK的情况下才是合适的。

##### 204 No Content

服务器成功处理了请求，但不需要返回任何实体内容，并且希望返回更新了的元信息。响应可能通过实体头部的形式，返回新的或更新后的元信息。如果存在这些头部信息，则应当与所请求的变量相呼应。

如果客户端是浏览器的话，那么用户浏览器应保留发送了该请求的页面，而不产生任何文档视图上的变化，即使按照规范新的或更新后的元信息应当被应用到用户浏览器活动视图中的文档。

由于204响应被禁止包含任何消息体，因此它始终以消息头后的第一个空行结尾。

##### 205 Reset Content

服务器成功处理了请求，且没有返回任何内容。但是与204响应不同，返回此状态码的响应要求请求者重置文档视图。该响应主要是被用于接受用户输入后，立即重置表单，以便用户能够轻松地开始另一次输入。

与204响应一样，该响应也被禁止包含任何消息体，且以消息头后的第一个空行结束。

##### 206 Partial Content

服务器已经成功处理了部分 GET 请求。类似于 FlashGet 或者迅雷这类的 HTTP下载工具都是使用此类响应实现断点续传或者将一个大文档分解为多个下载段同时下载。

该请求必须包含 Range 头信息来指示客户端希望得到的内容范围，并且可能包含 If-Range 来作为请求条件。

响应必须包含如下的头部域：

- Content-Range 用以指示本次响应中返回的内容的范围；如果是 Content-Type 为 multipart/byteranges 的多段下载，则每一 multipart 段中都应包含 Content-Range 域用以指示本段的内容范围。假如响应中包含 Content-Length，那么它的数值必须匹配它返回的内容范围的真实字节数。
- Date
- ETag 和/或 Content-Location，假如同样的请求本应该返回200响应。
- Expires, Cache-Control，和/或 Vary，假如其值可能与之前相同变量的其他响应对应的值不同的话。
假如本响应请求使用了 If-Range 强缓存验证，那么本次响应不应该包含其他实体头；假如本响应的请求使用了 If-Range 弱缓存验证，那么本次响应禁止包含其他实体头；这避免了缓存的实体内容和更新了的实体头信息之间的不一致。否则，本响应就应当包含所有本应该返回200响应中应当返回的所有实体头部域。

假如 ETag 或 Last-Modified 头部不能精确匹配的话，则客户端缓存应禁止将206响应返回的内容与之前任何缓存过的内容组合在一起。

##### 207 Multi-Status

由WebDAV(RFC 2518)扩展的状态码，代表之后的消息体将是一个XML消息，并且可能依照之前子请求数量的不同，包含一系列独立的响应代码。

#### 3xx 重定向

这类状态码代表需要客户端采取进一步的操作才能完成请求。通常，这些状态码用来重定向，后续的请求地址（重定向目标）在本次响应的 Location 域中指明。

当且仅当后续的请求所使用的方法是 GET 或者 HEAD 时，用户浏览器才可以在没有用户介入的情况下自动提交所需要的后续请求。客户端应当自动监测无限循环重定向（例如：A->A，或者A->B->C->A），因为这会导致服务器和客户端大量不必要的资源消耗。按照 HTTP/1.0 版规范的建议，浏览器不应自动访问超过5次的重定向。

##### 300 Multiple Choices

被请求的资源有一系列可供选择的回馈信息，每个都有自己特定的地址和浏览器驱动的商议信息。用户或浏览器能够自行选择一个首选的地址进行重定向。

除非这是一个 HEAD 请求，否则该响应应当包括一个资源特性及地址的列表的实体，以便用户或浏览器从中选择最合适的重定向地址。这个实体的格式由 Content-Type 定义的格式所决定。浏览器可能根据响应的格式以及浏览器自身能力，自动作出最合适的选择。当然，RFC 2616规范并没有规定这样的自动选择该如何进行。

如果服务器本身已经有了首选的回馈选择，那么在 Location 中应当指明这个回馈的 URI；浏览器可能会将这个 Location 值作为自动重定向的地址。此外，除非额外指定，否则这个响应也是可缓存的。

##### 301 Moved Permanently

被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个 URI 之一。如果可能，拥有链接编辑功能的客户端应当自动把请求的地址修改为从服务器反馈回来的地址。除非额外指定，否则这个响应也是可缓存的。

新的永久性的URI 应当在响应的 Location 域中返回。除非这是一个 HEAD 请求，否则响应的实体中应当包含指向新的 URI 的超链接及简短说明。

如果这不是一个 GET 或者 HEAD 请求，因此浏览器禁止自动进行重定向，除非得到用户的确认，因为请求的条件可能因此发生变化。

注意：对于某些使用 HTTP/1.0 协议的浏览器，当它们发送的 POST 请求得到了一个301响应的话，接下来的重定向请求将会变成 GET 方式。

##### 302 Move temporarily

请求的资源临时从不同的 URI响应请求。由于这样的重定向是临时的，客户端应当继续向原有地址发送以后的请求。只有在Cache-Control或Expires中进行了指定的情况下，这个响应才是可缓存的。
上文有提及。

如果这不是一个 GET 或者 HEAD 请求，那么浏览器禁止自动进行重定向，除非得到用户的确认，因为请求的条件可能因此发生变化。

注意：虽然RFC 1945和RFC 2068规范不允许客户端在重定向时改变请求的方法，但是很多现存的浏览器将302响应视作为303响应，并且使用 GET 方式访问在 Location 中规定的 URI，而无视原先请求的方法。状态码303和307被添加了进来，用以明确服务器期待客户端进行何种反应。

##### 303 See Other

对应当前请求的响应可以在另一个 URL 上被找到，而且客户端应当采用 GET 的方式访问那个资源。这个方法的存在主要是为了允许由脚本激活的POST请求输出重定向到一个新的资源。这个新的 URI 不是原始资源的替代引用。同时，303响应禁止被缓存。当然，第二个请求（重定向）可能被缓存。

注意：许多 HTTP/1.1 版以前的浏览器不能正确理解303状态。如果需要考虑与这些浏览器之间的互动，302状态码应该可以胜任，因为大多数的浏览器处理302响应时的方式恰恰就是上述规范要求客户端处理303响应时应当做的。

##### 304 Not Modified

如果客户端发送了一个带条件的 GET 请求且该请求已被允许，而文档的内容（自上次访问以来或者根据请求的条件）并没有改变，则服务器应当返回这个状态码。304响应禁止包含消息体，因此始终以消息头后的第一个空行结尾。

该响应必须包含以下的头信息：

- Date，除非这个服务器没有时钟。假如没有时钟的服务器也遵守这些规则，那么代理服务器以及客户端可以自行将 Date 字段添加到接收到的响应头中去（正如RFC 2068中规定的一样），缓存机制将会正常工作。
- ETag 和/或 Content-Location，假如同样的请求本应返回200响应。
- Expires, Cache-Control，和/或Vary，假如其值可能与之前相同变量的其他响应对应的值不同的话。
假如本响应请求使用了强缓存验证，那么本次响应不应该包含其他实体头；否则（例如，某个带条件的 GET 请求使用了弱缓存验证），本次响应禁止包含其他实体头；这避免了缓存了的实体内容和更新了的实体头信息之间的不一致。

假如某个304响应指明了当前某个实体没有缓存，那么缓存系统必须忽视这个响应，并且重复发送不包含限制条件的请求。

假如接收到一个要求更新某个缓存条目的304响应，那么缓存系统必须更新整个条目以反映所有在响应中被更新的字段的值。

##### 305 Use Proxy

被请求的资源必须通过指定的代理才能被访问。Location 域中将给出指定的代理所在的 URI 信息，接收者需要重复发送一个单独的请求，通过这个代理才能访问相应资源。只有原始服务器才能建立305响应。

注意：RFC 2068中没有明确305响应是为了重定向一个单独的请求，而且只能被原始服务器建立。忽视这些限制可能导致严重的安全后果。

##### 306 Switch Proxy

在最新版的规范中，306状态码已经不再被使用。

##### 307 Temporary Redirect

请求的资源临时从不同的URI 响应请求。

新的临时性的URI 应当在响应的 Location 域中返回。除非这是一个HEAD 请求，否则响应的实体中应当包含指向新的URI 的超链接及简短说明。因为部分浏览器不能识别307响应，因此需要添加上述必要信息以便用户能够理解并向新的 URI 发出访问请求。

如果这不是一个GET 或者 HEAD 请求，那么浏览器禁止自动进行重定向，除非得到用户的确认，因为请求的条件可能因此发生变化。

#### 4xx 请求错误

这类的状态码代表了客户端看起来可能发生了错误，妨碍了服务器的处理。除非响应的是一个 HEAD 请求，否则服务器就应该返回一个解释当前错误状况的实体，以及这是临时的还是永久性的状况。这些状态码适用于任何请求方法。浏览器应当向用户显示任何包含在此类错误响应中的实体内容。

如果错误发生时客户端正在传送数据，那么使用TCP的服务器实现应当仔细确保在关闭客户端与服务器之间的连接之前，客户端已经收到了包含错误信息的数据包。如果客户端在收到错误信息后继续向服务器发送数据，服务器的TCP栈将向客户端发送一个重置数据包，以清除该客户端所有还未识别的输入缓冲，以免这些数据被服务器上的应用程序读取并干扰后者。

##### 400 Bad Request

1. 语义有误，当前请求无法被服务器理解。除非进行修改，否则客户端不应该重复提交这个请求。
2. 请求参数有误。

##### 401 Unauthorized

当前请求需要用户验证。该响应必须包含一个适用于被请求资源的 WWW-Authenticate 信息头用以询问用户信息。客户端可以重复提交一个包含恰当的 Authorization 头信息的请求。如果当前请求已经包含了 Authorization 证书，那么401响应代表着服务器验证已经拒绝了那些证书。如果401响应包含了与前一个响应相同的身份验证询问，且浏览器已经至少尝试了一次验证，那么浏览器应当向用户展示响应中包含的实体信息，因为这个实体信息中可能包含了相关诊断信息。参见RFC 2617。

##### 402 Payment Required

该状态码是为了将来可能的需求而预留的。

##### 403 Forbidden

服务器已经理解请求，但是拒绝执行它。与401响应不同的是，身份验证并不能提供任何帮助，而且这个请求也不应该被重复提交。如果这不是一个 HEAD 请求，而且服务器希望能够讲清楚为何请求不能被执行，那么就应该在实体内描述拒绝的原因。当然服务器也可以返回一个404响应，假如它不希望让客户端获得任何信息。

##### 404 Not Found

请求失败，请求所希望得到的资源未被在服务器上发现。没有信息能够告诉用户这个状况到底是暂时的还是永久的。假如服务器知道情况的话，应当使用410状态码来告知旧资源因为某些内部的配置机制问题，已经永久的不可用，而且没有任何可以跳转的地址。404这个状态码被广泛应用于当服务器不想揭示到底为何请求被拒绝或者没有其他适合的响应可用的情况下。出现这个错误的最有可能的原因是服务器端没有这个页面。

##### 405 Method Not Allowed

请求行中指定的请求方法不能被用于请求相应的资源。该响应必须返回一个Allow 头信息用以表示出当前资源能够接受的请求方法的列表。

鉴于 PUT，DELETE 方法会对服务器上的资源进行写操作，因而绝大部分的网页服务器都不支持或者在默认配置下不允许上述请求方法，对于此类请求均会返回405错误。

##### 406 Not Acceptable

请求的资源的内容特性无法满足请求头中的条件，因而无法生成响应实体。

除非这是一个 HEAD 请求，否则该响应就应当返回一个包含可以让用户或者浏览器从中选择最合适的实体特性以及地址列表的实体。实体的格式由 Content-Type 头中定义的媒体类型决定。浏览器可以根据格式及自身能力自行作出最佳选择。但是，规范中并没有定义任何作出此类自动选择的标准。

##### 407 Proxy Authentication Required

与401响应类似，只不过客户端必须在代理服务器上进行身份验证。代理服务器必须返回一个 Proxy-Authenticate 用以进行身份询问。客户端可以返回一个 Proxy-Authorization 信息头用以验证。参见RFC 2617。

##### 408 Request Timeout

请求超时。客户端没有在服务器预备等待的时间内完成一个请求的发送。客户端可以随时再次提交这一请求而无需进行任何更改。

##### 409 Conflict

由于和被请求的资源的当前状态之间存在冲突，请求无法完成。这个代码只允许用在这样的情况下才能被使用：用户被认为能够解决冲突，并且会重新提交新的请求。该响应应当包含足够的信息以便用户发现冲突的源头。

冲突通常发生于对 PUT 请求的处理中。例如，在采用版本检查的环境下，某次 PUT 提交的对特定资源的修改请求所附带的版本信息与之前的某个（第三方）请求向冲突，那么此时服务器就应该返回一个409错误，告知用户请求无法完成。此时，响应实体中很可能会包含两个冲突版本之间的差异比较，以便用户重新提交归并以后的新版本。

##### 410 Gone

被请求的资源在服务器上已经不再可用，而且没有任何已知的转发地址。这样的状况应当被认为是永久性的。如果可能，拥有链接编辑功能的客户端应当在获得用户许可后删除所有指向这个地址的引用。如果服务器不知道或者无法确定这个状况是否是永久的，那么就应该使用404状态码。除非额外说明，否则这个响应是可缓存的。

410响应的目的主要是帮助网站管理员维护网站，通知用户该资源已经不再可用，并且服务器拥有者希望所有指向这个资源的远端连接也被删除。这类事件在限时、增值服务中很普遍。同样，410响应也被用于通知客户端在当前服务器站点上，原本属于某个个人的资源已经不再可用。当然，是否需要把所有永久不可用的资源标记为'410 Gone'，以及是否需要保持此标记多长时间，完全取决于服务器拥有者。

##### 411 Length Required

服务器拒绝在没有定义 Content-Length 头的情况下接受请求。在添加了表明请求消息体长度的有效 Content-Length 头之后，客户端可以再次提交该请求。

##### 412 Precondition Failed

服务器在验证在请求的头字段中给出先决条件时，没能满足其中的一个或多个。这个状态码允许客户端在获取资源时在请求的元信息（请求头字段数据）中设置先决条件，以此避免该请求方法被应用到其希望的内容以外的资源上。

##### 413 Request Entity Too Large

服务器拒绝处理当前请求，因为该请求提交的实体数据大小超过了服务器愿意或者能够处理的范围。此种情况下，服务器可以关闭连接以免客户端继续发送此请求。

如果这个状况是临时的，服务器应当返回一个 Retry-After 的响应头，以告知客户端可以在多少时间以后重新尝试。

##### 414 Request-URI Too Long

请求的URI 长度超过了服务器能够解释的长度，因此服务器拒绝对该请求提供服务。这比较少见，通常的情况包括：

本应使用POST方法的表单提交变成了GET方法，导致查询字符串（Query String）过长。

重定向URI “黑洞”，例如每次重定向把旧的 URI 作为新的 URI 的一部分，导致在若干次重定向后 URI 超长。

客户端正在尝试利用某些服务器中存在的安全漏洞攻击服务器。这类服务器使用固定长度的缓冲读取或操作请求的 URI，当 GET 后的参数超过某个数值后，可能会产生缓冲区溢出，导致任意代码被执行[1]。没有此类漏洞的服务器，应当返回414状态码。

##### 415 Unsupported Media Type

对于当前请求的方法和所请求的资源，请求中提交的实体并不是服务器中所支持的格式，因此请求被拒绝。

##### 416 Requested Range Not Satisfiable

如果请求中包含了 Range 请求头，并且 Range 中指定的任何数据范围都与当前资源的可用范围不重合，同时请求中又没有定义 If-Range 请求头，那么服务器就应当返回416状态码。

假如 Range 使用的是字节范围，那么这种情况就是指请求指定的所有数据范围的首字节位置都超过了当前资源的长度。服务器也应当在返回416状态码的同时，包含一个 Content-Range 实体头，用以指明当前资源的长度。这个响应也被禁止使用 multipart/byteranges 作为其 Content-Type。

##### 417 Expectation Failed

在请求头 Expect 中指定的预期内容无法被服务器满足，或者这个服务器是一个代理服务器，它有明显的证据证明在当前路由的下一个节点上，Expect 的内容无法被满足。
##### 418 I'm a teapot

##### 421 too many connections

There are too many connections from your internet address

从当前客户端所在的IP地址到服务器的连接数超过了服务器许可的最大范围。通常，这里的IP地址指的是从服务器上看到的客户端地址（比如用户的网关或者代理服务器地址）。在这种情况下，连接数的计算可能涉及到不止一个终端用户。

##### 422 Unprocessable Entity

请求格式正确，但是由于含有语义错误，无法响应。（RFC 4918 WebDAV）

##### 423 Locked

当前资源被锁定。（RFC 4918 WebDAV）

##### 424 Failed Dependency

由于之前的某个请求发生的错误，导致当前请求失败，例如 PROPPATCH。（RFC 4918 WebDAV）

##### 425 Unordered Collection

在WebDav Advanced Collections 草案中定义，但是未出现在《WebDAV 顺序集协议》（RFC 3658）中。

##### 426 Upgrade Required

客户端应当切换到TLS/1.0。（RFC 2817）

##### 449 Retry With

由微软扩展，代表请求应当在执行完适当的操作后进行重试。

##### 451 Unavailable For Legal Reasons

该请求因法律原因不可用。（RFC 7725）

#### 5xx、6xx 服务器错误

这类状态码代表了服务器在处理请求的过程中有错误或者异常状态发生，也有可能是服务器意识到以当前的软硬件资源无法完成对请求的处理。除非这是一个HEAD 请求，否则服务器应当包含一个解释当前错误状态以及这个状况是临时的还是永久的解释信息实体。浏览器应当向用户展示任何在当前响应中被包含的实体。

这些状态码适用于任何响应方法。

##### 500 Internal Server Error

服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。一般来说，这个问题都会在服务器端的源代码出现错误时出现。

##### 501 Not Implemented

服务器不支持当前请求所需要的某个功能。当服务器无法识别请求的方法，并且无法支持其对任何资源的请求。

##### 502 Bad Gateway

作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。

##### 503 Service Unavailable

由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是临时的，并且将在一段时间以后恢复。如果能够预计延迟时间，那么响应中可以包含一个 Retry-After 头用以标明这个延迟时间。如果没有给出这个 Retry-After 信息，那么客户端应当以处理500响应的方式处理它。

注意：503状态码的存在并不意味着服务器在过载的时候必须使用它。某些服务器只不过是希望拒绝客户端的连接。

##### 504 Gateway Timeout

作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI标识出的服务器，例如HTTP、FTP、LDAP）或者辅助服务器（例如DNS）收到响应。
注意：某些代理服务器在DNS查询超时时会返回400或者500错误

##### 505 HTTP Version Not Supported

服务器不支持，或者拒绝支持在请求中使用的 HTTP 版本。这暗示着服务器不能或不愿使用与客户端相同的版本。响应中应当包含一个描述了为何版本不被支持以及服务器支持哪些协议的实体。

##### 506 Variant Also Negotiates

由《透明内容协商协议》（RFC 2295）扩展，代表服务器存在内部配置错误：被请求的协商变元资源被配置为在透明内容协商中使用自己，因此在一个协商处理中不是一个合适的重点。

##### 507 Insufficient Storage

服务器无法存储完成请求所必须的内容。这个状况被认为是临时的。WebDAV (RFC 4918)

##### 509 Bandwidth Limit Exceeded

服务器达到带宽限制。这不是一个官方的状态码，但是仍被广泛使用。

##### 510 Not Extended

获取资源所需要的策略并没有被满足。（RFC 2774）

##### 600 Unparseable Response Headers

源站没有返回响应头部，只返回实体内容。

## 3. DNS

### 3.1. 不同系统处理dns响应和首选备选的区别

#### dns响应的理解

- 空结果代表存在此dns的SOA记录，但是此域名没有请求的类型，系统会认为此域名没有这个类型，不会使用备选dns
- no such name代表dns上不存在此域名的记录，系统会认为此域名没有配置，不会使用备选dns
- server failure代表dns服务器出错，会立即使用备选dns

#### 1) linux

- ping会同时请求A记录和AAAA记录
- nslookup会先A记录再AAAA记录，如果A不通不会请求AAAA
- 浏览器请求了A记录，根据当前网卡来
- 本地没有dns时，请求127地址
- 首选不通，超时5s请求备选

#### 2) windows

- 基本同上，但是超时时间为1s

### 3.2. tcp和udp

- tcpdns和udpdns相比在前两个字节多了一个长度（单位字节，不带长度所占的两个字节的大小），也就是dns请求和响应包最大不超过65535
- udp的dns如果超过512字节，就会使用tcp传输

### 3.3. SOA和NS

参考 [SOA记录和NS记录的通俗解释](https://www.cnblogs.com/bighammerdata/p/12776830.html)

DNS服务器里有两个比较重要的记录。一个叫SOA记录（起始授权机构） 一个叫NS（Name Server）记录（域名服务器）关于这两个记录，很多文章都有解释，但是很多人还是很糊涂。我现在通俗的解释一下这两个记录是干什么的。如果理解有错误，欢迎高手来指正。
SOA记录表明了DNS服务器之间的关系。SOA记录表明了谁是这个区域的所有者。比如51CTO.COM这个区域。一个DNS服务器安装后，需要创建一个区域，以后这个区域的查询解析，都是通过DNS服务器来完成的。现在来说一下所有者，我这里所说的所有者，就是谁对这个区域有修改权利。常见的DNS服务器只能创建一个标准区域，然后可以创建很多个辅助区域。标准区域是可以读写修改的。而辅助区域只能通过标准区域复制来完成，不能在辅助区域中进行修改。而创建标准区域的DNS就会有SOA记录，或者准确说SOA记录中的主机地址一定是这个标准区域的服务器IP地址。

如果是两台集成了DNS的DC，实际上由于要求DNS区域可写，所以打破了单纯DNS服务器只能有一个标准区域的限制。所以两台DC都有SOA记录指向自己。

NS记录实际上也是在DNS服务器之间，表明谁对某个区域有解释权，即权威DNS。大家都知道电信和网通都有很多的DNS服务器。这些服务器为我们上公网做域名解析提供了很多方便。但是这些DNS服务器有一个有意思的地方是这些DNS不存放任何区域，看上去更像是一个DNS CLIENT，它们被称为唯缓存DNS服务器。它们会缓存大量的解析地址，这样就会让你解析的时候选择它们会觉得很快。它们在查询的时候就会查询NS记录，通过这个记录就知道谁在负责比如51CTO.COM这个地域的管理工作。还有一种情况来说明NS记录的作用。比如你先在万网申请了一个域名ABC.COM。一般情况是万网的域名服务器替你来解析如WWW.ABC.COM这样的主机记录。如果你想自己架设一个DNS服务器，让这台服务器从今往后替代万网的DNS服务器解析，那么你就需要在你的DNS上设置NS记录，然后将万网域名管理系统中的NS记录改成你的DNSIP。这样以后就是你自己的DNS服务器负责提供解析了。即使万网的DNS服务器出现故障，别人仍然可以找到你。

另外值得一说的是，相对你DNS的CLIENT，你设置的DNS服务器地址就是你的权威DNS。通过NSLOOKUP工具可以看到。而那个非权威应答，恰恰是那个区域真正的NS。

# 七、网络设备

| 名称   | 工作层        | 作用                                                       | 备注                                             |
| ------ | ------------- | ---------------------------------------------------------- | ------------------------------------------------ |
| 中继器 | 物理层        | 将一个网络信号放大发送                                     |                                                  |
| 集线器 | 物理层        | 将一个端口的网络信号放大发送到其他端口                     | 多端口中继器                                     |
| 网桥   | 数据链路层    | 将一个端口的数据发送给另一个端口，需要mac地址对应          |                                                  |
| 交换机 | 数据链路层    | 学习端口mac地址，将一个端口的数据查询mac地址发送给对应端口 | 相当于多个网桥                                   |
| 路由器 | 网络层        | 根据目的ip和路由规则决定转发到哪个端口                     |                                                  |
| 网关   | 网络层/应用层 | 连接两个不同的网络区域                                     | 网关只是概念<br>路由器和三层交换机都可以算作网关 |

- 工作在网络层及以上的设备有自己的mac地址，所以目的mac地址是此设备才能到，设备发出去的包源mac一般为自己

## 1. 家用路由器和计算机网络中的路由器的区分

- 家用路由器可以算作pppoe拨号客户端 + DHCP服务器 + NAT转换 + DNS代理，有路由的功能
- 计算机网络中的路由器仅是路由这个功能，主要是根据路由表进行ip数据包转发

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

## 1. tun网卡和tap网卡区别

- tun网卡工作在第三层网络层，没有mac地址，不能跟物理网卡做桥转发
- tap网卡工作在第二层数据链路层，有mac地址，可以跟物理网卡做桥转发
