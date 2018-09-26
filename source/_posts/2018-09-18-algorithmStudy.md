---
title: 算法与数据结构学习
date: 2018-09-18 15:07:21
tags: [study, notes, algorithm]
categories: [notes, study]
---

# 数学知识复习

## 对数

### 定理 1.1
$$ log_A{B} = \frac{log_C{B}}{log_C{A}} ; C > 0 $$

## 级数

### 几何级数公式

$$ \sum_{i = 0}^{N}2^i = 2^{N+1} - 1 $$

$$ \sum_{i = 0}^{N}A^i = \frac{A^{N+1} - 1}{A - 1} $$

当N趋于$\infty$时，该和趋于$\frac{1}{1-A}$。

# 算法分析

## 数学基础

### 四个定义

- 如果存在正常数 $c$ 和 $n_0$ 使得当 $N \geq n_0$ 时 $T(N) \leq cf(N)$ ，则即为 $T(N) = O(f(N))$ 。
- 如果存在正常数 $c$ 和 $n_0$ 使得当 $N \geq n_0$ 时 $T(N) \geq cg(N)$ ，则即为 $T(N) = \Omega(f(N))$ 。
- $T(N) = \Theta (h(N))$ 当且仅当 $T(N) = O(h(N))$ 且 $T(N) = \Omega(h(N))$ 。
- 如果 $T(N) = O(p(N))$ 且 $T(N) \neq \Theta(p(N))$ ，则 $T(N) = o(p(N))$

# 排序算法

## 时间复杂度总结

<img src = "2018_09_26_01.jpg" width = "80%" />

## 插入排序

### 原理

- 自我理解像扑克牌起牌一样，一张一张插入到已有的序列中

<img src = "2018_09_26_02.gif" width = "80%" />

```C++
    /*
     * @function 用插入排序数组
     * @param data 数组首地址
     * @param length 数组长度
     * @param order 排列顺序：顺序，true；倒序，false
     */
    template<typename T>
    void insertionSort(T *data, int length, bool order = true) {
        T tmp = 0;
        for (int i = 1; i < length; ++i) {
            tmp = data[i];
            int j = 0;
            for (j = i; j > 0 && ((data[j - 1] > tmp) ^ !order); --j) {
                data[j] = data[j - 1];
            }
            data[j] = tmp;
        }
    }
```

### 定理

- $N$个互异数的数组的平均逆序数是$N(N-1)/4$。
- 通过交换相邻元素进行排序的任何算法平均需要$\Omega(N^2)$时间。

## 希尔排序

- 带间隔的插入排序

<img src = "2018_09_26_03.gif" width = "80%" />

```C++
    /*
     * @function 用希尔增量的希尔排序数组
     * @param data 数组首地址
     * @param length 数组长度
     * @param order 排列顺序：顺序，true；倒序，false
     */
    template<typename T>
    void shellSort(T *data, int length, bool order = true) {
        T tmp = 0;
        for (int increment = length / 2; increment > 0; increment /= 2) {
            for (int i = increment; i < length; ++i) {
                tmp = data[i];
                int j = 0;
                for (j = i; j >= increment && ((data[j - increment] > tmp) ^ !order); j -= increment) {
                    data[j] = data[j - increment];
                }
                data[j] = tmp;
            }
        }
    }
```
