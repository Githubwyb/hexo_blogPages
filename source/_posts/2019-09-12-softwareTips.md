---
title: 软件使用技巧记录
date: 2019-09-12 10:02:35
tags:
categories: [Software Usage]
---

# 一、source insight

## 只添加特定后缀文件

在`Add and remove project files`时，选中要添加的文件夹，在框内输入`*.xx`然后回车，再点击`Add Tree`即可。

## 工程文件目录显示裁剪前面绝对路径

在`Project Setting`里面配置，`File Paths`->`Project Source Directory`下配置源文件路径。配好后就可以看到工程文件目录下文件的路径前面裁剪掉了配置的路径，只留下配置路径的子目录。

# 二、chrome

## 下载离线包

针对在内网环境下需要更新chrome需要下载离线包，官网下载为在线安装包，下载离线包的网址为，也就是在原本的网址上添加`?standalone=1`

[https://www.google.cn/intl/zh-CN/chrome/?standalone=1](https://www.google.cn/intl/zh-CN/chrome/?standalone=1)

# 三、vscode

## 1. 下载加速

- 将下载链接的地址替换为国内镜像地址`vscode.cdn.azure.cn`即可

## 2. 好用的插件

### 2.1. 跨平台快捷键不一致

- 在windows上使用习惯的快捷键在linux不适用
- 安装一个`windows default keybinding`就好了


# 四、firefox

## 1. 修改user-agent

1. 访问`about:config`
2. 创建或修改`general.useragent.override`，值为想要的user-agent值

# 五、网页小技巧

## 1. github查看代码技巧

将`https://github.com/android/ndk-samples`替换成`https://github1s.com/android/ndk-samples`可以打开网页版vscode进行代码查看