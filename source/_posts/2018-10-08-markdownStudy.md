---
title: markdown学习笔记
date: 2018-10-08 14:37:39
tags:
categories: [Program, Document]
---

# 一、写文档

## 1. 空格

```markdown
&nbsp;
```

## 2. 删除线

```markdown
~~删除线~~
```

效果

~~删除线~~

## 3. 超链接

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

## 4. 文内跳转

代码

```markdown
<span id = "test">跳转到这里</span>

[跳转](#test)
```

效果

<span id = "test">跳转到这里</span>

[跳转](#test)

## 5. 代码块中有反引号

- 比要加的多就好了

`````markdown
- 行内 ``a`b``
- 代码块

````markdown
```shell
echo
```
````
`````

**效果**

- 行内 ``a`b``
- 代码块

````markdown
```shell
echo
```
````
