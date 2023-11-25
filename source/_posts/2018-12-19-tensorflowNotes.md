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

语法细节可以看[tensorflow官方文档](https://tensorflow.google.cn/guide)，下面记录的是自己觉得需要记录的东西

## keras

keras是tensorflow的上层api，可以使用keras方便的实现神经网络的搭建

### 数据处理

#### to_categorical

将标签转化成容易输出的类型，如10个数字作为输出，可转成[1, 0, ..., 0]表示0这种形式

```python
import tensorflow as tf
tmp = tf.keras.utils.to_categorical(Y_train, num_classes = 10)
```