---
title: 软件使用技巧记录
date: 2019-09-12 10:02:35
tags:
categories: [Software Usage]
---

# 一、source insight

## 只添加特定后缀文件

在`Add and remove project files`时，选中要添加的文件夹，在框内输入`*.xx`然后回车，再点击`Add Tree`即可。

## 工程文件目录显示裁剪前面绝对路径

在`Project Setting`里面配置，`File Paths`->`Project Source Directory`下配置源文件路径。配好后就可以看到工程文件目录下文件的路径前面裁剪掉了配置的路径，只留下配置路径的子目录。

# 二、chrome

## 1. 下载离线包

针对在内网环境下需要更新chrome需要下载离线包，官网下载为在线安装包，下载离线包的网址为，也就是在原本的网址上添加`?standalone=1`

[https://www.google.cn/intl/zh-CN/chrome/?standalone=1](https://www.google.cn/intl/zh-CN/chrome/?standalone=1)

## 2. windows抓https并解密

参考[这个帖子](https://www.cnblogs.com/aucy/p/9082429.html)

- 给系统环境变量加一个`SSLKEYLOGFILE`变量，路径随便设置，这个会把chrome解析到的私钥存放到这个位置
- 在wireshark里面配置中找到`protocol -> TLS/SSL`，将`(Pre)-Master-Secret log filename`设置到同样的文件
- 重启chrome就可以解密了
- 每次解密都要重启chrome才行

## 3. 好用的插件

### (1) github加速

- 加速github下载的插件

## 4. chrome处理json时preview中文乱码

- 需要服务端将响应头的`Content-Type`设置为`application/json; charset=utf-8`即可

# 三、vscode

[vscode使用技巧记录](/blogs/2022-02-15-vscode)

# 四、firefox

## 1. 修改user-agent

1. 访问`about:config`
2. 创建或修改`general.useragent.override`，值为想要的user-agent值

# 五、网页小技巧

## 1. github查看代码技巧

将`https://github.com/android/ndk-samples`替换成`https://github1s.com/android/ndk-samples`可以打开网页版vscode进行代码查看

# 六、putty

## 1. 修改配色方案

- 保存后执行就好，将arch-work改成自己的配置文件名字
- 下面是我的一个配色方案，感觉还不错

```bat
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour0 /t REG_SZ /d 58,244,213  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour1 /t REG_SZ /d 255,255,255  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour2 /t REG_SZ /d 0,0,0  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour3 /t REG_SZ /d 85,85,85  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour4 /t REG_SZ /d 0,0,0  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour5 /t REG_SZ /d 0,255,0  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour6 /t REG_SZ /d 68,68,68  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour7 /t REG_SZ /d 119,119,119  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour8 /t REG_SZ /d 255,0,84  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour9 /t REG_SZ /d 214,94,117  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour10 /t REG_SZ /d 177,214,48  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour11 /t REG_SZ /d 85,255,85  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour12 /t REG_SZ /d 240,230,140  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour13 /t REG_SZ /d 255,255,85  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour14 /t REG_SZ /d 182,224,229  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour15 /t REG_SZ /d 159,211,229  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour16 /t REG_SZ /d 255,222,173  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour17 /t REG_SZ /d 255,85,255  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour18 /t REG_SZ /d 103,190,227  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour19 /t REG_SZ /d 255,215,0  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour20 /t REG_SZ /d 237,237,237  /f
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour21 /t REG_SZ /d 255,255,255  /f
```

## 2. 使用id_rsa私钥登陆

- 需要使用自带的puttygen将id_rsa导入进去另存为putty识别的密钥，再次使用就可以了

# 七、sublime text

## 1. 查看文件在文件树中的位置

- 在打开的文件中右键选`Reveal in Side Bar`

# 八、vmware

## 1. windows安装linux虚拟机启动报`A fault has occurred causing a virtual CPU to enter the shutdown state`

- 看看是不是装了`vmare 16.0`，是的话更新到最新版vmware，将对应的虚拟机迁移到新版本即可，`16.0`存在bug

## 2. 合并分离的磁盘文件

- 查看对应虚拟机中磁盘文件路径，然后执行命令

```shell
vmware-vdiskmanager -r Windows\ 10-0-000004.vmdk -t 0 merged.vmdk
```

# 九、Visual Studio Community

## 1. 增加评估时间

参考 [https://github.com/beatcracker/VSCELicense](https://github.com/beatcracker/VSCELicense)

- 使用管理员启动powershell
- 把仓库clone下来，cd到目录

```bat
@REM 添加可执行策略
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
@REM 导入powershell命令
Import-Module -Name .\VSCELicense.psd1
@REM 查看当前过期时间
Get-VSCELicenseExpirationDate -Version 2015
@REM 基于当前时间增加评估时间，多次执行效果一样，不会多增加时间，而且最大31
Set-VSCELicenseExpirationDate -Version 2015 -AddDays 31
```

# 十、Remmina

## 1. 远程连接到windows中文输入法无法使用

- 在`Remmina->Preferences->RDP->Use client keyboard mapping`把前面的勾去掉即可

# 十一、shadowsocks

## 1. windows上配置shadowsocks需要git bash使用需要单独配置

```shell
export http_proxy=socks5://127.0.0.1:1080
export https_proxy=socks5://127.0.0.1:1080
```
