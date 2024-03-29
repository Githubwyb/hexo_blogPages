---
title: 有关人工智能的随笔
date: 2021-11-03 16:14:06
tags: [AI]
categories: [Essay]
---

# 2021.11.03 神经网络算法研究和工程开发

## 1. 神经网络算法研究

- 对于神经网络的各个算法来讲，AI的算法主要依赖于对于现有场景的数学抽象，根据对应的数学模型找到特定的算法对应，根据输出反向求导进行训练，最终让算法中的参数趋于真实值
- 举个例子

**卷积神经网络**

- 图像的抽象就是像素点
- 图像中的信息依赖于各个像素点之间的位置关系
- 使用普通的相乘相加无法得到位置关系，但是使用卷积就可以将图像的位置关系进行抽象
- 对于一组图像，不同的图像具有不同的特征，所以卷积的filter也就需要多样化，所以卷积层存在一个高度表示不同的特征

**长短时记忆神经网络（LSTM）**

- 语言本身不能直接进入网络进行计算，所以对每个词组进行抽象编号，作为输入
- 语言具有上下文关系，所以需要进行循环处理
- 既要根据当前词的前后进行推导，又可能对于很早之前的语句进行推导，所以要引入一个长久保持的状态量，和一个短期的状态量
- 为了不让很早之前的无关量进行误导，所以要引入遗忘

## 2. 工程开发

- 在处理真实场景时，主要还是对输入进行抽象成数学模型
- 根据其中各个输入量之间的关系，调整模型或对数据预处理
- 整个神经网络搭建可能牵扯到多个类型网络结合等
- 各个超参数的调整却又需要根据实验进行分析总结，并不是直接就可以推导出来需要设置为多少
