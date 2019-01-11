---
title: 在线机器学习作业解答
date: 2019-01-11 09:27:18
tags: [homework, AI]
categories: [homework]
---

# Homework 1

## 1. Answer：

## 2. Answer：

$ \because f\ is\ a\ convex\ function $
$ \therefore f(\alpha x + (1 - \alpha)x') \leq \alpha f(x) + (1 - \alpha) f(x') $
$ \because g\ is\ a\ monotonically\ non-decreasing\ function,$
$ \therefore $ 
$$ g[f(\alpha x + (1 - \alpha)x')] \leq g[\alpha f(x) + (1 - \alpha) f(x')] \tag{1} $$
$ \because g\ is\ a\ convex\ function $
$ \therefore $
$$ g[\alpha f(x) + (1 - \alpha)f(x')] \leq \alpha g[f(x)] + (1 - \alpha) g[f(x')] \tag{2} $$
$ (1) \& (2) \Rightarrow $
$$ \begin{array}{rl}
    g[f(\alpha x + (1 - \alpha)x')] & \leq \alpha g[f(x)] + (1 - \alpha) g[f(x')] \\\\
    g \circ f(\alpha x + (1 - \alpha)x') & \leq \alpha g \circ f(x) + (1 - \alpha) g \circ f(x') \\\\
\end{array} $$
$ \therefore g \circ f\ is\ convex $

## 3. Answer:

### a. 

$ t = 1, \phi_1 = \phi_0 $
$ t = 2, \phi_2 = \sum_{i = 1}^{n} \frac{\phi_1 w_{2, i} c_{2, i}}{c_{1, i}} = \phi_1 \sum_{i = 1}^{n} \frac{w_{2, i} c_{2, i}}{c_{1, i}} $
...
$ t = t, \phi_t = \phi_0 \prod_{j = 1}^{t - 1}\sum_{i = 1}^{n} \frac{w_{j + 1, i} c_{j + 1, i}}{c_{j, i}} $

### b.

$ t = t, \phi_t = \phi_0 \prod_{j = 1}^{t - 1}\sum_{i = 1}^{n} \frac{w_i c_{j + 1, i}}{c_{j, i}} $