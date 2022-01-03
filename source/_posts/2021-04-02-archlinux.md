---
title: archlinux 安装和配置
date: 2021-04-02 03:43:32
tags: [Linux]
categories: [Program, Shell]
---

# 前言

此文记录我将 archlinux 安装到使用的全过程，供记录备忘

# 一、安装

iso 下载启动我就不多说了，自己搞定

## 1. 分区

新硬盘需要先分区再安装系统，第一次手动命令行分区，可以激动一下

1. 想要 efi 启动，需要在头部创建一个 128M 的 efi 分区
2. 需要交换分区，创建 swap 分区，大小自己定，一般内存两倍
3. 根分区，后面两个想分，根分区给 30G 足以
4. /opt 看着办，如果想重装不影响自己下的软件，可以分
5. /home 看着办，如果想重装不影响自己平常使用的文件数据，可以分

### 1.1. MBR 分区

MBR 分区表最大支持 2T，最多 4 个主分区，属于旧式分区表

```shell
# 开始分区/dev/sda
fdisk /dev/sda
# 删除已有分区
d
# 创建MBR分区表
o
```

**全是主分区的分区**

1. /opt 这个就放到逻辑分区就好了
2. /home 这个就放到逻辑分区就好了

**全是逻辑分区的分区**

只给 efi 分区作为主分区，其他全部逻辑分区
U 盘中这样搞启动时没检测到硬盘，GG 了

## 2. 安装配置

### 2.1. 安装

```shell
pacstrap /mnt linux base base-dev linux-firmware
```

### 2.2. 配置

-   生成 linux 的 img

```shell
mkinitcpio -p linux
```

## 3. 引导

### 3.1. 安装引导

#### (1) GPT + UEFI

1. gpt 下如果要使用 uefi 需要在整个磁盘前面有一个 efi 分区，大小 512M 就可以了，某些情况下需要这么大，如果没有特殊情况 2M 都能用
2. efi 分区需要磁盘类型为`EFI System`

```shell
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

#### (2) MBR + BIOS

```shell
grub-install --target=i386-pc /dev/sda
```

### 3.2. grub 创建引导菜单

-   如果想要探测其他分区的系统，需要安装 os-prober，并且在`/etc/default/grub`添加`GRUB_DISABLE_OS_PROBER=false`
-   需要将需要启动系统的磁盘先挂载好再执行
-   执行下面语句添加引导菜单

```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

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

## 2. 一些命令所在的包

| 命令                    | 包名                 |
| ----------------------- | -------------------- |
| lsusb                   | usbutils             |
| arch-chroot<br>genfstab | arch-install-scripts |

# 三、好用的工具

## 1. kde 桌面下

### 1.1. xpad 桌面便签

#### (1) 快捷键

-   `Ctrl + F8`: 置顶/取消置顶
-   `Ctrl + 鼠标`: 拖动位置

### 1.2. goldendict 翻译工具，支持鼠标取词

-   这个并不是说翻译的多精准，主要是可以各种自定义翻译来源
-   配合命令行翻译工具就很神奇了

#### (1) 配合使用 translate-shell 结果

-   在`Edit -> Dictionary`里面找到 Program，然后新建

| Enabled | Type       | Name  | Command Line                  | Icon |
| ------- | ---------- | ----- | ----------------------------- | ---- |
| $\surd$ | Plain Text | trans | trans en:zh %GDWORD% -no-ansi |      |


### 1.2. fcitx5 输入法

#### (1) 安装

-   需要安装`fcitx5`基础包和`fcitx5-chinese-addons`中文输入包
-   在桌面系统中配置开机启动，程序路径通过`which fcitx5`
-   安装完成后，需要在环境变量配置一下，不然命令行会用不了

```shell
# /etc/environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

-   安装完成后找到`fcitx5-configure`配置自己输入习惯

**主题**

-   github 搜索`fcitx5-themes`下载后，将里面的文件夹拷贝到`~/.local/share/fcitx5/themes`下就可以配置了

#### (2) 注意事项

**1. 无法输入中文方括号**

- 需要修改`/usr/share/fcitx5/punctuation/punc.mb.zh_CN`
- 将下面两行修改一下，重启fcitx5即可

```shell
[ ·
] 「 」
# 改成
[ 【
] 】
```

# 踩坑记和小技巧

## 1. 安装 kde 桌面后，发现特别卡慢

-   原因是 baloo_file_extractor 这个进程占用太多的磁盘 io，导致特别卡
-   解决方法是执行

```shell
balooctl disable
```
