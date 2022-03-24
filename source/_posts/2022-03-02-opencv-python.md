---
title: opencv-python 图像处理
date: 2022-03-02 15:05:38
tags: [image]
categories: [Program, Python]
---

# 一、语法相关

## 1. 图像读取和展示

- 展示使用notebook

```python
import cv2
import matplotlib.pyplot as plt

image = cv2.imread('/path/to/image')
x, y, color = image.shape   # 高度 宽度 通道数量（3代表RGB）
print(x, y, color)
# 代表高（x）100，宽（y）200，RGB颜色的图像
plt.imshow(image)
```

<img src='2022-03-04-01.png' />

# 小技巧和踩坑记
