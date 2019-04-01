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
    sudo npm install
    sudo npm audit fix
```

# 插件和操作

## 博客置顶

### 单纯置顶

```shell
    npm uninstall hexo-generator-index --save
    npm install hexo-generator-index-pin-top --save
```

- 然后在需要置顶的文章的Front-matter中加上top: true即可。比如下面这篇文章：

```
    ---
    title: hexo+GitHub博客搭建实战
    date: 2017-09-08 12:00:25
    categories: 博客搭建系列
    top: true
    ---
```

### 自定义排序置顶

- 使用的是top属性，top值越高，排序越在前，不设置top值得博文按照时间顺序排序。
- 修改Hexo文件夹下的`node_modules/hexo-generator-index/lib/generator.js`，打开在最后添加如下javascript代码

```javascript
    posts.data = posts.data.sort(function(a, b) {
        if(a.top && b.top) { // 两篇文章top都有定义
            if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
            else return b.top - a.top; // 否则按照top值降序排
        }
        else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
            return -1;
        }
        else if(!a.top && b.top) {
            return 1;
        }
        else return b.date - a.date; // 都没定义按照文章日期降序排
    });
```

### 设置置顶标志

- 打开：`/blog/themes/next/layout/_macro`目录下的`post.swig`文件，定位到`<div class="post-meta">`标签下，插入如下代码:

```php
    {% if post.top %}
    <i class="fa fa-thumb-tack"></i>
    <font color=7D26CD>置顶</font>
    <span class="post-meta-divider">|</span>
    {% endif %}
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

# 注意事项

## 1. [latex注意事项](#latex)

## 2. 表格相关

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