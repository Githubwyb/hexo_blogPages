---
title: python学习笔记
date: 2018-04-03 14:13:45
tags: [python, 笔记]
categories: notes_personal
---

# numpy

## numpy数据存取

numpy有一个可以直接将numpy数组矩阵按照原来的格式储存和读取的函数

    # 储存data
    np.save("file directory", data)

    # 读取data
    data = np.load("file directory")

这个方法可以直接将shape也存下来，不需要考虑格式、类型等，缺点是文件会有点大。

