---
title: python学习笔记
date: 2018-09-16 13:17:51
tags: [study, notes, python]
categories: [notes, study]
top: true
---

# 调用可执行文件

## 获取输出结果

```python
    import os

    f = os.popen("(cmd) (param)")
    data = f.readlines()
    f.close()
```

## 获取返回值

```python
    import os

    r_v = os.system("(cmd) (param)")
    print r_v
```

