---
title: 软件使用技巧记录
date: 2019-09-12 10:02:35
tags:
categories: [Software Usage]
---

# source insight

## 只添加特定后缀文件

在`Add and remove project files`时，选中要添加的文件夹，在框内输入`*.xx`然后回车，再点击`Add Tree`即可。

## 工程文件目录显示裁剪前面绝对路径

在`Project Setting`里面配置，`File Paths`->`Project Source Directory`下配置源文件路径。配好后就可以看到工程文件目录下文件的路径前面裁剪掉了配置的路径，只留下配置路径的子目录。

# chrome

## 下载离线包

针对在内网环境下需要更新chrome需要下载离线包，官网下载为在线安装包，下载离线包的网址为，也就是在原本的网址上添加`?standalone=1`

[https://www.google.cn/intl/zh-CN/chrome/?standalone=1](https://www.google.cn/intl/zh-CN/chrome/?standalone=1)

# <span id = "tmux">tmux</span>

## 快捷键

- 工具特定命令前缀为`Ctrl + b`
- `<方向键>`: 切换到相应窗口
- `shift + "`: 纵向分屏
- `shift + %`: 横向分屏
- `Ctrl + <方向键>`: 朝相应方向移动边界

# <span id = "ctags">ctags</span>

## C/C++

### 基本选项

- `-I xxx`: 许多函数定义最后有一个`__THROW`类似的，ctags将解析出错，加上此选项会忽略xxx
- `--fields=+iaS`:
- `--extra=+q`
- `--c-kinds=+p`

### 添加系统头文件支持

- 执行以下命令生成系统头文件的tags
- 添加`set tags+=~/.vim/systags`包含系统头文件tags

系统头文件检索列表，里面尖括号字段根据系统适配
```
/usr/include
/usr/include/linux
/usr/include/net
/usr/include/netinet
/usr/include/arpa
/usr/include/c++/<version>
/usr/include/<sys-version>-linux-gnu/sys
```

```shell
# 把上面列表文件都扫出来，加到files.list里面
find xxx -maxdepth 1 -type f >> files.list
# ctags根据files.list生成tags文件
ctags -I __wur -I __THROW -I __THROWNL -I __attribute_pure__ -I __nonnull -I __attribute__ --file-scope=yes --language-force=C++ --links=yes --c-kinds=+p --c++-kinds=+p --fields=+iaS --extra=+q -f ~/.vim/systags -L files.list
```

## php支持

转载自 [https://www.cnblogs.com/longdouhzt/archive/2013/04/15/3022908.html](https://www.cnblogs.com/longdouhzt/archive/2013/04/15/3022908.html)

1. 添加以下命令到`~/.bashrc`或者`~/.zshrc`等当前使用的bash的起始配置文件中，可以使用phptags来生成php的tags文件

```shell
    alias phptags='ctags --langmap=php:.engine.inc.module.theme.php  --php-kinds=cdf  --languages=php'
```

2. 添加以下配置到`~/.ctags`中，添加一些匹配规则

```vim
    --regex-php=/^[ \t]*[(private|public|static)( \t)]*function[ \t]+([A-Za-z0-9_]+)[ \t]*\(/\1/f, function, functions/
    --regex-php=/^[ \t]*[(private|public|static)]+[ \t]+\$([A-Za-z0-9_]+)[ \t]*/\1/p, property, properties/
    --regex-php=/^[ \t]*(const)[ \t]+([A-Za-z0-9_]+)[ \t]*/\2/d, const, constants/
```

3. 到工程目录下`phptags -R`即可生成相应tags

# fzf

## 配置

1. 安装完成后，将下面配置加到`.zshrc`或者`.bashrc`中

```shell
# 40%屏显示非全屏，展示反向（输入框在上面，文件在下面正序），文件在右侧展示前50行
export FZF_DEFAULT_OPTS="--height 40% --layout=reverse --preview 'head -n 50 {} 2>/dev/null'"
```
