---
title: 软件使用技巧记录
date: 2019-09-12 10:02:35
tags:
categories: [Software Usage]
---

# source insight

## 只添加特定后缀文件

在`Add and remove project files`时，选中要添加的文件夹，在框内输入`*.xx`然后回车，再点击`Add Tree`即可。

## 工程文件目录显示裁剪前面绝对路径

在`Project Setting`里面配置，`File Paths`->`Project Source Directory`下配置源文件路径。配好后就可以看到工程文件目录下文件的路径前面裁剪掉了配置的路径，只留下配置路径的子目录。

# chrome

## 下载离线包

针对在内网环境下需要更新chrome需要下载离线包，官网下载为在线安装包，下载离线包的网址为，也就是在原本的网址上添加`?standalone=1`

[https://www.google.cn/intl/zh-CN/chrome/?standalone=1](https://www.google.cn/intl/zh-CN/chrome/?standalone=1)

# <span id = "tmux">tmux</span>

## 快捷键

- 工具特定命令前缀为`Ctrl + b`
- `<方向键>`: 切换到相应窗口
- `shift + "`: 纵向分屏
- `shift + %`: 横向分屏
- `Ctrl + <方向键>`: 朝相应方向移动边界