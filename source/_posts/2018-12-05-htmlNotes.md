---
title: HTML笔记
date: 2018-12-05 18:49:48
tags:
categories: [Program, Web]
---

# 一、上下标

```html
    y = x<sup>2</sup>
    H<sub>2</sub>O
```

效果

y = x<sup>2</sup>
H<sub>2</sub>O

# 二、表格

## 表格和表头

```html
    <table>
        <tr>
            <th>Heading</th>
            <th>Another Heading</th>
        </tr>
        <tr>
            <td>row 1, cell 1</td>
            <td>row 1, cell 2</td>
        </tr>
        <tr>
            <td>row 2, cell 1</td>
            <td>row 2, cell 2</td>
        </tr>
    </table>
```

效果

<table><tr><th>Heading</th><th>Another Heading</th></tr><tr><td>row 1, cell 1</td><td>row 1, cell 2</td></tr><tr><td>row 2, cell 1</td><td>row 2, cell 2</td></tr></table>

## 合并单元格

```html
    <h4>横跨两列的单元格：</h4>
    <table>
        <tr>
            <th>姓名</th>
            <th colspan="2">电话</th>
        </tr>
        <tr>
            <td>Bill Gates</td>
            <td>555 77 854</td>
            <td>555 77 855</td>
        </tr>
    </table>

    <h4>横跨两行的单元格：</h4>
    <table>
        <tr>
            <th>姓名</th>
            <td>Bill Gates</td>
        </tr>
        <tr>
            <th rowspan="2">电话</th>
            <td>555 77 854</td>
        </tr>
        <tr>
            <td>555 77 855</td>
        </tr>
    </table>
```

效果

<h4>横跨两列的单元格：</h4>
<table><tr><th>姓名</th><th colspan="2">电话</th></tr><tr><td>Bill Gates</td><td>555 77 854</td><td>555 77 855</td></tr></table>

<h4>横跨两行的单元格：</h4>
<table><tr><th>姓名</th><td>Bill Gates</td></tr><tr><th rowspan="2">电话</th><td>555 77 854</td></tr><tr><td>555 77 855</td></tr></table>

# 三、`<script>` 脚本标签

## 1. 使用import报错

```html
<canvas id="drawing" width="500" height="200"></canvas>
<script>
    import { drawArrayRect } from "/lib/diy_canvas/drawArrayRect.js";
    ...
</script>
```

```
Uncaught SyntaxError: Cannot use import statement outside a module
```

加上`type="module"`就好了


```html
<canvas id="drawing" width="500" height="200"></canvas>
<script type="module">
    import { drawArrayRect } from "/lib/diy_canvas/drawArrayRect.js";
    ...
</script>
```

# 小技巧和踩坑记

## 1. 浏览器不拉取最新代码

`Ctrl + F5`强制刷新，适用于chrome、edge
