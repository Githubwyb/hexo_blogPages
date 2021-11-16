---
title: 数据处理笔记（python）
date: 2018-10-25 16:40:34
tags:
categories: [Program, Python]
top: 16
---

# 环境

```
Python 3.8.0
```

# 1. 内建函数

## range 列表

创建一个整数列表

```python
>>> range(10)        # 从 0 开始到 10
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

## list操作

判断某个值是否在list中

```python
test_list = list(xxxx)
print('test' in test_list)  # 返回true和false
```

## filter过滤器

删除list中空值和nan

```python
# python3中filter返回迭代器，需要使用list()转化回list
test_list = list(filter(lambda s: s and s.strip(), test_list))
```

## map向量对应计算

- 两个同样长度的向量，每个元素按照function计算

```python
a = [1, 2, 3, 4]
b = [4, 5, 6, 7]
result = map(lambda x, y: x + y, a, b)
# 输出是map对象，好像不会直接出结果，访问时计算
print(list(result))  # [5, 7, 9, 11]
```

## zip向量对应打包

- 两个同样长度的向量，每个元素打包到一起

```python
a = [1, 2, 3, 4]
b = [4, 5, 6, 7]
result = zip(lambda x, y: x + y, a, b)
# 输出是zip对象，转list打印
print(list(result))  # [[1, 4], [2, 5], [3, 6], [4, 7]]
```

## reduce向量挨个计算

- 两个同样长度的向量，每个元素进行计算，结果和下个元素进行计算

```python
a = [1, 2, 3, 4]
result = reduce(lambda x, y: x + y, a)
# 输出是结果
print(list(result))  # 1 + 2 + 3 + 4 = 10
```

## dict操作

### dict合并

```python
dict1 = {"a": 1, "b": 2}
dict2 = {"a": 2, "c": 3}
dict3 = {**dict1, **dict2}  # 用dict2更新dict1
```

# 2. 第三方module

## 2.1. pandas

### csv格式文件读写

- 可以使用`read_csv()`函数将`csv`读入成`dataframe`的格式
- 用`to_csv()` 将dataframe写入到`csv`文件中

```python
import pandas as pd

test_df = pd.read_csv('test.csv')
# 以第一列为索引读取
test_df = pd.read_csv('test.csv', index_col=0)
# 写入csv文件，不写入index
test_df.to_csv('test.csv', index=False)
```

### dataframe相关操作

dataframe本质上多个相同索引的series组成的数据结构

#### 信息

```python
import pandas as pd

df = pd.DataFrame(xxxxx)
df.describe()       # 显示数据的各种统计数据包（中位数、平均数、标准差、最值等）
df.shape()          # 行列数
isnull_df = df.isnull()     # 得到一个同纬度的true和false组成的dataframe
isNull_df.any()             #显示各列的情况是否含有true，相当于各列取或
```

#### 操作

- 一般df会改变数据的方法都有一个参数inplace，为True的时候改变原数据，为False的时候不改变原数据

```python
import pandas as pd

df = pd.DataFrame(xxxxx)

#################### 增 ########################
# 插入一列，最右侧
df['newColumns1'] = True
df['newColumns2'] = [xxx]    # 要求数组个数和df的行数保持一致，否则会失败
# 在指定位置插入一列
df.insert(loc=0, column='newColumns3', value=True)
# 插入多行
df = pd.concat([df, addLines])

#################### 删 #######################
# 删除label列，改变原数据
df.drop(labels = ["label"], axis = 1, inplace=True)
# 同理删除一行
df.drop(index = 0, inplace=True)
# 删除所有含有nan的行
df.dropna(inplace=True)

################### 查 #######################
# 提取一列转成Series形式，以下两种都可以
test_series = df.test
test_series = df['test']

# 提取一行或几行，有两个定位的方法，一个是loc，一个是iloc。
## loc只能定位索引，传入的值只能是索引，如果索引不是数字，不能传入数字
test_series = df.loc[0]                     # 索引是数字，只有一个则返回series，多个将会返回dataframe
test_series = df.loc['test_index']          # 索引是字符串，传数字会报错，只有一个则返回series，多个将会返回dataframe
test_df = df.loc[0:5]                       # 多行，返回dataframe
test_df = df.loc[0:5, "test"]               # 0:5行，test列，一列返回series
test_df = df.loc[0:5, ["test", "test2"]]    # 0:5行，test和test2列，多列列返回series
## iloc只能传入行号，只能传入数字，如果索引不是数字，也只能传入行号
test_series = df.iloc[0]     # 只能传入数字，传入行号，不管索引是不是数字，只有一个则返回series，多个将会返回dataframe
test_df = df.iloc[0:5]       # 多行，返回dataframe
test_df = df.iloc[0:5, 0]    # 0:5行，0列，一列返回series
test_df = df.iloc[0:5, 0:1]  # 0:5行，0:1列，多列列返回series

# 提取单个值，可以用loc和iloc，但是速度不如at和iat
## at和iat对应loc和iloc，但是必须定位到一个特定的元素，要求行列并且只能传入单值
tmp = df.at["index", "columns"]
tmp = df.iat[0, 0]

# 直接索引
## df[]只能进行行选择，或列选择，不能同时进行列选择，列选择只能是列名。行号和区间选择只能进行行选择。当index和columns标签值存在重复时，通过标签选择会优先返回行数据。df.只能进行列选择，不能进行行选择
## 同上，一列或一行返回series，多行或多列返回dataframe，不能选择单个
tmp = df["index"]
tmp = df["columns"]
tmp = df.test       # 只能用于列选择
tmp = df[0:3]       # 只能用于行选择

# 获取列名
columns_value = df.columns          # 返回 <class 'pandas.core.indexes.base.Index'> 格式
columns_value = df.columns.values   # 返回list格式

#################### 改 ######################
# 修改某个值，查到就能改，用上面查找的方法直接改
df.at["index", "columns"] = 5
# 修改列名
df.rename(columns={'test':'aaa'}, inplace=True)     # 将test改为aaa
# 排序
## by为依据columns或index，需要指定axis，0是行排，1是列排，默认为0
## ascending，True升序，False降序
## inplace是否改变原数据
## na_position，缺失值所在位置，last或first
df.sort_values(by=["status"], ascending=[False], inplace=True)  # 依据status列进行行排序，降序，替换原数据
```

- 遍历

```python
# 行遍历
for index, row in df.iterrows():
    # index为索引值，row为series，每一行转成series
    print(index, row)
```

- 数据对比

```python
# 比较两个数据
## 内置函数merge速度比自己写快得多，原始用处是合并两个数据，但是可以用于数据对比
## left和right指定需要对比的两个数据
## how分为四种情况，left以left数据为基准，数据不会超过left数量；right同理；outer，并集；inner，交集
tmp = pd.merge(left = old_data, right = new_data, how = "outer", indicator = True)
old_data = tmp.loc[tmp._merge == 'left_only', :].drop(columns='_merge')
new_data = tmp.loc[tmp._merge == 'right_only', :].drop(columns='_merge')
```

#### 转换

```python
# list或者ndarray转成dataframe
test_list = [[1, 2, 3], [4, 5, 6]]
test_df = pd.DataFrame(test_list, columns=['a', 'b', 'c'])

# 转成ndarray多维矩阵形式
test_ndarray = df.values
```

### series相关操作

类似于python自带的dict类型

#### 信息

```python
import pandas as pd

test_series = pd.Series(xxx)
test_series.describe()   # 显示数据的各种统计数据包（中位数、平均数、标准差、最值等）
test_series.value_counts()  # 统计各个值出现的次数
```

#### list操作

```python
    # 查
    # 查一个值
    test_series['test']
    # 查找相关条件的值，返回list
    test_list = test_series[test_series.index == 'test']
```

### 写入excel

#### 不显示表头和索引

```python
    # header=None 不显示表头，index=False 不显示索引
    output_df.to_excel("out.xlsx", header=None, index=False)
```

#### 写入同一文件多个sheet

需要安装openpyxl，默认使用此module

```python
    writer = pd.ExcelWriter("output.xlsx")
    output1_df.to_excel(writer, sheet_name='output1_df')
    output2_df.to_excel(writer, sheet_name='output2_df')
    output3_df.to_excel(writer, sheet_name='output3_df')
    writer.save()
```

#### 出现非法字符问题，可以换用xlsxwriter作为写入

```python
    writer = pd.ExcelWriter("output.xlsx", engine='xlsxwriter')
    output1_df.to_excel(writer, sheet_name='output1_df')
    output2_df.to_excel(writer, sheet_name='output2_df')
    output3_df.to_excel(writer, sheet_name='output3_df')
    writer.save()
```

## 2.2. numpy

### 1. 内置函数

#### 1.1. 数学函数汇总

```python
from numpy import *
# e^12
exp(12)
```

#### 1.2. argmax获取最大值的位置

```python
import numpy as np

print(np.argmax([1, 2, 4, 3]))  # 2
```

### numpy数据存取

numpy有一个可以直接将numpy数组矩阵按照原来的格式储存和读取的函数

```python
    import numpy as np

    # 储存data
    np.save("file directory", data)

    # 读取data
    data = np.load("file directory")
```

这个方法可以直接将shape也存下来，不需要考虑格式、类型等，缺点是文件会有点大。

### ndarray 矩阵

#### reshape 重置矩阵形状

从最外层开始重置矩阵形状，默认按行读取，-1代表未知数量，由numpy自动计算

```python
    test_ndarray = test_ndarray.reshape(-1, 28, 28, 1)
    print(test_ndarray.shape)   #out: (42000, 28, 28, 1)
```

### 随机数

- 随机种子

```python
    import numpy as np

    np.random.seed(2)
```

### linespace 列表

```python
    import numpy as np

    y = np.linspace(m, n, z) # 在[m, n]等距离取z个点
    x = np.linspace(m, n) # 同上，z默认取50
```

## 2.3. matplotlib

### pyplot 画图

#### 新开一个页面 figure

```python
    import matplotlib.pyplot as plt

    plt.figure()
    ...
    plt.figure()
    ...
    plt.show()
```

#### 一页多图 subplot

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

#### 页面属性更改

```python
    import matplotlib.pyplot as plt

    plt.figure("abc") # 整个图表名字
    ...
    plt.xlabel("x") # 横坐标名称
    plt.ylabel("y") # 纵坐标名称
    plt.title("y = f(x)") # 当前图的名字
    plt.show()
```

#### stem 散点图

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

## 2.4. seaborn

### 画图统计向量中值的出现次数 countplot

```python
    import matplotlib.pyplot as plt
    import seaborn as sns
    ...
    sns.countplot(Y_train)
    plt.show()
```

效果图

<img src = "2018_11_29_03.png">

## 2.5.  scipy

### fftpack

#### fft 快速傅里叶变换

```python
    from scipy.fftpack import fft

    x = np.linspace(0, 100, 32)
    y = fft(x) # 得到x的32点fft
```
