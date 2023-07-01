---
title: windows上的go开发
date: 2023-06-05 19:24:32
tags: [go, windows]
categories: [Program, Web]
---

```powershell
PS C:\Users\sangfor> go version
go version go1.20.3 windows/amd64
```

# 一、前言

# 二、好用的第三方库

## 1. wintun `golang.zx2c4.com/wireguard`

- 三层网卡的虚拟网卡驱动
- 使用wireguard的wintun库
- 到官网下载安装包: https://www.wintun.net/
- 解压后是dll库，将dll库放到go程序运行目录即可
- 需要管理员权限运行程序才能安装虚拟网卡

### 1.1. 示例代码

```go
package main

import (
	"flag"
	"fmt"
	"net/netip"
	"os"
	"os/exec"

	"golang.zx2c4.com/wireguard/tun"
	"golang.zx2c4.com/wireguard/windows/tunnel/winipcfg"
)

func main() {
	flag.Parse()

	if flag.NArg() < 1 {
		fmt.Println("please input ip with mask x.x.x.x/xx")
		os.Exit(1)
	}
	addrWithMask := flag.Arg(0)

    // 创建网卡
	dev, err := tun.CreateTUN("MyNIC", 0)
	if err != nil {
		panic(err)
	}
	defer dev.Close()

	devName, err := dev.Name()
	if err != nil {
		panic(err)
	}

	nativeTunDevice := dev.(*tun.NativeTun)

	link := winipcfg.LUID(nativeTunDevice.LUID())

	ip, err := netip.ParsePrefix(addrWithMask)
	if err != nil {
		panic(err)
	}

    // 设置ip到网卡上
	err = link.SetIPAddresses([]netip.Prefix{ip})
	if err != nil {
		panic(err)
	}

	cmd := exec.Command("netsh", "interface", "ip", "set", "interface", devName, "metric=1")
	fmt.Println(cmd)
	if err = cmd.Run(); err != nil {
		panic(err)
	}

	select {}
}
```
