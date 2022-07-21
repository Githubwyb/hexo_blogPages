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
