---
title: ubuntu学习
date: 2018-07-27 20:58:22
tags: [study, notes, ubuntu]
categories: [notes, study]
---

# 解决依赖关系

## 不指名解决依赖关系
```shell
    sudo apt --fix-broken install
```

# 忽略某个包的更新

使用以下命令可以忽略某个包的更新

```shell
    sudo apt-mark hold (pack)
```

取消忽略

```shell
    sudo apt-mark unhold (pack)
```
