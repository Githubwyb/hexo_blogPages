---
title: keil5常见编译错误的原因
date: 2018-03-30 11:17:54
tags: [keil5,编译]
categories: problems
---

本博客仅为自己记录所遇到的问题的解决方案，希望有所帮助

## 程序环境

- 系统 `Windows10 1709`
- 软件 `keil-mdk arm 5.24`
- 语言 `C语言`

## 问题和原因及解决方案

### warnings

#### 1、函数隐式声明

> warning: #223-D: function "xxxx()" declared implicitly

一般为头文件未包含，但是c文件被编译器编译了，但是在函数被调用的地方没有包含头文件，所以报了一个warning。有些时候会被当做空函数执行，有时没有影响，养成习惯包含头文件。

### errors

#### *持续更新中......*

-------------------

<center> ------我是有底线的------- </center>
