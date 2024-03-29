---
title: zsh配置记录
date: 2018-10-25 13:18:21
tags: [Linux]
categories: [Program, Shell]
---

# 一、安装zsh

## 1. apt包管理系列

直接执行命令

```shell
sudo apt install zsh
```

# 二、配置zsh

## 1. 默认使用zsh作为shell

```shell
chsh -s /bin/zsh
```

重启终端即可

## 2. 使用oh-my-zsh美化zsh

### 2.1. 安装oh-my-zsh

```shell
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

自动安装，会直接使用默认主题`robbyrussell`

### 2.2. 修改主题

执行以下命令

```shell
vim ~/.zshrc
```

找到

```shell
ZSH_THEME="robbyrussell"
```

改为自己想要的主题即可，推荐一个主题`ys`

### 2.3. 插件

#### (1) 代码高亮 `zsh-syntax-highlighting`

使用以下命令安装插件

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

在`.zshrc`下修改`plugins`添加插件即可

```shell
plugins=(
    zsh-syntax-highlighting
)
```

#### (2) 上一次执行代码提示 `zsh-autosuggestions`

使用以下命令安装插件

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

在`.zshrc`下修改`plugins`添加插件即可

```shell
plugins=(
    zsh-autosuggestions
)
```

# 常见问题

## 进入git目录卡慢的问题

因为zsh默认加载git各种信息，可以通过配置关闭

```shell
# 不检测文件改动
git config --global --add oh-my-zsh.hide-dirty 1
# 隐藏所有git信息
git config --global --add oh-my-zsh.hide-status 1
```

## 目录权限问题

复制或者更新zsh，重启可能会出现以下报错

```shell
[oh-my-zsh] Insecure completion-dependent directories detected:
drwxrwxrwx 1 ***** ***** 512 Oct 25 13:51 /home/*****/.oh-my-zsh/custom/plugins
drwxrwxrwx 1 ***** ***** 512 Oct 25 13:49 /home/*****/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting

[oh-my-zsh] For safety, we will not load completions from these directories until
[oh-my-zsh] you fix their permissions and ownership and restart zsh.
[oh-my-zsh] See the above list for directories with group or other writability.

[oh-my-zsh] To fix your permissions you can do so by disabling
[oh-my-zsh] the write permission of "group" and "others" and making sure that the
[oh-my-zsh] owner of these directories is either root or your current user.
[oh-my-zsh] The following command may help:
[oh-my-zsh]     compaudit | xargs chmod g-w,o-w

[oh-my-zsh] If the above didn't help or you want to skip the verification of
[oh-my-zsh] insecure directories you can set the variable ZSH_DISABLE_COMPFIX to
[oh-my-zsh] "true" before oh-my-zsh is sourced in your zshrc file.
```

添加一下权限即可

```shell
chmod 755 /home/*****/.oh-my-zsh/custom/plugins
chmod 755 /home/*****/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```
