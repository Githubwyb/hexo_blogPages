---
title: 架构师相关知识
date: 2022-12-28 14:48:36
tags: [后台开发]
categories: [Program, Web]
---

见 [系统架构师](/docs/examination/system-architect/large-concurrency/common/)

# 二、数据库

## 1. 数据库读写分离

- 读操作和写操作对应不同场景，写操作更耗费性能
- 将写操作写入主服务器，通过复制的方式到读数据库，加快读数据的响应速度

## 2. mysql

### 2.1. 备份方案

参考 [9 Best MySQL Backup Tools](https://www.pcwdld.com/best-mysql-backup-tools#wbounce-modal)

- 物理备份：二进制文件拷贝，需要重启mysql但是耗时很短，不锁表，占用空间较大，percona开源
  - 首次物理备份后可以通过增量拉取变更数据（基于LSN计算偏移量）
  - 备份需要读取mysql的binlog，只能在对应机器执行，不能ip+端口远程执行
- 逻辑备份：基于sql脚本，占用空间小，但是会锁表，不需要重启mysql，官方自带

# 三、CDN和反向代理

## 1. 使用场景

- 一般对于静态资源放到cdn服务器上，不会到应用处理服务器上

# 四、分布式消息队列

# 五、本地集群和分布式集群

## 1. 本地集群

- 本地集群是指由多台计算机组成的集群，这些计算机通常位于同一地理位置，通过局域网或高速网络连接在一起。本地集群通常用于处理大量数据或执行计算密集型任务，例如科学计算、数据分析和机器学习等
- 在互联网场景下，本地集群可以认为是一台高性能设备

### 1.1. 本地集群的模式

- 主从集群是由一个主节点和多个从节点组成的集群，主节点负责处理所有的写操作，从节点则负责复制主节点的数据并处理读操作。主节点和从节点之间通过数据同步机制保持数据的一致性。当主节点发生故障时，从节点中的一个会被选举为新的主节点，继续处理写操作。
- 主备集群是由一个主节点和一个备节点组成的集群，主节点负责处理所有的写操作，备节点则负责复制主节点的数据。当主节点发生故障时，备节点会接管主节点的工作，成为新的主节点。与主从集群不同的是，主备集群只有一个备节点，因此在备节点接管主节点工作的过程中，可能会出现一段时间的数据不一致。

### 1.2. 主从集群

所有节点参与分发，主节点关注写操作，其他节点均只有读操作，写操作从主节点复制过来

#### 1) 集群分发模式

##### NAT 网络层处理

- 主节点将用户请求的数据包的目的地址修改为从节点，转发给从节点
- 从节点处理完后，返还给主节点，主节点再将源ip改为集群ip发出去
- 三层代理

##### DR 数据链路层处理

- 主节点将用户请求的数据包的目的mac地址修改为从节点，发给从节点
- 从节点处理后，将数据包以集群ip为源地址直接发出去
- 二层代理

### 1.3. 主备集群

## 2. 分布式集群

- 分布式集群是指由多台计算机组成的集群，这些计算机通常位于不同的地理位置，通过互联网或其他网络连接在一起。分布式集群通常用于处理大规模的数据和任务，例如大规模网站、分布式数据库和分布式文件系统等
- 互联网场景下，分布式集群是多个不同地理位置的设备共同提供服务

# 六、性能优化

## 1. 限流

### 1.1. 并发连接数

- 同一时间的连接总数上限
- 可能会因为某些请求的处理时间较长而导致限流

#### 1) 限流报错排查

- 一般看处理日志，是否某些请求的处理卡住超时

### 1.2. 新建连接数

- 每秒新建总数上限

#### 1) 限流报错排查

- 这种一般是下发了事件后，同一时间客户端的请求数量过大，一般做客户端请求离散化或服务端下发事件离散化处理即可
