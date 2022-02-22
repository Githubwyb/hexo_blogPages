---
title: linux deploy搭建笔记
date: 2021-02-26 09:39:30
tags: [Linux]
categories: [Program, Shell]
---

# 一、前言

linux deploy是在android手机上使用chroot搭建的linux环境，可以在手机上跑linux系统。
现在手机更新换代快。不用的android手机里面可是有高性能cpu和gpu，不用起来太浪费了。
但是不想学java，会用linux，那就在android手机上安装linux系统吧。

# 二、安装配置

## 1. 准备工作

- 手机要root
- 网络要好
- 安装apk: `linux deploy`、`busybox`

## 2. <span id="source_url">镜像站列表</span>

- ubuntu: `http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports`
- arch linux: `http://mirrors.ustc.edu.cn/archlinuxarm`

## 3. 安装

### 3.1. busybox安装

- 要获取root权限
- 小米或其他手机可能将`/system`分区锁定了，需要先解锁
- 默认安装位置`/system/xbin`
- 解锁并有root权限安装会很顺利
- 安装完成记得留意cpu架构，后面有用

### 3.2. linux deploy配置

**全局配置**

左上角三道横线，选设置。这里面的配置项要改

1. 屏幕常亮勾选
    - 大部分手机可能锁屏会自动休眠降低功耗，会导致linux运行很慢
    - 看情况勾选
2. 锁定wifi和cpu唤醒勾选
3. PATH变量，设置为`/system/xbin`，设置完要点击更新环境将path应用
4. 其他看着配置就可以了

**针对linux的配置**

左上角三道横线，选配置文件。针对自己要装的系统新建配置文件，需要手动选中
可以用默认的，就怕你反悔，老的配置再配一次。选好后右下角配置按钮进行下面配置

1. 发行版和版本自己选择
2. 架构
    - android手机一般都是arm64架构的
    - arm64包含armhf
    - aarch64和arm64可以混用
    - armv8是arm64，armv7是32位
    - busybox里面会显示cpu架构，可以参考
3. 源最好用国内源，不然下载很慢，[上面](#source_url)我列了几个
4. 安装类型
    - 镜像是将所有撞到一个文件里面，需要预设大小，默认2G，一般linux用不到2G，但是自己的文件放进去就不够了
    - 目录是将linux安装到磁盘上
    - `/sdcard`好像没有执行权限，反正我是失败了
    - `/data`目录用的也是手机内置储存的空间，和sdcard同样大小，里面放的都是应用的内置数据，不给用户操作，这里很适合放
    - `/`根目录用的好像是rom，很小，也就2G左右，最好不要放到根目录
    - 我设置为`/data/ubuntu`
5. 用户名，自己看着设置，root自动就有，也可以不要其他用户只要root
    - 特权用户不要动，`aid_inet`是给用户访问网络权限的用户组，删掉会无法访问外网
    - 其他特权用户自己摸索，我也不清楚
6. 密码自己看着办
7. 本地化，选`zh_CN.UTF-8`，或者自己想要的
8. 初始化，配置linux启动时是否要启动服务或执行脚本
    - `run-parts`是将`/etc/rc.local/`下的所有脚本依次执行
    - `sysv`是根据启动级别执行相应的`/etc/rc(x).c/`下的脚本，一般软件类似nginx会注册一个到这里面
9. 挂载，将目录挂载到系统中
    - source和target都要设置
    - 挂载组是`aid_everybody`，如果想要访问需要自己将用户添加到这个组里面
10. ssh启用会帮你安装sshd，最好启用
11. 图形化看情况，反正性能没那么高

### 3.3. 安装

1. 右上角三个点，安装
    - 安装网络一定要好，不然失败重新来
2. 如果改了配置，点配置就好了
3. 启动后，电脑或手机找个ssh客户端连接即可

# 三、archlinux在手机的安装

## 1. 软件配置

### 1.1. mysql安装配置

- 需要安装mariadb包，这个是arch官方的mysql社区包
- 装好后需要执行`sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql`进行初始化
- 启动MySQL `sudo /usr/bin/mysqld_safe --datadir='/var/lib/mysql' &`
- 进行安全配置 `sudo mysql_secure_installation`

### 1.2. aur软件助手

- arm上使用yay无法直接通过pacman进行安装
- 使用下面命令进行安装

```shell
sudo pacman -S git
sudo git clone https://aur.archlinux.org/yay-git.git
cd yay-git
makepkg -si
```

**报错fakeroot错误**

参考这篇博客[解决chroot/proot/wsl容器安装archlinux不能使用fakeroot的问题](https://zsxwz.com/2021/02/08/%e8%a7%a3%e5%86%b3chroot-proot-wsl%e5%ae%b9%e5%99%a8%e5%ae%89%e8%a3%85archlinux%e4%b8%8d%e8%83%bd%e4%bd%bf%e7%94%a8fakeroot%e7%9a%84%e9%97%ae%e9%a2%98/)

# 四、软件使用

## 1. nginx

- 由于底层使用的是chroot实现，无法使用system命令
- 启动nginx不需要使用system命令



# 踩坑记

## 1. adb调试本机

- 安装的是linux系统，自然可以安装adb，但是adb监听的端口和android自带的adbd监听端口冲突了，所以需要修改端口
- 保证adb没有启动，然后执行下面命令修改端口，然后可以愉快的使用adb了

```shell
export ANDROID_ADB_SERVER_PORT=9999
```

- 手机开启adb网络调试
- 如果adb启动前开启网络调试，adb启动后可以直接看到设备
- 如果adb启动后开启网络调试，可以使用`adb connect 127.0.0.1:5555`连接本机

## 2. arm64架构安装`opencv_python`

### 2.1. 先装opencv，再装`opencv_python`

- arch下面配置编译安装`opencv_python`步骤太繁琐了，没配好
- 直接`sudo pacman -Sy opencv`然后`pip install opencv_python`完事
- apt包就是`sudo apt install libopencv-dev`然后`pip install opencv_python`完事

### 2.2. 编译安装

- 默认的pip安装`opencv_python`包会报错，无法识别架构之类的，默认的安装只支持`x86_64/amd64`
- 想要使用opencv，需要使用源码编译

```shell
# clone opencv代码
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
# 到opencv目录进行编译
mkdir opencv/build
cd opencv/build
# 用cmake进行配置，注意修改里面的每个路径的值到真实的路径
# cmake过程报的错自己解决
cmake -D BUILD_opencv_python3=YES -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/home/wangyubo/miniforge3/opencv4.5.2 -D OPENCV_EXTRA_MODULES=../../opencv_contrib/modules -D PYTHON3_LIBRARIES=/home/wangyubo/miniforge3/lib/libpython3.9.so -D PYTHON3_EXECUTABLE=/home/wangyubo/miniforge3/bin/python -D PYTHON3_NUMPY_INCLUDE_DIRS=/home/wangyubo/miniforge3/lib/python3.9/site-packages/numpy/core/include/ -D PYTHON3_PACKAGES_PATH=/home/wangyubo/miniforge3/lib/python3.9/site-packages ..
# 编译安装
make -j4
make install
```

## 3. armv8l和conda

- miniforge和archiforge都识别不出来armv8l，会认为是`x86_64`安装的python无法使用
- 暂时没有解决方案，armv8l先不要用conda吧
