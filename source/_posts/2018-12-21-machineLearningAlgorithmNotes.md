---
title: 机器学习中的算法笔记
date: 2018-12-21 10:15:36
tags: [AI, algorithm]
categories: [notes, study]
---

# softmax函数

## 函数形式

$$ S_i = \frac{e^{V_i}}{\sum\nolimits_{i}^{C}e^{V_i}} $$

其中， $ V_i $ 是分类器前级输出单元的输出。$ i $ 表示类别索引，总的类别个数为 $ C $ 。$ S_i $ 表示的是当前元素的指数与所有元素指数和的比值。Softmax 将多分类的输出数值转化为相对概率，更容易理解和比较。

## 实例

一个多分类问题，C = 4。线性分类器模型最后输出层包含了四个输出值，分别是：

$$ V = \begin{bmatrix} -3 \\ 2 \\ -1 \\ 0 \end{bmatrix} $$

经过Softmax处理后，数值转化为相对概率：

$$ S = \begin{bmatrix} 0.0057 \\ 0.8390 \\ 0.0418 \\ 0.1135 \end{bmatrix} $$

很明显，Softmax 的输出表征了不同类别之间的相对概率。我们可以清晰地看出，S1 = 0.8390，对应的概率最大，则更清晰地可以判断预测为第2类的可能性更大。Softmax 将连续数值转化成相对概率，更有利于我们理解。

# 感知器（神经元）

- 神经网络的组成单元——神经元。
- 神经元也叫做感知器。

## 模型

<img src = "2018_12_24_01.png">

## 函数形式

- **输入权值** 一个感知器可以接收多个**输入** $ (x_1, x_2, ..., x_n | x_i \in R) $，每个输入上有一个**权值** $ w_i \in R $，此外还有一个**偏置项** $ b \in R $ 。
- **激活函数** 感知器的激活函数可以有很多选择，比如我们可以选择下面这个**阶跃函数** $ f $ 来作为激活函数：
$$ f(z) = \left\{\begin{array}{ll}
    1 & {z > 0} \\
    0 & otherwise
\end{array}\right. $$
- **输出** 感知器的输出用下面的公式计算
$$ y = f(w \cdot x + b) $$

## 训练神经元

感知器规则

$$ w_i \leftarrow w_i + \Delta w_i \\
b \leftarrow b + \Delta b $$

其中

$$ \Delta w_i = \eta(t - y)x_i \\
\Delta b = \eta(t - y) $$

- $ t $ 为训练样本的实际值，也就是label
- $ y $ 为感知器输出值
- $ \eta $ 为学习速率，也就是rate

## 实例

python编写感知器实现and运算符

Github: <https://github.com/Githubwyb/zeroDeepLearning/tree/master/1.Perceptron>

# 线性单元

## 模型

<img src = "2018_12_26_02.png">

## 函数

与感知器一致，仅仅将激活函数改为线性函数 $ f(x) = x $

## 训练线性单元

### 目标函数

线性单元所要达到的目标是预测结果和实际结果相同，可以定义单个样本误差为：

$$ e = \frac{1}{2}(y - \bar y)^2 = \frac{1}{2}(y - w \cdot x)^2 $$

其中 $ y $ 为实际值，$ \bar y $ 为预测值。线性单元的目标为使 $ e $ 达到最小。

实际情况中，样本有很多个，需要使一批样本的误差达到最小，定义整体误差 $ E $ ：

$$ \begin{aligned}
    E & = e_1 + e_2 + ... + e_n \\\\
    & = \frac{1}{2}\sum_{i = 1}^{n}(y_i - \bar y_i)^2 \\\\
    & = \frac{1}{2}\sum_{i = 1}^{n}(y_i - w_i \cdot x_i)^2
\end{aligned} $$

训练线性单元目的即为将 $ E $ 变为最小

### 梯度下降算法

为使整体误差下降到最小，需要改变 $ w $ 使预测值更接近于真实值。可以定义整体误差 $ E $ 为 $ w $ 的函数，利用梯度的方法使整体误差取极小值点。由于计算机没办法计算梯度，但是计算能力强大，可以使用尝试法接近极小值。引入渐进到极小值及**梯度下降算法**的公式，对每个 $ w $ 来说：

$$ w_{new} = w_{old} -  \eta\nabla E(w_{old}) $$

对于 $ \nabla E $ ，有：

$$ \begin{aligned}
    \nabla E(w) & = \frac{1}{2}\sum_{i = 1}^{n}\frac{\partial}{\partial w}(y_i - \bar{y_i})^2 \\\\
    & = \frac{1}{2}\sum_{i = 1}^{n}\frac{\partial}{\partial w}(y_i - w \cdot x)^2 \\\\
    & = \frac{1}{2}\sum_{i = 1}^{n}[- 2(y_i - w \cdot x)x] \\\\
    & = -\sum_{i = 1}^{n}(y_i - \bar{y_i})x \\\\
\end{aligned} $$

所以，训练线性单元的规则为：

$$ w_{new} = w_{old} + \eta\sum_{i = 1}^{n}(y_i - \bar{y_i})x $$

## 实例

python编写线性单元实现线性预测

Github: <https://github.com/Githubwyb/zeroDeepLearning/tree/master/2.LinearUnit>
