---
title: archlinux 安装和配置
date: 2021-04-02 03:43:32
tags: [Linux]
categories: [Program, Shell]
---

# 前言

此文记录我将archlinux安装到使用的全过程，供记录备忘

# 一、安装

iso下载启动我就不多说了，自己搞定

## 1. 分区

新硬盘需要先分区再安装系统，第一次手动命令行分区，可以激动一下

1. 想要efi启动，需要在头部创建一个128M的efi分区
2. 需要交换分区，创建swap分区，大小自己定，一般内存两倍
3. 根分区，后面两个想分，根分区给30G足以
4. /opt看着办，如果想重装不影响自己下的软件，可以分
5. /home看着办，如果想重装不影响自己平常使用的文件数据，可以分

### 1.1. MBR分区

MBR分区表最大支持2T，最多4个主分区，属于旧式分区表

```shell
# 开始分区/dev/sda
fdisk /dev/sda
# 删除已有分区
d
# 创建MBR分区表
o
```

**全是主分区的分区**

1. /opt这个就放到逻辑分区就好了
2. /home这个就放到逻辑分区就好了

**全是逻辑分区的分区**

只给efi分区作为主分区，其他全部逻辑分区
U盘中这样搞启动时没检测到硬盘，GG了

# 二、日常操作

## 1. 常用命令

### 1.1. pacman 安装软件

#### 1.1.1. 一些基本用法

```shell
########## S --sync 同步（安装搜索） ##########
# y 同步最新仓库
# u update
sudo pacman -Syu xxx

# s 搜索软件
sudo pacman -Sys xxx

# c 从缓存仓库清理旧包，cc清理所有
sudo pacman -Scc

########## R --remove 移除 ##########
# u 移除用不到的包
# s 递归移除用不到的依赖
# n 删除配置文件
# c 移除包和依赖它的包
sudo pacman -Rusnc xxx

########## Q --query 查询（本地） ##########
# d 列出作为依赖项安装的包
# t 列出不被其他包需要的包
# q 只展示包名，不展示版本号
sudo pacman -Qdtq
```

### 1.2. journalctl

#### 1.2.1. 一些基本用法

```shell
# 查看日志的磁盘使用量
sudo journalctl --disk-usage

# 清理5天之前的日志
sudo journalctl --vacuum-time=5d
```

# 踩坑记
