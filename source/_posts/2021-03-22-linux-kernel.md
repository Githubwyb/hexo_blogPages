---
title: linux kernel源码学习记录
date: 2021-03-22 16:55:36
tags: [Linux]
categories: [Program, C/C++]
---

# 一、前言

本文为研究linux kernel源码所记录的一些笔记

源码下载路径
```
https://mirrors.edge.kernel.org/pub/linux/kernel/
```

# 二、入口

- linux kernel启动函数（main函数）并不是常规的main，是`init/main.c`里面的`start_kernel()`函数

