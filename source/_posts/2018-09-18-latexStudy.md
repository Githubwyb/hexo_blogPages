---
title: latex笔记
date: 2018-09-18 15:16:39
tags: [latex]
categories: [notes, study]
---

# markdown中的latex

- 行内公式 `$\{[z-(1+\frac23x)y]\div 4\}$`: $\{[z-(1+\frac23x)y]\div4\}$
- 块级公式 `$$\sum_{i = 1}^n a_i=0$$`: $$\sum_{i = 1}^n a_i=0$$

# 公式格式记录

## 符号

参考文档: [常用数学符号的 LaTeX 表示方法][1]

[1]: http://www.mohu.org/info/symbols/symbols.htm

### 常用数学符号

- `$\sigma$`: $\sigma$
- 累加算子`$\Sigma$`: $\Sigma$
- 梯度算子`$\nabla$`: $\nabla$
- 偏导算子`$\partial$`: $\partial$
- 属于`$\in$`: $\in$
- 无穷`$\infty$`: $\infty$
- 任意`$\forall$`: $\forall$
- 约等于`$\approx$`: $\approx$
- 因为`$\because$`: $\because$
- 所以`$\therefore$`: $\therefore$
- 字母右上角一撇`$a^{\prime}$`: $a^{\prime}$

## 空格

| 描述         | 代码        | 效果          | 宽度         |
| ------------ | ----------- | ------------- | ------------ |
| 两个quad空格 | a \\qquad b | $a \\qquad b$ | 两个m的宽度  |
| quad空格     | a \\quad b  | $a \\quad b$  | 一个m的宽度  |
| 大空格       | a\\ b       | $a\\ b$       | 1/3m宽度     |
| 中等空格     | a\\;b       | $a\\;b$       | 2/7m宽度     |
| 小空格       | a\\,b       | $a\\,b$       | 1/6m宽度     |
| 没有空格     | ab          | $ab$          |              |
| 紧贴         | a\\!b       | $a\\!b$       | 缩进1/6m宽度 |

## 乘方

```latex
    $$ a^b $$
    $$ a^bc $$
    $$ a^{bc} $$
```

效果

$$ a^b $$
$$ a^bc $$
$$ a^{bc} $$

## 分数

```latex
    $$ \frac{A}{B} $$
    $$ \frac abc $$
```

效果

$$ \frac{A}{B} $$
$$ \frac abc $$

## 阶段函数表示

array后面的`ll`表示每一列的对齐方式，l：左对齐，c：居中，r：右对齐

```latex
    $$ f(z) = \left\{\begin{array}{ll}
        1 & {z > 0} \\
        0 & otherwise
    \end{array}\right. $$
```

效果

$$ f(z) = \left\\{\begin{array}{ll}
    1 & {z > 0} \\\\
    0 & otherwise
\end{array}\right. $$

## 公式推导换行等号对齐

```latex
    $$ \begin{aligned}
        f(x) & = (x + 1)^2 \\
        & = x^2 + 2x + 1
    \end{aligned} $$
```

$$ \begin{aligned}
    f(x) & = (x + 1)^2 \\\\
    & = x^2 + 2x + 1
\end{aligned} $$

## 公式编号

```latex
    % 手动编号，一个块级公式只能用一个tag
    $$ x = (tp_1, tp_2, ..., tp_{N - 1}, tp_N), \tag{1}$$
    $$ y = (0, 1, ..., 0, 0), \tag{2}$$

    % 自动编号
    $$\begin{equation}
    x^n+y^n=z^n
    \end{equation}$$
```

$$ x = (tp_1, tp_2, ..., tp_{N - 1}, tp_N), \tag{1}$$
$$ y = (0, 1, ..., 0, 0), \tag{2}$$

$$\begin{equation}
    x^n+y^n=z^n
\end{equation}$$

## log函数

```latex
    $$ log_A{B} $$
```

效果

$$ log_A{B} $$

## 累加符号

```latex
    - 行间公式 $ \sum_{N}^{n}a $
    - 独立公式 $$ \sum_{N}^{n}a $$
    - 行间公式使用上下形式 $ \sum\limits_{N}^{n}a $
    - 独立公式使用左右形式 $$ \sum\nolimits_{N}^{n}a $$
    - 不加大括号 $ \sum\limits_N^na $
    - 不加大括号 $ \sum\limits_Nb^na $
    - 加大括号 $ \sum\limits_{Nb}^{na} $
```

效果

- 行间公式 $ \sum_{N}^{n}a $
- 独立公式 $$ \sum_{N}^{n}a $$
- 行间公式使用上下形式 $ \sum\limits_{N}^{n}a $
- 独立公式使用左右形式 $$ \sum\nolimits_{N}^{n}a $$
- 不加大括号 $ \sum\limits_N^na $
- 不加大括号 $ \sum\limits_Nb^na $
- 加大括号 $ \sum\limits_{Nb}^{na} $

## 最小值下方加参数

```latex
    $$ \min\limits_{w \in W} $$
```

效果

$$ \min\limits_{w \in W} $$

## 矩阵

### 直接使用矩阵符号

```latex
    $$
    \begin{matrix} 0 & 1 \\ 1 & 0 \end{matrix}
    \quad
    \begin{pmatrix} 0 & -i \\ i & 0 \end{pmatrix}
    \quad
    \begin{bmatrix} 0 & -1 \\ 1 & 0 \end{bmatrix}
    \quad
    \begin{Bmatrix} 1 & 0 \\ 0 & -1 \end{Bmatrix}
    \quad
    \begin{vmatrix} a & b \\ c & d \end{vmatrix}
    \quad
    \begin{Vmatrix} i & 0 \\ 0 & -i \end{Vmatrix}
    $$
```

效果

$$
\begin{matrix} 0 & 1 \\\\ 1 & 0 \end{matrix}
\quad
\begin{pmatrix} 0 & -i \\\\ i & 0 \end{pmatrix}
\quad
\begin{bmatrix} 0 & -1 \\\\ 1 & 0 \end{bmatrix}
\quad
\begin{Bmatrix} 1 & 0 \\\\ 0 & -1 \end{Bmatrix}
\quad
\begin{vmatrix} a & b \\\\ c & d \end{vmatrix}
\quad
\begin{Vmatrix} i & 0 \\\\ 0 & -i \end{Vmatrix}
$$

### 使用`array`构建矩阵

```latex
    $$
    \left(                  %左括号
    \begin{array}{ccc}      %该矩阵一共3列，每一列都居中放置
        a11 & a12 & a13\\   %第一行元素
        a21 & a22 & a23\\   %第二行元素
    \end{array}
    \right)                 %右括号
    $$
```

效果

$$ \left(\begin{array}{ccc}
    a11 & a12 & a13 \\\\
    a21 & a22 & a23
\end{array}\right) $$