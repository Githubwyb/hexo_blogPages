---
title: python学习笔记
date: 2018-09-16 13:17:51
tags: [study, notes, python]
categories: [notes, study]
top: true
---

# 指定编码格式

## 格式1

源文件第一行或第二行直接定义

```python
    # coding=utf-8
```

# 调用可执行文件

## 获取输出结果

```python
    import os

    f = os.popen("(cmd) (param)")
    data = f.readlines()
    f.close()
```

## 获取返回值

```python
    import os

    r_v = os.system("(cmd) (param)")
    print r_v
```

# 汉字转拼音

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
