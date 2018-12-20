---
title: 数据处理笔记（python）
date: 2018-10-25 16:40:34
tags: [notes, python, data]
categories: [notes, work]
top: true
---

# 环境

```
    Python 3.6.4
    Anaconda, Inc.
    (default, Jan 16 2018, 10:22:32) [MSC v.1900 64 bit (AMD64)] on win32
```

# 默认函数

## range 列表

创建一个整数列表

```python
    >>>range(10)        # 从 0 开始到 10
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    >>> range(1, 11)     # 从 1 开始到 11
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    >>> range(0, 30, 5)  # 步长为 5
    [0, 5, 10, 15, 20, 25]
    >>> range(0, 10, 3)  # 步长为 3
    [0, 3, 6, 9]
    >>> range(0, -10, -1) # 负数
    [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
    >>> range(0)
    []
    >>> range(1, 0)
    []
```

# pandas

## 显示数据的各种统计数据

包括中位数、平均数、标准差、最值等

```python
    dataframe.describe()
    series.describe()
```

## csv格式文件读取

可以使用`read_csv()`函数将`csv`读入成`dataframe`的格式

```python
    import pandas as pd

    test_df = pd.read_csv('test.csv')
```

## dataframe

### 显示行列数

```python
    print(X_train_df.shape)
```

### 转成ndarray多维矩阵形式

```python
    test_ndarray = X_train_df.values
```

### 删除一列

不会改变原变量，删除结果由返回值给出

```python
    X_train = train.drop(labels = ["label"], axis = 1)
```

### 删除一行

不会改变原变量，删除结果由返回值给出

```python
    X_train_df = train.drop(index = 0)
```

### 提取一列

```python
    Y_train_series = train['labels']
```

### 判断空值

```python
    isNull_df = X_train_df.isnull() #得到一个同纬度的true和false组成的dataframe
    isNull_df.any()                 #显示各列的情况是否含有true，相当于各列取或
```

## Series 序列

### 统计各值出现数量

```python
    print(Y_train_series.value_counts())
```

# numpy

## numpy数据存取

numpy有一个可以直接将numpy数组矩阵按照原来的格式储存和读取的函数

```python
    import numpy as np

    # 储存data
    np.save("file directory", data)

    # 读取data
    data = np.load("file directory")
```

这个方法可以直接将shape也存下来，不需要考虑格式、类型等，缺点是文件会有点大。

## ndarray 矩阵

### reshape 重置矩阵形状

从最外层开始重置矩阵形状，默认按行读取，-1代表未知数量，由numpy自动计算

```python
    test_ndarray = test_ndarray.reshape(-1, 28, 28, 1)
    print(test_ndarray.shape)   #out: (42000, 28, 28, 1)
```

## 随机数

- 随机种子

```python
    import numpy as np

    np.random.seed(2)
```

## linespace 列表

```python
    import numpy as np

    y = np.linspace(m, n, z) # 在[m, n]等距离取z个点
    x = np.linspace(m, n) # 同上，z默认取50
```

# matplotlib

## pyplot 画图

### 新开一个页面 figure

```python
    import matplotlib.pyplot as plt

    plt.figure()
    ...
    plt.figure()
    ...
    plt.show()
```

### 一页多图 subplot

```python
    import matplotlib.pyplot as plt
    
    plt.figure()
    plt.subplot(3, 2, 1) # 3行2列，从左向右，从上向下，第一个
    ...
    plt.subplot(3, 2, 3) # 3行2列，从左向右，从上向下，第三个
    ...
    plt.subplot(3, 2, 5) # 3行2列，从左向右，从上向下，第五个
    ...
    plt.subplot(1, 2, 2) # 1行2列，从左向右，从上向下，第二个
    ...
    plt.show()
```

效果图

<img src = "2018_11_29_01.png">

### 页面属性更改

```python
    import matplotlib.pyplot as plt

    plt.figure("abc") # 整个图表名字
    ...
    plt.xlabel("x") # 横坐标名称
    plt.ylabel("y") # 纵坐标名称
    plt.title("y = f(x)") # 当前图的名字
    plt.show()
```

### stem 散点图

```python
    #coding=utf-8
    import matplotlib.pyplot as plt
    import numpy as np

    y = np.linspace(0, 100, 32)
    x = list(range(0, 32))

    plt.figure()
    plt.stem(x, y)
    plt.show()
```

效果图

<img src = "2018_11_29_02.png">

# seaborn

## 画图统计向量中值的出现次数 countplot

```python
    import matplotlib.pyplot as plt
    import seaborn as sns
    ...
    sns.countplot(Y_train)
    plt.show()
```

效果图

<img src = "2018_11_29_03.png">

# scipy

## fftpack

### fft 快速傅里叶变换

```python
    from scipy.fftpack import fft

    x = np.linspace(0, 100, 32)
    y = fft(x) # 得到x的32点fft
```
