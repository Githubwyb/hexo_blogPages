---
title: ubuntu学习
date: 2018-07-27 20:58:22
tags: [study, notes, linux]
categories: [notes, study]
---

# 解决依赖关系

## 不指名解决依赖关系
```shell
    sudo apt --fix-broken install
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