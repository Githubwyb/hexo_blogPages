---
title: linux下shadowsocks代理配置（服务端+客户端）
date: 2021-03-05 16:25:51
tags: [Linux]
categories:
---

# 前言

本文梳理linux上使用shadowsocks的一些方法和坑

# 一、安装

- shadowsocks服务端和客户端都可以使用python3-pip进行安装，比较方便
- 最好用管理员权限装，可以所有用户使用，否则只会装到`${HOME}/.local/bin`下面

```shell
sudo pip3 install shadowsocks
```

# 二、服务端配置

- 编写`/etc/shadowsocks.json`文件，也可以自己定义

```json
{
    "server": "0.0.0.0",
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "port_password": {
        "1234": "xxxx"
    },
    "timeout": 600,
    "method": "aes-256-cfb"
}
```

- 配置都能看懂，就不详细说明了
- 用下面的命令使用

```shell
ssserver -c /etc/shadowsocks.json start
```

## 1. 开机启动服务

- 编写`/usr/lib/systemd/system/ssserver.service`
- 使用绝对路径防止没有加载PATH环境变量

```ini
[Unit]
Description=Shadowsocks server

[Install]
WantedBy=multi-user.target

[Service]
User=root
Group=root
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks.json start
Restart=on-failure
```

- 调用`sudo systemctl enable ssserver`来添加开机启动项
- 调用`sudo service ssserver start`来启动服务

# 三、客户端使用

客户端可以使用sslocal命令

```shell
=> sslocal -h
usage: sslocal [OPTION]...
A fast tunnel proxy that helps you bypass firewalls.

You can supply configurations via either config file or command line arguments.

Proxy options:
  -c CONFIG              path to config file
  -s SERVER_ADDR         server address
  -p SERVER_PORT         server port, default: 8388
  -b LOCAL_ADDR          local binding address, default: 127.0.0.1
  -l LOCAL_PORT          local port, default: 1080
  -k PASSWORD            password
  -m METHOD              encryption method, default: aes-256-cfb
  -t TIMEOUT             timeout in seconds, default: 300
  --fast-open            use TCP_FASTOPEN, requires Linux 3.7+

General options:
  -h, --help             show this help message and exit
  -d start/stop/restart  daemon mode
  --pid-file PID_FILE    pid file for daemon mode
  --log-file LOG_FILE    log file for daemon mode
  --user USER            username to run as
  -v, -vv                verbose mode
  -q, -qq                quiet mode, only show warnings/errors
  --version              show version information
```

## 1. 开机自启动

同样编写`/usr/lib/systemd/system/sslocal.service`

```ini
[Unit]
Description=Shadowsocks client

[Install]
WantedBy=multi-user.target

[Service]
User=root
Group=root
ExecStart=/usr/local/bin/sslocal -c /etc/shadowsocks.json
Restart=on-failure
```

- `sudo systemctl enable sslocal`添加开机启动项
- `sudo service sslocal start`启动服务

## 2. 代理设置

- 上面客户端启用仅仅是监听了1080端口做socks5代理，想要使用还需要做处理

### 2.1. proxychains做socks5代理

- 如果系统支持socks5代理，可以使用`export http_proxy="socks5://127.0.0.1:1080"`
- 不支持可以使用proxychains做命令行代理
- 还有部分命令，即使设置了proxy就是不走代理，那么使用proxychains治治他们的脾气

```shell
sudo apt install proxychains
```

**配置**

- 修改`/etc/proxychains.conf`

```ini
[ProxyList]
# 新增
socks5 127.0.0.1 1080
```

**使用**

```shell
proxychains -q git clone xxx
```

### 2.2. privoxy做http代理

- 很多情况使用socks5不能代理，需要使用http代理
- 可以安装privoxy来讲socks5转成http代理

```shell
sudo apt intall privoxy
```

**配置**

- 编辑`/etc/privoxy/config`，新增配置
- 想配置端口自己搜，默认8118

```ini
forward-socks5   /               127.0.0.1:1080 .
```

- 改完重启服务`sudo service privoxy restart`

**使用**

- 直接在需要修改代理的地方加上`http://127.0.0.1:8118`即可
- 命令行配置

```shell
export http_proxy="127.0.0.1:8118"
export https_proxy="127.0.0.1:8118"
```

