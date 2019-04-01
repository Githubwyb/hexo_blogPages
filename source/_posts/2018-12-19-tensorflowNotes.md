---
title: tensorflow学习笔记
date: 2018-12-19 14:00:46
tags: 
categories: [Program, Python]
---

# TensorFlow是什么

- TensorFlow是谷歌基于DistBelief进行研发的第二代人工智能学习系统
- 用于语音识别或者图像识别多项机器学习和深度学习领域
- 将复杂的数据结构传输到人工智能神经网中进行分析和处理过程的系统、
- 支持CNN、RNN和LSTM算法，都是Image、Speech和NLP最流行的深度神经网络模型

# tensorflow语法

语法细节可以看[W3Cschool的tensorflow官方文档][1]，下面记录的是自己觉得需要记录的东西

[1]: https://www.w3cschool.cn/tensorflow_python/

## 变量

在TensorFlow的世界里，变量的定义和初始化是分开的，所有关于图变量的赋值和计算都要通过tf.Session的run来进行。想要将所有图变量进行集体初始化时应该使用tf.global_variables_initializer。

```python
    tf.Variable(initializer, name) #参数initializer是初始化参数，name是可自定义的变量名称
```

实例

```python
    import tensorflow as tf

    v1 = tf.Variable(tf.random_normal(shape=[4, 3], mean=0, stddev=1), name='v1')
    v2 = tf.Variable(tf.constant(2), name='v2')
    v3 = tf.Variable(tf.ones([4, 3]), name='v3')
    with tf.Session() as sess:
        sess.run(tf.global_variables_initializer())
        print(sess.run(v1))
        print(sess.run(v2))
        print(sess.run(v3))
```

### 变量命名空间 variable_scope

variable_scope定义了变量的命名空间

#### 示例1-如何创建一个新变量：

```python
    with tf.variable_scope("foo"):
        with tf.variable_scope("bar"):
            v = tf.get_variable("v", [1])
```

其中`v.name == "foo/bar/v:0"`

#### 示例2-共享变量AUTO_REUSE：

```python
    def foo():
        with tf.variable_scope("foo", reuse=tf.AUTO_REUSE):
            v = tf.get_variable("v", [1])
        return v


    v1 = foo()  # Creates v.
    v2 = foo()  # Gets the same, existing v.
    print(v1.name) # foo/v:0
    print(v2.name) # foo/v:0
```

其中`v1 == v2`

#### 示例3-使用reuse=True共享变量：

```python
    with tf.variable_scope("foo"):
        v = tf.get_variable("v", [1])
    with tf.variable_scope("foo", reuse=True):
        v1 = tf.get_variable("v", [1])
```

其中`v == v1`

#### 示例4-通过捕获范围并设置重用来共享变量：

```python
    with tf.variable_scope("foo") as scope:
        v = tf.get_variable("v", [1])
        scope.reuse_variables()
        v1 = tf.get_variable("v", [1])
```

其中`v == v1`

#### 为了防止意外共享变量，我们在获取非重用范围中的现有变量时引发异常。

```python
    with tf.variable_scope("foo"):
        v = tf.get_variable("v", [1])
        v1 = tf.get_variable("v", [1])
```

将会触发`ValueError("... v already exists ...")`异常

#### 同样，我们在尝试获取重用模式中不存在的变量时引发异常。

```python
    with tf.variable_scope("foo", reuse=True):
        v = tf.get_variable("v", [1])
```

将会触发`ValueError("... v does not exists ...")`异常

## 矩阵

### zeros矩阵

```python
    import tensorflow as tf

    v1 = tf.Variable(zeros([2, 3])， name='v1') #两行三列矩阵

    with tf.Session() as sess:
        sess.run(tf.global_variables_initializer())
        print(sess.run(v1))
```

输出

```shell
    [[0. 0. 0.]
     [0. 0. 0.]]
```