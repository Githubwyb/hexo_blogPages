---
title: web攻防相关知识
date: 2021-09-23 15:38:01
tags: [网络]
categories: [Program, Web]
---

# 前言

记录一些攻防相关的web知识

# 一、特定语言的攻防

## 1. shtml 可以执行SSI指令的前端页面

参考链接: [HTML语言SSI指令语法](https://blog.csdn.net/dadou2007/article/details/2521365)

### 1.1. 攻

#### SSI指令

SSI（Server Side Include），当客户端访问这些shtml文件时，服务器端会把这些SHTML文件进行读取和解释，把SHTML文件中包含的SSI指令解释出来比如：你可以在SHTML文件中用SSI指令引用其他的html文件（#include ），服务器传送给客户端的文件，是已经解释的SHTML不会有SSI指令。它实现了HTML所没有的功能，就是可以实现了动态的SHTML，可以说是HTML的一种进化吧。像新浪的新闻系统就是这样的，新闻内容是固定的但它上面的广告和菜单等就是用#include引用进来的

目前，主要有以下几种用用途：

1. 显示服务器端环境变量<#echo>
2. 将文本内容直接插入到文档中<#include>
3. 显示WEB文档相关信息<#flastmod #fsize> (如文件制作日期/大小等)
4. 直接执行服务器上的各种程序<#exec>(如CGI或其他可执行程序)
5. 设置SSI信息显示格式<#config>(如文件制作日期/大小显示方式)

高级SSI`<XSSI>`可设置变量使用if条件语句。

##### 几种ssi指令实例和解析

**#include**

- file: 文件名是一个相对路径，该路径相对于使用 #include 指令的文档所在的目录。被包含文件可以在同一级目录或其子目录中，但不能在上一级目录中。如表示当前目录下的的nav_head.htm文档，则为file="nav_head.htm"。
- virtual: 文件名是 Web 站点上的虚拟目录的完整路径。如表示相对于服务器文档根目录下hoyi目录下的nav_head.htm文件；则为file="/hoyi/nav_head.htm"

```html
<!--#include file="info.htm"-->
<!--#include virtual="文件名称"-->
```

**#echo**

```html
本文档名称
<!--#echo var="DOCUMENT_NAME"-->

现在时间
<!--#echo var="DATE_LOCAL"-->

你的IP地址
<!--#echo var="REMOTE_ADDR"-->
```

**#flastmod和#fsize**

- flastmod: 上次更改时间
- fsize: 文件大小

```html
<!--#flastmod file="文件名称"-->
<!--#fsize file="文件名称"-->
```

**#exec**

```html
<!--#exec cmd="cat /etc/passwd"-->          将会显示密码文件
<!--#exec cmd="dir /b"-->                   将会显示当前目录下文件列表
<!--#exec cgi="/cgi-bin/gb.cgi"-->          将会执行CGI程序gb.cgi
```

### 1.2. 防

- 需要在web服务端配置拦截

**nginx配置**

[参考链接](https://blog.csdn.net/qq_33616529/article/details/79061608)

需要的选项主要是以下三个：
- `ssi`: 默认值off，启用ssi时将其设为on
- `ssi_silent_errors`: 默认值off，开启后在处理SSI文件出错时不输出错误提示"[an error occurred while processing the directive]"。
- `ssi_types`: 默认是text/html，所以如果需支持html，则不需要设置这句，如果需要支持shtml则需要设置：`ssi_types` text/shtml
三个参数可以放在http, server或location作用域下。

## 2. sql

### 2.1. 攻

- 一般对于sql注入，使用`' or 1=1; #`
- `'`结束前面的参数引号
- `or 1=1`令整个where失效
- `;`作为结束
- `#`或者`--`将后续的字符串变成注释

# 二、特定软件的攻防

## 1. apache

### 1.1. htaccess

[参考链接](https://baike.baidu.com/item/htaccess/1645473?fr=aladdin)

#### 1.1.1. 攻

#### 1.1.2. 防

关闭htaccess功能，需要修改httpd.conf，设置`AllowOverride none`，将会忽略所有的`.htaccess`配置
