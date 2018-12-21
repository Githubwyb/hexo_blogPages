---
title: stm32cubemx使用freertos的介绍
date: 2018-03-30 11:17:54
tags: [软件, 使用教程, stm32cubemx, freertos]
categories: [technology]
top: true
---

本博客仅为自己在使用时的总结，希望有所帮助

## 程序环境

- 系统 `Windows10 1709`
- 软件 `stm32cubemx 4.25.0`
- 编译软件 `keil-mdk arm 5.24`
- 语言 `C语言`

## 前言

本文是在上一篇文章（[stm32cubemx配置介绍][1]）的基础上，利用freeRTOS的一些介绍，希望对想要了解或者使用freeRTOS的码友们提供帮助。本文只是自己在使用时的总结，仅供参考。

[1]: http://localhost:4000/2018/03/30/2018_03_16_stm32cubemx_config/