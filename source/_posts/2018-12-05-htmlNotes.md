---
title: HTML笔记
date: 2018-12-05 18:49:48
tags:
categories: [Program, Web]
---

# 一、语法相关

## 1. 上下标

```html
y = x<sup>2</sup>
H<sub>2</sub>O
```

效果

y = x<sup>2</sup>
H<sub>2</sub>O

## 2、表格

### 2.1. 表格和表头

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

### 2.2. 合并单元格

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

## 3. `<script>` 脚本标签

### 3.1. 使用import报错

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

## 4. 表单提交

### 4.1. 上传文件

- 一定要加上`enctype="multipart/form-data"`

```html
<form action="/uploadfile" method="POST" enctype="multipart/form-data">
    <input type="file" name="file"/><br>
    <input type="submit"/>
</form>
```

## 5. ajax提交

- 需要使用`jquery.min.js`库才能使用ajax

### 5.1. 提交json

```html
<!DOCTYPE html>
<html>

<head>
    <script src="./jquery.min.js"></script>
    <script type="text/javascript">
        function submitform() {
            let send_value;
            // 从id获取
            send_value = {
                firstName: document.getElementById("fname").value,
                lastName: document.getElementById("lname").value,
            };

            // 表单转化
            send_value = {};
            let form_data = $('#myForm').serializeArray();
            for (const value of form_data) {
                send_value[value.name] = value.value;
            }

            console.log(send_value);
            $.ajax({
                type: "POST",
                url: "/a/b/c",
                data: JSON.stringify(send_value),
                dataType: "json",
                contentType : "application/json",
                success: function(){
                    console.log("send success");
                },
                error: function (response) {
                    console.log("get error " + response.responseText);
                },
            });
        }
    </script>

</head>

<body>
    <form id="myForm">
        <p><label for="first_name">First Name:</label>
            <input type="text" name="first_name" id="fname" />
        </p>

        <p><label for="last_name">Last Name:</label>
            <input type="text" name="last_name" id="lname" />
        </p>

        <input value="Submit" type="button" onclick="submitform();" />
    </form>
</body>

</html>
```

### 5.2. 提交表单

```html
<!DOCTYPE html>
<html>

<head>
    <script src="./jquery.min.js"></script>
    <script type="text/javascript">
        function submitform() {
            $.ajax({
                type: "POST",
                url: "/a/b/c",
                data: $("#myForm").serialize(),
                success: function(){
                    console.log("send success");
                },
            });
        }
    </script>

</head>

<body>
    <form id="myForm">
        <p><label for="first_name">First Name:</label>
            <input type="text" name="first_name" id="fname" />
        </p>

        <p><label for="last_name">Last Name:</label>
            <input type="text" name="last_name" id="lname" />
        </p>

        <input value="Submit" type="button" onclick="submitform();" />
    </form>
</body>

</html>
```

### 5.3. 上传文件

```html
<!DOCTYPE html>
<html>

<head>
    <script src="./jquery.min.js"></script>
    <script type="text/javascript">
        function submitform() {
            $.ajax({
                type: "POST",
                url: "/uploadfile",
                data: new FormData($('#myForm')[0]),
                processData: false,     // 不预处理数据
                contentType: false,     // 必须，虽然不知道为什么
                success: function () {
                    console.log("send success");
                },
            });
        }
    </script>

</head>

<body>
    <form id="myForm">
        <input type="file" name="file" /><br>
        <input value="Submit" type="button" onclick="submitform();" />
    </form>
</body>

</html>
```

### 5.4. 结果实时更新

```html
<!DOCTYPE html>
<html>

<head>
    <script src="./jquery.min.js"></script>
    <script type="text/javascript">
        function submitform() {
            $.ajax({
                type: "POST",
                url: "/uploadfile",
                data: new FormData($('#myForm')[0]),
                processData: false,
                contentType: false,
                success: function (result) {
                    $('#output').text(result);  // 这里更新结果
                },
            });
        }
    </script>

</head>

<body>
    <form id="myForm">
        <input type="file" name="file" /><br>
        <input value="Submit" type="button" onclick="submitform();" />
    </form>
    <br>
    <span id="output"></span>
</body>

</html>
```

## 6. input 输入框

### 6.1. 剪贴板图片粘贴上传

```html
<!DOCTYPE html>
<html>

<head>
    <script src="./jquery.min.js"></script>
</head>

<body>
    <input type="text" name="file" id="fileInput" /><br>
    <script type="text/javascript">
        // 监听input内的粘贴事件
        document.getElementById("fileInput").addEventListener('paste', function (e) {
            // 判断剪贴板是否有内容
            if (!(e.clipboardData) && e.clipboardData.items) {
                return;
            }

            let data = new FormData();
            for (let i = 0, len = e.clipboardData.items.length; i < len; i++) {
                let item = e.clipboardData.items[i];
                console.log(item);
                // 仅处理png图片类型
                if (item.kind === "file" && item.type === 'image/png') {
                    // 添加到formData中
                    data.append("file", item.getAsFile());
                    break;
                }
            }
            if (!data.get("file")) {
                return;
            }
            // 上传
            $.ajax({
                type: "POST",
                url: "/uploadfile",
                data,
                processData: false,
                contentType: false,
                success: function (result) {
                    console.log(result);
                },
            });
        });
    </script>
</body>

</html>
```

### 6.2. 禁止回车自动提交

```html
<input type="text" name="moduleName" onkeydown="if (event.keyCode == 13) { return false; }" />
```

## 7. span标签

### 7.1. 显示换行

- 设置`white-space: pre;`就可以把换行符显示出来

```html
<span style="white-space: pre;"></span>
```

## 8. 注释

```html
<!--我是行注释-->
<h1>我是代码</h1>
<!--我是段注释
aaaasdf
-->
<h1>我还是代码<h1>
```

## 9. 键盘事件

- 可以用下面代码把所有键盘事件输出对应的keyCode

```html
<script type="text/javascript">
    window.addEventListener("keydown", (event) => {
        if (event.keyCode != undefined) {
        	alert(event.keyCode);
            return;
        }
    }, true);
</script>
```

## 10. 调整样式

- 在内部使用`style=""`

```html
<input type="text" style="width: 50px;" id="queryInterval" />
```

# 二、工程相关

## 1. html模板

```html
<!DOCTYPE html>
<html>
    <head>

    </head>
    <body>
        hello world
    </body>
</html>
```

# 小技巧和踩坑记

## 1. 浏览器不拉取最新代码

`Ctrl + F5`强制刷新，适用于chrome、edge

## 2. 访问https站点，其中的http请求被转成https

- 这个是chrome的安全设置，不允许在https页面访问http的请求，会自动提升为https
- 在火狐上没有这个限制
