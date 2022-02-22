---
title: 前端的javascript笔记
date: 2022-02-22 15:15:32
tags:
categories: [Program, Web]
---

# 一、语法

## 1. import导入js文件

### 1.1. export和import

- 使用`export default`直接导出

```js
const testObj = {a: 1, b: 2};

function testFunc(a, b) {
    ...
}

export default {
    testObj,
    testFunc,
};
```

使用

```html
<script type="module">
    import data from '/path/to/xxx.js'
    data.testFunc(data.testObj.a, data.testObj.b)
</script>
```
