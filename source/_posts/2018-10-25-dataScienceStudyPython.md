---
title: 数据处理笔记（python）
date: 2018-10-25 16:40:34
tags: [notes, python, data]
categories: [notes, work]
---

# 环境

```
    Python 3.6.4
    Anaconda, Inc.
    (default, Jan 16 2018, 10:22:32) [MSC v.1900 64 bit (AMD64)] on win32
```

# pandas

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

## Series

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

## ndarray

### reshape

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

# matplotlib

## pyplot

### seaborn

#### 统计向量中值的出现次数画图

```python
    import matplotlib.pyplot as plt
    import seaborn as sns

    sns.countplot(Y_train)
    plt.show()
```
