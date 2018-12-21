---
title: 数字图像处理课设
date: 2018-03-30 11:17:54
tags: [课程, AI]
categories: [notes, course]
---

## first

- "SIFT" 图像特征算法
- "Spatial Pyramid Matching" 空间金字塔匹配
- 梯度直方图 检测行人 行人是竖着的、好检测
- "Deformable Part Model" 可变形零件模型
- "Faster RCNN"
- "PASCAL Visual Object Challenge" 图像比赛
- 李飞飞教授 IMAGENET
- "Large Scale Visual Recognition Challenge" 图像比赛

| 2010     | 2012        | 2014               |
| -------- | ----------- | ------------------ |
| NEC-UIUC | SuperVision | GoogleNet & VGG & MSRA |

- 李飞飞教授 提问题让其他人解决 做的差

## next

### Image Classification

- 生活中分类大概1w类左右
- the problem: semantic gap 语义鸿沟
- challenge:
    - illumination 光照问题
    - deformation 形变
    - occlusion 闭塞
    - background clutter 背景嘈杂
    - intraclass variation 类别差异大
- data-driven approach
    - collect a dataset of images and labels
    - Use Machine Lerning to train an image classifier
    - Evaluate the classifier

#### First classifier: Nearest Neighbor Classifier

L1(Manhattan)distance

$$ d_1(I_1, I_2) = \sum\limits_{p}|I_1^P - I_2^P| $$

L2(Euclidean)distance

$$ d_1(I_1, I_2) = \sqrt{\sum\limits_{P}(I_1^P - I_2^P)^2}$$

- k-Nearest Neighbor

#### Second classifier: linear classification

$$ f(x, W, b) = W_{x_i} + b $$


