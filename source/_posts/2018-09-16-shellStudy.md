---
title: shell学习笔记
date: 2018-09-16 13:23:55
tags: [Linux]
categories: [Program, Shell]
---

# 目录特殊符号

- 上一级 `..`
- 当前目录 `.`
- 当前用户目录 `~`
- 根目录 `/`

# 常用命令详解

## > 输出

```shell
    ls ./ >a.text           # 正确输出到a.txt(覆盖)
    ls ./ 1>a.text          # 正确输出到a.txt(覆盖)
    ls ./ 2>a.txt           # 错误输出到a.txt(覆盖)
    ls ./ 2>a.txt           # 错误输出到a.txt(覆盖)
    ls ./ >a.txt 2>&1       # 标准输出和标准错误输出到a.txt(覆盖)
    ls ./ &>a.txt           # 标准输出和标准错误输出到a.txt(覆盖)
    # 追加使用>>
    ls ./ >>a.text          # 正确输出到a.txt(追加)
```

## $ 意义

```shell
    $$      # Shell本身的PID（ProcessID）
    $!      # Shell最后运行的后台Process的PID
    $UID    # 当前用户的id
    $PPID   # 父进程id
    $?      # 最后运行的命令的结束代码（返回值）
    $-      # 使用Set命令设定的Flag一览
    $*      # 所有参数列表。如*所有参数列表。如"*“用「”」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
    $@      # 所有参数列表。如@所有参数列表。如"@“用「”」括起来的情况、以"$1 $2 … $n" 的形式输出所有参数。
    $#      # 添加到Shell的参数个数
    $0      # Shell本身的文件名
    $1～n   # 添加到Shell的各参数值。$1是第1参数、$2是第2参数…
```

## cd 跳转目录命令

### 特殊跳转

- 跳转上一级 `..`
- 跳转当前目录 `.`
- 跳转当前用户目录 `~`
- 跳转根目录 `/`
- 跳转上一个目录 `cd -`
- 跳转前n目录 `cd -n`
- 跳转后n目录 `cd +n`

## if 条件判断

### 文件表达式

```shell
    if [ -e file ]      # 如果文件或者目录存在，不管有没有权限
    if [ -f file ]      # 如果文件是普通文件，不是目录或者设备文件
    if [ -d ...  ]      # 如果目录存在
    if [ -s file ]      # 如果文件存在且非空
    if [ -r file ]      # 如果文件存在且可读
    if [ -w file ]      # 如果文件存在且可写
    if [ -x file ]      # 如果文件存在且可执行
```

### 整数变量表达式

```shell
    if [ int1 -eq int2 ]        # 如果int1等于int2
    if [ int1 -ne int2 ]        # 如果不等于
    if [ int1 -ge int2 ]        # 如果>=
    if [ int1 -gt int2 ]        # 如果>
    if [ int1 -le int2 ]        # 如果<=
    if [ int1 -lt int2 ]        # 如果<
```

### 字符串变量表达式

```shell
    if  [ $a = $b ]                     # 如果string1等于string2
                                        # 字符串允许使用赋值号做等号
    if  [ $string1 != $string2 ]        # 如果string1不等于string2
    if  [ -n $string ]                  # 如果string 非空(非0），返回0(true)
    if  [ -z $string ]                  # 如果string 为空
    if  [ $sting ]                      # 如果string 非空，返回0 (和-n类似)
```

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

## tail 实时查看文件内容（可用于查看log文件）

```shell
    tail -f (fileName)
```

## grep 查找文件内容

### 查找目录下所有文件匹配对应字符串

```shell
    grep -r -n "test" ./
```
- `-r`遍历子目录
- `-n`遍历行数
- `-i`大小写无关

# 快捷键

## 搜索历史命令输入

- `Ctrl + r`
- 支持模糊搜索

# 工具

## 网络嗅探 nmap

[nmap使用](https://baike.baidu.com/item/nmap/1400075?fr=aladdin)

## ssh生成公钥和私钥

```shell
    ssh-keygen -t rsa -C "xxx@xxx.com"
```

## 压缩和解压缩命令

### tar命令

#### 参数解析

```shell
    -c: 建立压缩档案
    -x：解压
    -t：查看内容
    -r：向压缩归档文件末尾追加文件
    -u：更新原压缩包中的文件
```

这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。

```shell
    -z：有gzip属性的
    -j：有bz2属性的
    -Z：有compress属性的
    -v：显示所有过程
    -O：将文件解开到标准输出
```

这几个根据需要在压缩或解压档案时可选的。

```
    -f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。
```

参数-f是必须的

```shell
    -C: 解压到什么目录
```

#### 示例

```shell
    tar -xzf xxx.tar.gz             # 解压tar.gz文件
    tar -xzvf xxx.tar.gz -C tmp     # 解压tar.gz文件显示过程，解压到tmp目录
```

### zip格式

#### 压缩

压缩目录和目录下所有文件

```shell
    zip -r (filename.zip) (path)
```

#### 解压

```shell
    unzip (filename.zip) -d (path)
```

- 不加`-d`就解压到当前目录

# 系统命令

## 修改卷标名称

分区为`ext2/ext3`类型使用

```shell
    e2label /dev/(partition) "(name)"
```

# 脚本语法

## 函数调用

```shell
    abc() {
        # ...
    }

    abc
```

函数调用不加括号
