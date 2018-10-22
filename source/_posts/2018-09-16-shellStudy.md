---
title: shell学习笔记
date: 2018-09-16 13:23:55
tags: [study, notes, linux]
categories: [notes, study]
---

# 目录特殊符号

- 上一级 `..`
- 当前目录 `.`
- 当前用户目录 `~`
- 根目录 `/`

# 跳转目录cd命令

## 特殊跳转

- 跳转上一级 `..`
- 跳转当前目录 `.`
- 跳转当前用户目录 `~`
- 跳转根目录 `/`
- 跳转上一个目录 `cd -`
- 跳转前n目录 `cd -n`
- 跳转后n目录 `cd +n`

# 命令详解

## watch

watch命令以周期性的方式执行给定的指令，指令输出以全屏方式显示。watch是一个非常实用的命令，基本所有的Linux发行版都带有这个小工具，如同名字一样，watch可以帮你监测一个命令的运行结果，省得你一遍遍的手动运行。

### 语法

    watch(选项)(参数)

### 选项

- -n：指定指令执行的间隔时间（秒）；
- -d：高亮显示指令输出信息不同之处；
- -t：不显示标题。

### 参数
指令：需要周期性执行的指令。

### 实例

```shell
    watch uptime
    watch -t uptime
    watch -d -n 1 netstat -ntlp
    watch -d 'ls -l | fgrep goface'     //监测goface的文件
    watch -t -differences=cumulative uptime
    watch -n 60 from            //监控mail
    watch -n 1 "df -i;df"       //监测磁盘inode和block数目变化情况
```

# 常用命令

## 实时查看文件内容（可用于查看log文件）

```shell
    tail -f (fileName)
```