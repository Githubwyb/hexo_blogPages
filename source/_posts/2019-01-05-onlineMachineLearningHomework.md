---
title: 在线机器学习作业
date: 2019-01-05 10:30:36
tags: [homework, AI]
categories: [homework]
---

# Homework 1

在在线优化设置中，玩家玩点 $ w_t \in W $，对手以非负函数 $ f_t $ 响应，并且玩家遭受 $ f_t(w_t) $ 的损失。 假设 $ W $ 是一个有界集合，并且每个 $ f_t $ 在 $ W $ 上有较低的界限。玩家在 $ T $ 轮之后的遗憾被定义为：
$$ \sum_{t = 1}^{T} f_t(w_t) - \min\limits_{w \in W} \sum_{t = 1}^{T} f_t(w) $$
遗憾的界限是函数 $ R(T) $ ，使得对于任何序列 $ f_1, ..., f_T $ 它保持：
$$ \forall T \sum_{t = 1}^{T} f_t(w_t) - \min\limits_{w \in W} \sum_{t = 1}^{T} f_t(w) \leq R(T) $$
1. 首先，证明我们可以在不失一般性的情况下假设每个 $ t $ 的 $ \min f_t(x) = 0 $ 。如果：
$$ f_t(w_t) = 0 \Rightarrow w_{t + 1} = w_t $$
在线优化算法是保守的。
换句话说，保守算法只要不遭受任何损失就会保持同一点。设 $ A $ 为具有 $ R(T) $ 的后悔界限的在线优化算法。使用 $ A $ 定义具有相同后悔限制的保守在线优化算法 $ A' $ .