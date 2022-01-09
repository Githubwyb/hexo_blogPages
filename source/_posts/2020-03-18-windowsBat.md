---
title: windows命令行记录
date: 2020-03-18 14:51:41
tags: [windows]
categories: [Program, Shell]
---

# 一、bat脚本

## 1. 一些基本语法（cmd使用）

```bat
REM 这是注释
:: 这是注释

:: 打印环境变量
echo %xxx%

:::::::::: 目录操作 ::::::::::
:: 列举目录内容，类似ls
dir xxx/xxx

:::::::::: 文件操作 ::::::::::
:: 新建文件
type NUL > xxx.txt
:: 删除文件
del xxx.txt
```

## 2. 创建链接 mklink

- `/j`: 创建目录链接
- `/d`: 创建目录符号链接。默认为文件符号链接
- `/h`: 创建硬链接而非符号链接

```bat
mklink /j [link_name] [src_dir]
```

**硬链接、符号链接、目录链接、快捷方式区别**

|            | 硬链接                     | 符号链接  | 目录链接           | 快捷方式    |
| ---------- | -------------------------- | --------- | ------------------ | ----------- |
| 类型       | 链接                       | 链接      | 链接               | xxx.lnk文件 |
| 作用对象   | 仅文件                     | 目录/文件 | 目录/文件          | 目录/文件   |
| 大小       | 和源文件一致（但不占空间） | 0         | 0                  | 几百字节    |
| 源文件删除 | 文件内容存在               | 失效      | 失效               | 失效        |
| 源文件替换 | 文件内容还是原始文件       | 新文件    | 新文件             | 新文件      |
| 局限       | 仅同一个分区               | 失效      | 失效               | 失效        |
| 路径       | -                          | 相对路径  | 自动转化为绝对路径 | -           |

## 3. 合并文件 COPY

- 合并两个二进制文件

```bat
COPY /B [bin_file1] [bin_file2]
```

# 二、powershell

## 1. 一些基本用法

```bat
:: 打印环境变量
PS> $env:xxx
:: 设置环境变量
PS> $env:TestVar1="This is my environment variable"
:: 枚举环境变量，注意后面有个冒号
PS> ls env:
:: 删除环境变量
PS> del env:windir

:::::::::: 文件操作 ::::::::::
:: 新建文件
PS> New-item xxx.txt -type file
```

# 小技巧

## 1. powershell使用utf-8编码

- 启动powershell时输入

```bat
chcp 65001
```

## 2. windows 10启用sshd服务

- 需要在功能中安装`OpenSSH Server`
- 在命令行中执行，登陆帐号密码和微软一致

```bat
:: 开启
net start sshd
:: 关闭
net stop sshd
```
