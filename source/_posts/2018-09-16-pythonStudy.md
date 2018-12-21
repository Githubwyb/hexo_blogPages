---
title: python学习笔记
date: 2018-09-16 13:17:51
tags: [python]
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

## 变量

### None

- None是一个特殊的常量。
- None和False不同。
- None不是0。
- None不是空字符串。
- None和任何其他的数据类型比较永远返回False。
- None有自己的数据类型NoneType。
- 你可以将None赋值给任何变量，但是你不能创建其他NoneType对象。

python中矩阵索引使用None表示此维度不切片，同样意味着此维度大小未知

## del 删除一个变量释放空间

```python
    del var
```

## try 异常处理

### try-except-else

```python
    try:
        <语句>        #可能出错的代码
    except <名字>：
        <语句>        #如果在try部份引发了'name'异常
    except <名字>，<数据>:
        <语句>        #如果引发了'name'异常，获得附加的数据
    else:
        <语句>        #如果没有异常发生
```

### try-finally

```python
    try:
        <语句>
    finally:
        <语句>    #退出try时总会执行
```

### raise触发异常

#### 形式

```python
    raise [Exception [, args [, traceback]]]
```

#### 实例

```python
    #!/usr/bin/python
    # -*- coding: UTF-8 -*-

    # 定义函数
    def mye(level):
        if level < 1:
            raise Exception, "Invalid level!"
            # 触发异常后，后面的代码就不会再执行
    try:
        mye(0)            # 触发异常
    except Exception, err:
        print(1, err)
    else:
        print(2)
```

输出

```shell
    1 Invalid level!
```

#### 用户自定义异常

通过创建一个新的异常类，程序可以命名它们自己的异常。异常应该是典型的继承自Exception类，通过直接或间接的方式。

```python
    class Networkerror(RuntimeError):
        def __init__(self, arg):
            self.args = arg

    try:
        raise Networkerror("Bad hostname")
    except Networkerror,e:
        print(e.args)
```

## with 上下文

### with是什么

with处理相当于`try-finally`

```python
    file = open("/tmp/foo.txt")
    try:
        data = file.read()
    finally:
        file.close()

    # 等价于

    with open("/tmp/foo.txt") as file:
        data = file.read()
```

### with怎么工作

基本思想是with所求值的对象必须有一个`__enter__()`方法，一个`__exit__()`方法。

紧跟with后面的语句被求值后，返回对象的`__enter__()`方法被调用，这个方法的返回值将被赋值给as后面的变量。当with后面的代码块全部被执行完之后，将调用前面返回对象的`__exit__()`方法。

实例

```python
    #!/usr/bin/env python
    # with_example01.py
    
    class Sample:
        def __enter__(self):
            print "In __enter__()"
            return "Foo"
    
        def __exit__(self, type, value, trace):
            print "In __exit__()"
    
    def get_sample():
        return Sample()
    
    with get_sample() as sample:
        print "sample:", sample
```

输出

```shell
    In __enter__()
    sample: Foo
    In __exit__()
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
