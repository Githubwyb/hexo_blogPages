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

1. 先安装

```shell
sudo apt install fcitx5 fcitx5-frontend-gtk3 fcitx5-frontend-gtk2 fcitx5-frontend-gtk3 fcitx5-frontend-gtk2
```

2. 在网上找个配置替换`~/.config/fcitx5/*`


# 解决依赖关系

## 不指名解决依赖关系
```shell
sudo apt --fix-broken install
```

# 设置默认终端

```shell
gsettings set org.gnome.desktop.default-applications.terminal exec /usr/bin/terminator
gsettings set org.gnome.desktop.default-applications.terminal exec-arg "-x"
```

# 快捷键

- `Ctrl + d`: 收藏文件夹

# 添加移除开机启动程序

```shell
update-rc.d xxx enable/disable
```

# 忽略某个包的更新

使用以下命令可以忽略某个包的更新

```shell
sudo apt-mark hold (pack)
```

取消忽略

```shell
sudo apt-mark unhold (pack)
```

# 添加编码

```shell
sudo dpkg-reconfigure locales   # 跟着步骤配一下自己需要的编码
```

# 清除已经删除的软件的配置

```shell
dpkg -l |grep "^rc"|awk '{print $2}' |xargs apt -y purge
```

# 添加/删除ppa源

```shell
# 添加ppa源
sudo add-apt-repository xxx
# 删除ppa源
sudo add-apt-repository -r xxx
```

# 安装中文输入法

## 1. 安装fcitx

执行以下命令相关依赖包都会安装上去

```shell
    sudo apt-get install fcitx-bin
```

安装输入法

```shell
    sudo apt-get install fcitx-table
```

## 2. 配置fcitx

选择设置中的`Region & Language`一栏中的`Manage Installed Language`

<img src = "2018_11_30_01.png">

选择`Keyboard input method system:`为`fcitx`

<img src = "2018_11_30_02.png">

**重启ubuntu**

## 3. 配置输入法

重启后看到右上角有个小键盘，点击进入`Configure`

<img src = "2018_11_30_03.png">

点击`+`添加输入法

<img src = "2018_11_30_04.png">

添加其中的`pinyin`，即可使用中文输入

<img src = "2018_11_30_05.png">

**`Ctrl + Space`切换输入法**

## 4. 安装搜狗输入法

到[官网][1]下载搜狗输入法，安装后重启，在配置界面调整位置即可，第一次启动或许会有一些问题，重启即可

[1]: https://pinyin.sogou.com/linux/?r=pinyin


# 踩坑记

## 1. ubuntu安装deepin-terminal，设置默认

- `x-terminal-emulator`里面没有`deepin-terminal`，无法更新成默认terminal
- 需要执行下面命令进行注册

```shell
# 注册deepin-terminal
sudo update-alternatives --install /usr/bin/x-terminal-emulator x-terminal-emulator /usr/bin/deepin-terminal 50
# 设置默认终端
sudo update-alternatives --config x-terminal-emulator
```
