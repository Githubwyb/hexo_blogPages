---
title: python学习笔记
date: 2018-09-16 13:17:51
tags:
categories: [Program, Python]
top: 96
---

# 环境

```
Python 3.6.4
```

# 一、安装配置

## 1. pip包管理器

### 1.1. 安装

安装相应python的版本的pip

```shell
sudo apt install python3-pip
ln -s /usr/bin/pip3 /usr/bin/pip    # 将pip3使用pip命令代替，创建一个快捷方式
```

### 1.2. 配置pip源

```shell
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 1.3. 生成requirements.txt

```shell
pip freeze > requirements.txt
```

### 1.4. pip安装失败，无法再使用pip怎么办

```shell
# 修复本地的pip
python -m ensurepip
```

## 2. jupyter notebook

### 2.1. 安装

使用pip安装没有找到怎么命令行调用，使用apt可以直接命令行调用

```shell
sudo apt install jupyter-notebook
```

### 2.2. 配置远程访问

```shell
~|⇒ jupyter notebook --generate-config      # 生成jupyter配置文件
Writing default config to: /home/you/.jupyter/jupyter_notebook_config.py
~|⇒ jupyter notebook password               # 设置远程访问密码
Enter password:
Verify password:
[NotebookPasswordApp] Wrote hashed password to /home/wangyubo/.jupyter/jupyter_notebook_config.json
```

编辑`jupyter_notebook_config.py`，找到以下配置，改成这样

```python
c.NotebookApp.ip = '*'              # 允许所有ip访问
c.NotebookApp.open_browser = False  # 不自动打开浏览器
c.NotebookApp.port = 8888           #可自行指定一个端口, 访问时使用该端口
```

### 2.3. 设置matplotlib的图像显示大小

```python
plt.rcParams["figure.figsize"] = (15.0, 12.0)
```

## 3. pylint 代码风格检查配置

# 二、语法相关

参考文档: [W3Cschool][1]、[Runoob][2]

[1]: https://www.w3cschool.cn/uqmpir/
[2]: http://www.runoob.com/python/python-tutorial.html

## 1. 风格和编码

### 1.1. 指定编码格式

源文件第一行或第二行直接定义

```python
# coding=utf-8
```

或者

```python
# -*- coding:utf-8 -*-
```

### 1.2. 代码风格

- 函数和变量使用下划线命名
- 全局常量使用大写字母

## 2. 变量

### 2.1. None

- None是一个特殊的常量。
- None和False不同。
- None不是0。
- None不是空字符串。
- None和任何其他的数据类型比较永远返回False。
- None有自己的数据类型NoneType。
- 你可以将None赋值给任何变量，但是你不能创建其他NoneType对象。

python中矩阵索引使用None表示此维度不切片，同样意味着此维度大小未知

### 2.2. del 删除一个变量释放空间

```python
del var
```

## 3. 字符串操作

### 3.1. 替换字符串

```python
str = 'abc'
# 替换字符串，以返回值形式返回，不会在原数据做更改
str = str.replace('b', '')
```

### 3.2. 删除首位空格（包括换行符）

```python
str = str.dropip()
```

### 3.3. 和ASCII码之间的转换

```python
# 转ASCII码，仅单个字符
print(ord('a'))
# ASCII码转字符，仅单个ascii码
print(chr(97))
```
### 3.4. 字符串格式化

**(1) `f'xxx'`格式**

```python
a = 1
str_test = f'a = {a}'
```

**(2) `format()`格式**

1. 指定位置

```python
>>>"{} {}".format("hello", "world")    # 不设置指定位置，按默认顺序
'hello world'

>>> "{0} {1}".format("hello", "world")  # 设置指定位置
'hello world'

>>> "{1} {0} {1}".format("hello", "world")  # 设置指定位置
'world hello world'
```

2. 使用字典

```python
print("网站名：{name}, 地址 {url}".format(name="菜鸟教程", url="www.runoob.com"))

# 通过字典设置参数
site = {"name": "菜鸟教程", "url": "www.runoob.com"}
print("网站名：{name}, 地址 {url}".format(**site))

# 通过列表索引设置参数
my_list = ['菜鸟教程', 'www.runoob.com']
print("网站名：{0[0]}, 地址 {0[1]}".format(my_list))  # "0" 是必须的
```

3. 使用格式

| 数字       | 格式                 | 输出         | 描述                          |
| ---------- | -------------------- | ------------ | ----------------------------- |
| 11         | `'{:b}'.format(11)`  | 1011         | 二进制                        |
| 11         | `'{:d}'.format(11)`  | 11           | 十进制                        |
| 11         | `'{:o}'.format(11)`  | 13           | 八进制                        |
| 11         | `'{:x}'.format(11)`  | b            | 小写十六进制                  |
| 11         | `'{:X}'.format(11)`  | 0xb          | 大写十六进制                  |
| 11         | `'{:#x}'.format(11)` | 0Xb          | 补 0x 小写十六进制            |
| 11         | `'{:#X}'.format(11)` | 0XB          | 补 0X 大写写十六进制          |
| 3.1415926  | `{:.2f}`             | 3.14         | 保留小数点后两位              |
| 3.1415926  | `{:+.2f}`            | +3.14        | 带符号保留小数点后两位        |
| -1         | `{:+.2f}`            | -1.00        | 带符号保留小数点后两位        |
| 2.71828    | `{:.0f}`             | 3            | 不带小数                      |
| 5          | `{:0>2d}`            | 05           | 数字补零 (填充左边, 宽度为 2) |
| 5          | `{:x<4d}`            | 5xxx         | 数字补 x (填充右边, 宽度为 4) |
| 10         | `{:x<4d}`            | 10xx         | 数字补 x (填充右边, 宽度为 4) |
| 1000000    | `{:,}`               | 1,000,000    | 以逗号分隔的数字格式          |
| 0.25       | `{:.2%}`             | 25.00%       | 百分比格式                    |
| 1000000000 | `{:.2e}`             | 1.00e+09     | 指数记法                      |
| 13         | `{:>10d}`            | `--------13` | 右对齐 (默认, 宽度为 10)      |
| 13         | `{:<10d}`            | `13--------` | 左对齐 (宽度为 10)            |
| 13         | `{:^10d}`            | `----13----` | 中间对齐 (宽度为 10)          |

### 3.4. 二进制和字符串转换

- 使用`str.encode()`将字符串转成二进制
- 使用`str.decode()`将二进制转成字符串

```python
>>> str = b'fW0v6MG1C3\n/UrT6bdQ==\n'
>>> str = str.decode(encoding='utf8')
>>> print(str)
fW0v6MG1C3
/UrT6bdQ==

>>> str = str.encode(encoding='utf8')
>>> print(str)
b'fW0v6MG1C3\n/UrT6bdQ==\n'
```

### 3.5. 原始字符串

- 三个双引号可以输出原始字符串，换行符空格都会保留

```python
str = """CREATE TABLE `emm_data` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `path` varchar(512) COLLATE utf8mb4_bin NOT NULL,
  `size` int unsigned NOT NULL,
  `modify` date DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `emm_data_path` (`path`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin"""
```

## 4. list操作

### 4.1. 排序

```python
# 正向
>>> arr = [3, 2, 1]
>>> arr.sort()
>>> print(arr)
[1, 2, 3]
# 反向
>>> arr.sort(reverse=True)
>>> print(arr)
[3, 2, 1]

# 处理dict类型
>>> arr = [{"name": "c"}, {"name": "b"}, {"name": "a"}]
>>> arr.sort(key=lambda item: item["name"])
>>> print(arr)
[{'name': 'a'}, {'name': 'b'}, {'name': 'c'}]
```

### 4.2. 去重

- 使用set的特性去重

```python
>>> a = [1, 1, 2, 2, 3, 3]
>>> a = list(set(a))
>>> print(a)
[1, 2, 3]
```

### 4.3. 遍历

```python
listValues = list(xxx)

for value in listValues:
    print(value)
    value = 'a'     # list本身不会修改

for index in range(len(listValues)):
    print(listValues[index])
    listValues[index] = 'a'         # list本身会修改

for index, value in enumerate(listValues):
    print(value)
    value = 'a'                 # list不会修改
    listValues[index] = 'a'     # list会修改

```

- 普通的`for value in listValues:`无法修改list的值，list值修改只能用index的方式

### 4.4. 索引

- 没有冒号就是单纯找下标

```python
>>> a = [1, 2, 3, 4, 5, 6]
# 正序下标
>>> a[2]
3
# 超出范围
>>> a[12]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
# 倒序下标
>>> a[-1]
6
```

- 单冒号输出数组

```python
>>> a = [1, 2, 3, 4, 5, 6]
# 取1到2，不包含2为下标的数组
>>> a[1:2]
[2]
# 右边没有，代表到末尾
>>> a[-1:]
[6]
# 左边没有，代表从头开始
>>> a[:4]
[1, 2, 3, 4]
# 超出边界相当于没写
>>> a[1:10]
[2, 3, 4, 5, 6]
# 相等是空数组
>>> a[1:1]
[]
# 倒序返回空
>>> a[2:1]
[]
# 倒序返回空
>>> a[-1:1]
[]
```

- 双冒号相当于在后面加了一个间隔

```python
>>> a = [1, 2, 3, 4, 5, 6]
# 反转数组
>>> a[::-1]
[6, 5, 4, 3, 2, 1]
# 挑奇数位置
>>> a[::2]
[1, 3, 5]
# 倒序，下标不倒序为空
>>> a[0:4:-1]
[]
# 4开始倒序到0，不包含0
>>> a[4:0:-1]
[5, 4, 3, 2]
```

## 4. 类

### 4.1. 类的几个特殊函数

**1. 构造函数**

```python
class Test(object):
    def __init__(self, a):
        self.a = a

tmp = Test(1)
tmp = Test()        # 报错，需要传入1个参数
```

**2. 字符串输出**

- 将类作为字符串时，会自动调用此函数输出

```python
class Test(object):
    def __str__(self):
        return f'ccccccc Test xxxxxxx'

tmp = Test()
print(tmp)
print(str(tmp))
```

### 4.2. 多态

**1. 虚函数**

```python
from abc import abstractmethod

class Base(object):
    @abstractmethod
    def test_func(self):
        pass

```

## 5. set 集合

### 5.1. 基本操作

```python
########## 初始化 ##########
# 通过list初始化
>>> a_list = [1, 1, 2, 2, 3]
>>> a_set = set(a_list)
>>> print(a_set)
{1, 2, 3}
# 直接初始化
>>> a_set = set()

########## 添加元素 ##########
>>> a_set = set()
>>> a_set.add(4)
>>> a_set.add(4)
>>> a_set.add(3)
>>> a_set.add(2)
>>> a_set.add(3)
>>> print(a_set)
{2, 3, 4}
```

## 6. 函数

### 6.1. lambda 匿名函数

```python
lambda arg1, arg2, ...argN : expression using arguments
```

#### 注意

- lambda 函数不能包含命令，包含的表达式不能超过一个。不要试图向 lambda 函数中塞入太多的东西；如果你需要更复杂的东西，应该定义一个普通函数，然后想让它多长就多长。
- 就lambda而言，它并没有给程序带来性能上的提升，它带来的是代码的简洁。

### 6.2. 不定参数



## 7. try 异常处理

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
        raise Exception("Invalid level!")
        # 触发异常后，后面的代码就不会再执行
try:
    mye(0)            # 触发异常
except Exception as err:
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

## 8. with 上下文

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

## 9. print 打印操作

### 9.1. 一些基本操作

- `\r`: 将光标定位到行首
- `\b`: 光标前移一个字符
- `end=""`: 参数end控制结束符

## 10. import 导入模块

- 导入会先找脚本所在同级目录，再去找系统path下

### 10.1. 使用字符串导入

```python
mod_name = "json"
# __import__方法
__import__(mod_name)
# exec的方法
exec('import ' + mod_name)
# import_module 官方推荐
import importlib
string = importlib.import_module(mod_name)
```

## 11. 文件操作

### 11.1. 文件读取

- 读取成字符串，使用`r`
- 读取成二进制，使用`rb`

#### (1) read()方法

- 返回string到变量中，会读取所有内容
- 比较占用内存

```python
with open('aa', 'r') as f:
    content = f.read()
```

#### (2) readline()方法

- 每次只读一行
- 比较慢

```python
with open('aa', 'r') as f:
    line = f.readline()
    while line:
        print line
        line = f.readline()
```

#### (3) readlines()方法

- 一次全部读取，按照换行返回list

```python
with open('aa', 'r') as f:
    for line in f.readlines():
        print(line)         # line带"\n"
```

#### (4) 直接对f进行遍历

```python
with open('aa', 'r') as f:
    try:
        for line in f:
            do_somthing_with(line)      # line带"\n"
    finally:
        f.close()
```

## 12. 内置函数

### (1) 进制转换

**10进制转其他进制**

```python
>>> hex(446)
'0x1be'
>>> bin(446)
'0b110111110'
>>> oct(446)
'0o676'
```

**其他进制转10进制**

```python
>>> int('0x1be', 16)
446
>>> int('0o676', 8)
446
>>> int('0b110111110', 2)
446
```
或者
```python
>>> a=0x1be
>>> print(a)
446
>>> a=0o676
>>> print(a)
446
>>> a=0b110111110
>>> print(a)
446
```

### (2) locals() 获取本地变量dict

**判断变量是否定义**

```python
if 'aaa' in locals().keys():
    print('aaa is defined')
```

### (3) isinstance() 类型比较函数

- pylint不建议使用`type(xxx) == type("")`的格式判断特定的类型

```python
if isinstance("aaa", str):
    pass
if isinstance(123, int):
    pass
```

## 13. dict操作

### 1) dict合并

```python
dict1 = {"a": 1, "b": 2}
dict2 = {"a": 2, "c": 3}
dict3 = {**dict1, **dict2}  # 用dict2更新dict1
```

# 三、系统内置module介绍

## 1. 操作系统组件 os

### 1.1. 路径相关操作

#### (1) 获取当前路径

```python
import os

print(os.getcwd())                  #获取当前工作目录路径
print(os.path.abspath('.'))         #获取当前工作目录路径
print(os.path.abspath('test.txt'))  #获取当前目录文件下的工作目录路径
print(os.path.abspath('..'))        #获取当前工作的父目录 ！注意是父目录路径
print(os.path.abspath(os.curdir))   #获取当前工作目录路径
```

#### (2) 改变当前路径

```python
import os

os.chdir(path)
```

#### (3) 遍历目录

此命令会遍历目录下的所有文件包括子文件

```python
import os
for dirPath, dirNames, fileNames in os.walk('./'):
    print('dirPath', dirPath)       # 当前遍历的目录
    print('dirNames', dirNames)     # 该目录下所有的文件夹名字组成的列表
    print('fileNames', fileNames)   # 该目录下所有的文件名字组成的列表
```

### 1.2. 调用可执行文件

#### (1) 获取输出结果

```python
import os

f = os.popen("(cmd) (param)")
data = f.readlines()
f.close()
```

#### (2) 获取返回值

```python
import os

r_v = os.system("(cmd) (param)")
print r_v
```

## 2. 系统组件 sys

### 内置常量

```python
import sys

print(sys.executable)       # 当前python命令所在路径，/usr/bin/python
```

## 3. json

### 3.1. json.loads() 字符串转dict

```python
import json

# 字符串到dict
dict_data = json.loads('{"a": 1}')
```

### 3.2. json.dumps() dict转json

**原型**

```python
def dumps(obj, *, skipkeys=False, ensure_ascii=True, check_circular=True,
        allow_nan=True, cls=None, indent=None, separators=None,
        default=None, sort_keys=False, **kw):
    """Serialize ``obj`` to a JSON formatted ``str``.

    If ``skipkeys`` is true then ``dict`` keys that are not basic types
    (``str``, ``int``, ``float``, ``bool``, ``None``) will be skipped
    instead of raising a ``TypeError``.

    If ``ensure_ascii`` is false, then the return value can contain non-ASCII
    characters if they appear in strings contained in ``obj``. Otherwise, all
    such characters are escaped in JSON strings.

    If ``check_circular`` is false, then the circular reference check
    for container types will be skipped and a circular reference will
    result in an ``OverflowError`` (or worse).

    If ``allow_nan`` is false, then it will be a ``ValueError`` to
    serialize out of range ``float`` values (``nan``, ``inf``, ``-inf``) in
    strict compliance of the JSON specification, instead of using the
    JavaScript equivalents (``NaN``, ``Infinity``, ``-Infinity``).

    If ``indent`` is a non-negative integer, then JSON array elements and
    object members will be pretty-printed with that indent level. An indent
    level of 0 will only insert newlines. ``None`` is the most compact
    representation.

    If specified, ``separators`` should be an ``(item_separator, key_separator)``
    tuple.  The default is ``(', ', ': ')`` if *indent* is ``None`` and
    ``(',', ': ')`` otherwise.  To get the most compact JSON representation,
    you should specify ``(',', ':')`` to eliminate whitespace.

    ``default(obj)`` is a function that should return a serializable version
    of obj or raise TypeError. The default simply raises TypeError.

    If *sort_keys* is true (default: ``False``), then the output of
    dictionaries will be sorted by key.

    To use a custom ``JSONEncoder`` subclass (e.g. one that overrides the
    ``.default()`` method to serialize additional types), specify it with
    the ``cls`` kwarg; otherwise ``JSONEncoder`` is used.

    """
```

**实例**

```python
import json

# dict到字符串，是否排列key
json_str = json.dumps(dict_data, sort_keys=True)
# dict到字符串，是否排列key，按照格式化输出，缩进为2，字符串原样输出，不转ascii
json_str = json.dumps(dict_data, sort_keys=True, indent=2, ensure_ascii=False)
```

## 4. 加载C/C++的so库 ctypes

### 4.1. 基本操作

```python

```

## 5. 时间库 time

### 5.1. 当前时间操作

```python
>>> import time
>>> print(time.time())
1641548351.8147426
>>> time.strftime("%Y-%m-%d %X")
'2022-01-07 17:39:38'
```

### 5.2. struct_time操作

```python
import time
########## struct_time创建 ##########
# 时间字符串转struct_time
time_s = time.strptime("2020-01-02 19:00:00", "%Y-%m-%d %H:%M:%S")
# 时间戳转struct_time
time_s = time.localtime(1577962800.0)

########## struct_time转其他 ##########
# 转时间戳，ms
timestamp_float = time.mktime(time_s)
timestamp_s = int(timestamp_float)
# 转时间字符串
time_str = time.strftime("%Y-%m-%d %H:%M:%S", time_s)
```

### 5.3. sleep 睡眠

```python
# 睡眠1.5s
time.sleep(1.5)
```

## 6. math 数学库

### 6.1. 取整

```python
>>> import math
# 向下
>>> math.floor(2.9)
2
# 向上
>>> math.ceil(2.1)
3
# 四舍五入，2.5可能认为是2.49999999
>>> round(2.5)
2
>>> round(2.6)
3
```

## 7. hashlib hash算法库

### 7.1. sha256

#### (1) 计算文件的sha256值

```python
import hashlib

sha256_handle = hashlib.sha256()
with open(file_path, 'rb') as f:
    sha256_handle.update(f.read())
    hash_value = sha256_handle.hexdigest()

print(hash_value)
```

## 8. functools 内置的一些函数

### 8.1. reduce

- 就是python2的reduce
- 第一个x和y为前两个元素，计算结果作为下一个x，下一个元素作为y

```python
>>> import functools
>>> a = [1, 2, 3, 4, 5, 6]
>>> result = functools.reduce(lambda x, y: x + y, a)
>>> print(result)
21
```

# 四、好用的module推荐

## module对应pack

| module | pack          |
| ------ | ------------- |
| cv2    | opencv-python |
| PIL    | pillow        |
| dns    | dnspython     |

## 1. 数据处理

数据处理相关(pandas、numpy等)的在[另一篇博客详解](/blogs/2018-10-25-dataScienceStudyPython)

## 2. 压缩

### zip格式

创建压缩

```python
import zipfile

z = zipfile.ZipFile(folderName + '.zip', 'w', zipfile.ZIP_STORED) # 创建文件
z.write('test.txt')             # 写入一个文件
z.write('dirName')              # 写入一个空文件夹
z.write('folderName/fileName')  # 可以在没有上级目录的情况下直接写一个文件
z.close()                       # 关闭文件
```

压缩目录下的所有文件

```python
import zipfile
import os

z = zipfile.ZipFile('dirName.zip', 'w', zipfile.ZIP_STORED)
for dirPath, dirNames, fileNames in os.walk('dirName'):
    for fileName in fileNames:
        z.write(dirPath + '/' + fileName)
        print(dirPath + '/' + fileName)

    # 防止空文件夹没有被添加入压缩文件
    for dirName in dirNames:
        z.write(dirPath + '/' + dirName)
        print(dirPath + '/' + dirName + '/') # 区别于文件的打印
z.close()
```

## 3. 汉字转拼音 xpinyin

简单用例

```python
import xpinyin
pin = xpinyin.Pinyin()
test1 = pin.get_pinyin("大河向东流")   #默认分割符为-
print(test1)

test2 = pin.get_pinyin("大河向东流", "")
print(test2)
```

## 4. 参数解析器 argparse

很方便的管理命令行参数，支持选项添加

## 5. 文件、文件夹对比 filecmp

一个开源比较好的文件夹对比仓库: [https://github.com/Pixinn/compare_folders](https://github.com/Pixinn/compare_folders)

## 6. git操作 gitpython

## 7. url请求操作 requests

### 7.1. 请求基本操作

```python
import requests

# 关闭https请求打印的warning
requests.packages.urllib3.disable_warnings()

# params对应请求后面的参数 https://127.0.0.1:1234?a=1&b=2
params = {
    "a": 1,
    "b": 2
}
# data对应表单数据
data = {
    "ab": 1,
    "cd": 2
}
# json对应body的json
json_data = {
    "abc": 1,
    "cde": 2
}
r = requests.get("https://127.0.0.1:1234", params=params)
r = requests.post("https://127.0.0.1:1234", params=params, data=data, json=json_data)
```

### 7.2. 下载文件

```python
import requests

url = "https://image.csslcloud.net/image.jpg".format(i)
down_res = requests.get(url=url)

with open('./tmp/test.png', "wb") as code:
    code.write(down_res.content)
```

## 8. 接口测试框架 robot framework

### 8.1. 命令使用

```shell
# 执行tag为xxx的用例
robot -i xxx caseDir/
# 执行tag为xxx的用例，日志级别为TRACE，默认显示级别INFO
robot -i xxx -L TRACE:INFO caseDir/
```

## 9. 浏览器自动化操作 selenium

见[python selenium使用记录](/blogs/2022-02-27-selenium)

## 10. pdf编辑 PyPDF2

### 10.1. 删除一页pdf

```python
from PyPDF2 import PdfFileWriter, PdfFileReader

output = PdfFileWriter()
input1 = PdfFileReader(open("test.pdf", "rb"))


def delete_pdf(index: list):
    pages = input1.getNumPages()

    for i in range(pages):
        if i+1 in index:
            continue
        output.addPage(input1.getPage(i))

    outputStream = open("test1.pdf", "wb")
    output.write(outputStream)


delete_pdf([1])
```

## 11. pymysql mysql操作库

```python
import pymysql

# 连接数据库
mysql_conn = pymysql.connect(host='1.0.2.3', port=3306, user='root', password='123456', db='web_analyze')

# 执行sql语句
sql = """CREATE TABLE `emm_data` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `path` varchar(512) COLLATE utf8mb4_bin NOT NULL,
  `size` int unsigned NOT NULL,
  `modify` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `emm_data_path` (`path`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin"""
try:
    with mysql_conn.cursor() as cursor:
        cursor.execute(sql)
    mysql_conn.commit()
except Exception as e:
    print(e)
    mysql_conn.rollback()
```


# 小技巧和踩坑记

## 1. 判断文件是否为二进制

```python
import codecs
import os

#: BOMs to indicate that a file is a text file even if it contains zero bytes.
_TEXT_BOMS = (
    codecs.BOM_UTF16_BE,
    codecs.BOM_UTF16_LE,
    codecs.BOM_UTF32_BE,
    codecs.BOM_UTF32_LE,
    codecs.BOM_UTF8,
)

def is_binary_file(file_path):
    file_path = os.path.abspath(file_path)

    with open(file_path, 'rb') as file:
        initial_bytes = file.read(8192)

    return not any(initial_bytes.startswith(bom) for bom in _TEXT_BOMS) and b'\0' in initial_bytes
```

## 2. 动态添加PYTHONPATH

```python
import sys
sys.path.append('/xxx/xxx')
```

## 3. 跨平台文件操作需要确定编码

- windows的默认编码是gbk，如果是写文件，确定是utf8的一定要在open的时候确定是utf8编码

## 4. pip升级后运行失败

apt安装的pip一般只有8.1版本，而最新已经有19.1版本了，所以安装后使用pip自己更新自己

```shell
pip install -U pip
```

更新后执行`pip -V`会报错，因为老的pip文件api和新的没对上，需要修改`/usr/bin/pip`如下，主要是`main`需要换成`__main__`

```python
#!/usr/bin/python3
# GENERATED BY DEBIAN

import sys

# Run the main entry point, similarly to how setuptools does it, but because
# we didn't install the actual entry point from setup.py, don't use the
# pkg_resources API.
from pip import __main__
if __name__ == '__main__':
    sys.exit(__main__._main())
```
