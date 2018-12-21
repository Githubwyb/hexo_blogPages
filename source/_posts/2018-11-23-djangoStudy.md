---
title: Django框架学习
date: 2018-11-23 16:42:23
tags: [python, web]
categories: [notes, study]
---

# 安装

## 使用pip命令安装

```python
    pip install Django==2.1.3
```

## 官网下载安装

[官网下载地址][1]

[1]: https://www.djangoproject.com/download/

下载最新包后使用以下命令安装

```python
    python setup.py install
```

## 验证安装是否成功

```python
    import django
    django.get_version()
```