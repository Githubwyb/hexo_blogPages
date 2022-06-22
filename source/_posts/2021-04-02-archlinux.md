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

1. 想要 efi 启动，需要在头部创建一个 1G 的 efi 分区，格式化为fat32，标识要求是`EFI System`，挂载到`/boot/efi`
2. 需要交换分区，创建 swap 分区，大小自己定，一般内存两倍
3. 根分区，后面两个想分，根分区给 30G 足以
4. /opt 看着办，如果想重装不影响自己下的软件，可以分
5. /home 看着办，如果想重装不影响自己平常使用的文件数据，可以分

### 1.1. GPT 分区

GPT 分区表最多支持200个主分区

### 1.2. MBR 分区

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
pacstrap /mnt linux base linux-firmware networkmanager tmux vim grub net-tools efibootmgr archlinux-keyring
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

## 4. 桌面环境

### 4.1. kde

- 安装下面的包

```shell
sudo pacman -S plasma kde-applications xorg-server xorg-drivers sddm
```

#### 踩坑记

##### (1) 屏幕显示抖动，设置页面点击没有变化等异常问题

- 尝试将`xorg.conf`的`driver`改成

```conf
Driver      "modesetting"
```

# 二、日常操作

## 1. 常用命令

### 1.1. pacman 安装软件

#### 1) 一些基本用法

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
# 查看哪些包可以更新
sudo pacman -Qu
# 查看二进制是哪个包安装进来的
sudo pacman -Qo /usr/sbin/tcpdump
# 查看包安装了哪些文件
sudo pacman -Ql tcpdump
```

#### **<font color="red">踩坑记</font>**

1. 遇到安装提示`PGP Signature`错误的，安装一下`archlinux-keyring`，原因是签名过期了

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
| telnet                  | inetutils            |
| ntpdate                 | ntp                  |
| nslookup                | bind                 |

## 3. aur软件包

- aurhelper很多，我一般使用yay进行aur包安装
- yay可以使用所有pacman能用的命令选项
- 下面列举一些pacman没有的命令

```shell
# 打印当前yay的配置选项
yay -Pg
```

## 4. 信任CA证书

- 执行完需要重启浏览器

```shell
# 查看已经授信的ca机构
trust list
# 添加授信的ca机构
trust anchor /path/to/cacert.pem
# 移除授信的ca机构，值由list给出
trust anchor --remove 'pkcs11:id=%2E%57%67%B4%D5%D0%13%93%52%B5%4F%7C%87%1C%FC%45%43%FF%E7%02;type=cert'
```

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

-   需要安装`fcitx5-im`基础包和`fcitx5-chinese-addons`中文输入包
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

## 2. 命令行工具

### 2.1. mysql

#### (1) 安装

- archlinux中安装mysql需要安装mariadb包

```shell
sudo pacman -Sy mariadb
```

#### (2) 配置

1. 要先执行下面命令配置mysql

```shell
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

2. 新建用户，首次需要使用管理员权限登陆root帐号，密码为空

```shell
sudo mysql -u root -p
```

```sql
MariaDB [(none)]> CREATE USER 'xxx'@'localhost' IDENTIFIED BY 'xxxx';
Query OK, 0 rows affected (0.034 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON mydb.* TO 'xxx'@'localhost';
Query OK, 0 rows affected (0.023 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> exit;
```

3. 使用新用户登陆，开始你的表演

```shell
mysql -u xxx -p
```

## 3. vmware

### 3.1. 网络和usb

- vmware启动前需要启动两个服务，不然无法联网和usb映射

```shell
sudo systemctl start vmware-networks.service
sudo systemctl start vmware-usbarbitrator.service
```

# 踩坑记和小技巧

## 1. 安装 kde 桌面后，发现特别卡慢

-   原因是 baloo_file_extractor 这个进程占用太多的磁盘 io，导致特别卡
-   解决方法是执行

```shell
balooctl disable
```

## 2. 安装vscode之后，登陆github提示`writing login information to keychain failed`

- 由于arch安装的vscode使用的是ubuntu的包，所以keyring需要使用`gnome-keyring`

```shell
sudo pacman -S gnome-keyring
```

## 3. `/etc/sysctl.conf`重启后没有自动加载

- arch将`sysctl.conf`分到了`/etc/sysctl.d/*.conf`下，需要移动进去
- 具体说明见`man sysctl.d`
