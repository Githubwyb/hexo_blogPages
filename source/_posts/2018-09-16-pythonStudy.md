---
title: python学习笔记
date: 2018-09-16 13:17:51
tags: [study, notes, python]
categories: [notes, study]
top: true
---

# 环境

```
    Python 3.6.4
    Anaconda, Inc.
    (default, Jan 16 2018, 10:22:32) [MSC v.1900 64 bit (AMD64)] on win32
```

#  语法相关

## 删除一个变量释放空间

```python
    del var
```

# 指定编码格式

## 格式1

源文件第一行或第二行直接定义

```python
    # coding=utf-8
```

# 特殊操作

## 路径相关操作

### 获取当前路径

```python
    import os

    print(os.getcwd())                  #获取当前工作目录路径
    print(os.path.abspath('.'))         #获取当前工作目录路径
    print(os.path.abspath('test.txt'))  #获取当前目录文件下的工作目录路径
    print(os.path.abspath('..'))        #获取当前工作的父目录 ！注意是父目录路径
    print(os.path.abspath(os.curdir))   #获取当前工作目录路径
```

### 改变当前路径

```python
    import os

    os.chdir(path)
```

## 调用可执行文件

### 获取输出结果

```python
    import os

    f = os.popen("(cmd) (param)")
    data = f.readlines()
    f.close()
```

### 获取返回值

```python
    import os

    r_v = os.system("(cmd) (param)")
    print r_v
```

## 汉字转拼音

需要安装xpinyin模块

```shell
    pip install xpinyin
```

简单用例

```python
    import xpinyin
    pin = xpinyin.Pinyin()
    test1 = pin.get_pinyin("大河向东流")   #默认分割符为-
    print(test1)

    test2 = pin.get_pinyin("大河向东流", "")
    print(test2)
```
