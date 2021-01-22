---
title: markdown学习笔记
date: 2018-10-08 14:37:39
tags: 
categories: [Program, Document]
---

# 写文档

## 空格

```markdown
    &nbsp;
```

## 删除线

```markdown
    ~~删除线~~
```

效果

~~删除线~~

## 超链接

### 链接写在文中

代码

```markdown
    [我的博客](https://githubwyb.github.io/)
```

效果

[我的博客](https://githubwyb.github.io/)

### 链接写在文后

代码

```markdown
    [我的博客][1]

    [1]: https://githubwyb.github.io/
```

效果

[我的博客][1]

[1]: https://githubwyb.github.io/

## 文内跳转

代码

```markdown
    <span id = "test">跳转到这里</span>

    [跳转](#test)
```

效果

<span id = "test">跳转到这里</span>

[跳转](#test)