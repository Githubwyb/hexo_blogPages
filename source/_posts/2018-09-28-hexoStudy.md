---
title: hexo搭建中的学习笔记
date: 2018-09-28 17:06:17
tags:
categories: [Program, Document]
---

# 安装

## ubuntu

### 安装nodejs和npm

```shell
    curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh
    sudo bash nodesource_setup.sh
    sudo apt-get install nodejs
```

### 安装hexo和hexo-cli

```shell
    sudo npm install hexo -g
    sudo npm install hexo-cli -g
```

### 迁移博客代码到另一台电脑

- 先安装`hexo`和`hexo-cli`的环境
- 然后执行

```shell
    # 安装所有组件忽略脚本
    sudo npm install --ignore-scripts
    sudo npm audit fix
```

# 插件和操作

## 博客置顶

### 单纯置顶

```shell
    npm uninstall hexo-generator-index --save
    npm install hexo-generator-index-pin-top --save
```

- 然后在需要置顶的文章的Front-matter中加上top属性即可，根据top属性的大小排序，越大越靠前。比如下面这篇文章：

```
    ---
    title: hexo+GitHub博客搭建实战
    date: 2017-09-08 12:00:25
    categories: 博客搭建系列
    top: 5
    ---
```

## pdf插件

### 安装

- 在博客bash中执行下列命令安装

```shell
    npm install --save hexo-pdf
```

### 展示

- 使用外部文章网页链接

```php
    {% pdf http://7xov2f.com1.z0.glb.clouddn.com/bash_freshman.pdf %}
```

- 本地

需要创建一个同名的文件夹，放我们要上传的PDF文章

```php
    {% pdf  test.pdf %}
```

## 字数统计和阅读时长(网站底部/文章内)

### 安装插件

```shell
    npm install hexo-symbols-count-time --save
```

### 修改 站点配置文件

```yml
    symbols_count_time:
     #文章内是否显示
     symbols: true
     time: true
     # 网页底部是否显示
     total_symbols: true
     total_time: true
```

### 修改 主题配置文件

```yml
    # Post wordcount display settings
    # Dependencies: https://github.com/theme-next/hexo-symbols-count-time
    symbols_count_time:
     separated_meta: true
     #文章中的显示是否显示文字（本文字数|阅读时长）
     item_text_post: true
     #网页底部的显示是否显示文字（站点总字数|站点阅读时长）
     item_text_total: false
     # Average Word Length (chars count in word)
     awl: 4
     # Words Per Minute
     wpm: 275
```

## latex公式支持

### Next主题中自带latex插件

修改`_config.xml`文件即可

#### <span id = "latex">注意事项</span>

##### 1. 表格中不能使用`\|`，需要使用latex中的`\mid`来代替

示例

```latex
    | 描述       | 代码              |
    | ---------- | ----------------- |
    | 使用`\|`   | a            \| b |
    | 使用`\mid` | a $ \mid $ b      |
```

效果

| 描述       | 代码              |
| ---------- | ----------------- |
| 使用`\|`   | a            \| b |
| 使用`\mid` | a $ \mid $ b      |

##### 2. latex公式在hexo中不能写注释，并且换行使用`\\`无效，需要使用`\\\\`，大括号需要使用`\\{`才可使用

##### 3. latex公式在hexo中，出现没有解析成公式的情况，仔细对照代码和显示的代码，找到少的那个特殊字符（如`_`），给它加上`\`

# 踩坑记

## [latex注意事项](#latex)

## 表格相关

表格使用markdown无法合并单元格，如果使用html标签来写，需要写到一行中，否则会出现大段空行

```markdown
    # 没有空行
    <table><tr><th>姓名</th><td>Bill Gates</td></tr><tr><th rowspan="2">电话</th><td>555 77 854</td></tr><tr><td>555 77 855</td></tr></table>

    # 大段空行
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

<table><tr><th>姓名</th><td>Bill Gates</td></tr><tr><th rowspan="2">电话</th><td>555 77 854</td></tr><tr><td>555 77 855</td></tr></table>

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

## 自建域名在博客部署后解析失败

### 原因

- 博客部署到github上时，如果使用自建域名需要添加`CNAME`文件到根目录下
- hexo在部署时会把远程仓库整个覆盖掉，导致`CNAME`文件缺失而解析失败

### 解决方法

- 在`\hexo\source`目录下放入`CNAME`文件即可
- 所有source目录下的除了规定格式的会解析，其余文件都会原封不动放入仓库
