---
title: ubuntu学习
date: 2018-07-27 20:58:22
tags: [Linux]
categories: [Program, Shell]
---

# 一、gnome桌面操作

## 1. 开机启动程序

在命令行执行下面语句，然后配置即可

```shell
gnome-session-properties
```

## 2. 安装fcitx5

- ubuntu 20.04没有`fcitx5-configtool`，没有办法配置，只能使用纯文本配置

### 安装步骤

1. 先安装

```shell
sudo apt install fcitx5 fcitx5-frontend-gtk3 fcitx5-frontend-gtk2
```

2. 找个主题装上去
3. 在网上找个配置替换`~/.config/fcitx5/*`
4. 执行`im-config`，选择fcitx5
5. 添加环境变量

```shell
# /etc/environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

5. 添加开机启动程序，参照上面
6. 重启


## 3. 快捷键

- `Ctrl + d`: 收藏文件夹

# 二、kde桌面操作

## 1. 安装

- 需要安装`kubuntu-desktop`包，即为kde的ubuntu包
- 不知道为什么在`ubuntu 20.04`上面，`sddm`起不起来，所以需要安装`gdm3`来启动

## 2. 安装fcitx5

- 步骤同上，就是把`fcitx5-frontend-gtk3 fcitx5-frontend-gtk2`换成`fcitx5-frontend-gtk3 fcitx5-frontend-qt5`

# 三、纯命令行的操作

## 1. apt安装软件

### 1.1. 解决依赖关系

#### 不指名解决依赖关系

```shell
sudo apt --fix-broken install
```

### 1.2. 忽略某个包的更新

使用以下命令可以忽略某个包的更新

```shell
sudo apt-mark hold (pack)
```

取消忽略

```shell
sudo apt-mark unhold (pack)
```

### 1.3. 清除已经删除的软件的配置

```shell
dpkg -l |grep "^rc"|awk '{print $2}' |xargs apt -y purge
```

### 1.4. 添加/删除ppa源

```shell
# 添加ppa源
sudo add-apt-repository xxx
# 删除ppa源
sudo add-apt-repository -r xxx
```

### 1.5. 下载包不安装

- 使用下面命令进行下载，下载的包在`/var/cache/apt/archives/`下

```shell
# 没有安装的包
sudo apt install -d xxx
# 已经安装过的包
sudo apt reinstall -d xxx
```

### 1.6. 包和文件互查

```shell
# 查找包安装的文件列表
=> dpkg -L openssl
/.
/usr
/usr/lib
/usr/lib/ssl
/usr/lib/ssl/misc
...
=> dpkg-query -S /usr/bin/openssl
openssl: /usr/bin/openssl
```

## 2. 设置默认终端

```shell
gsettings set org.gnome.desktop.default-applications.terminal exec /usr/bin/terminator
gsettings set org.gnome.desktop.default-applications.terminal exec-arg "-x"
```

## 3. 添加移除开机启动程序

```shell
update-rc.d xxx enable/disable
```

## 4. 添加编码

```shell
sudo dpkg-reconfigure locales   # 跟着步骤配一下自己需要的编码
```

## 5. apt-file 查看文件所在包的位置

- 使用`apt-file`可以查看

```shell
# 安装
=> sudo apt install apt-file
# 更新数据库
=> sudo apt-file update
# 查找文件
=> apt-file search "dbus/dbus.h"
libdbus-1-dev: /usr/include/dbus-1.0/dbus/dbus.h
```

# 四、一些软件的安装配置

## 1. bind9 搭建dns服务器

参考 [Ubuntu 用bind9搭建DNS服务器](https://blog.csdn.net/weixin_37813152/article/details/122521851)

### 1.1. 安装启动

```shell
sudo apt install bind9
sudo systemctl enable bind9
sudo systemctl start bind9
```

### 1.2. 添加dns正向解析记录

1. 编辑`/etc/bind/named.conf.local`，添加下面的记录

```conf
zone "proxy.com" {
    type master;
    file "/etc/bind/zones/db.proxy.com";
};
```

2. 新建文件`/etc/bind/zones/db.proxy.com`，目录不存在就创建，内容如下

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     proxy.com. root.proxy.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
; proxy.com
@       IN  A   199.200.2.170
; www.proxy.com
@       IN  NS  www
www     IN  A   199.200.2.170
; test.proxy.com
@       IN  NS  test
test    IN  A   199.200.2.170
```

3. 重启bind9服务就可以解析`proxy.com`、`www.proxy.com`、`test.proxy.com`

### 1.3. 添加反向记录 PTR

- 反向记录一般是nslookup用于展示dns服务器地址的域名使用
- 如下面的结果会发起一个 `101.17.240.10.in-addr.arpa` 的 `PTR` 请求

```shell
=> nslookup www.testweb.com
Server:  dns.proxy.com
Address:  10.240.17.101

Name:    www.testweb.com
Address:  199.200.2.170
```

1. 编辑`/etc/bind/named.conf.local`，添加下面的记录，添加`10.240.17.0/24`的反查记录

```conf
zone "17.240.10.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/17.240.10.zone";
};
```

2. 新增文件`/etc/bind/zones/17.240.10.zone`，`NS`是必须的，添加101的解析

```conf
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     proxy.com. root.proxy.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
; dns.proxy.com
@       IN  NS      dns.proxy.com.
101     IN  PTR     dns.proxy.com.
```

## 2. lightdm 使用vnc远程连接

- 需要安装`tigervnc-standalone-server`和`tigervnc-common`
- 其他参考 [archlinux配置lightdm远程桌面](/blogs/2021-04-02-archlinux/#1-4-远程桌面)

# 小技巧和踩坑记

## 1. ubuntu安装deepin-terminal，设置默认

- `x-terminal-emulator`里面没有`deepin-terminal`，无法更新成默认terminal
- 需要执行下面命令进行注册

```shell
# 注册deepin-terminal
sudo update-alternatives --install /usr/bin/x-terminal-emulator x-terminal-emulator /usr/bin/deepin-terminal 50
# 设置默认终端
sudo update-alternatives --config x-terminal-emulator
```

## 2. 信任CA证书

```shell
sudo cp xxx.crt /usr/local/share/ca-certificates
sudo update-ca-certificates
```

- 删除和新增同理，都只需要文件删除和添加执行命令即可
