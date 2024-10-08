---
title: windows命令行记录
date: 2020-03-18 14:51:41
tags: [windows]
categories: [Program, Shell]
---

# 一、bat 脚本

## 1. 一些基本语法

### 1.1. 变量

```bat
:: 声明变量
set test_var=""
:: 变量赋值
set /p test_var="aaa"
:: 文件内容赋值给变量
set /p test_var=<D:\path\to\file.txt
```

### 其他

```bat
REM 这是注释
:: 这是注释

:: 打印环境变量
echo %xxx%

:::::::::: 目录操作 ::::::::::
:: 列举目录内容，类似ls
dir xxx/xxx

:::::::::: 文件操作 ::::::::::
:: 新建文件
type NUL > xxx.txt
:: 删除文件
del xxx.txt
```

## 2. 创建链接 mklink

- `/j`: 创建目录链接
- `/d`: 创建目录符号链接。默认为文件符号链接
- `/h`: 创建硬链接而非符号链接

```bat
mklink /j [link_name] [src_dir]
```

**硬链接、符号链接、目录链接、快捷方式区别**

|            | 硬链接                     | 符号链接  | 目录链接           | 快捷方式     |
| ---------- | -------------------------- | --------- | ------------------ | ------------ |
| 类型       | 链接                       | 链接      | 链接               | xxx.lnk 文件 |
| 作用对象   | 仅文件                     | 目录/文件 | 目录/文件          | 目录/文件    |
| 大小       | 和源文件一致（但不占空间） | 0         | 0                  | 几百字节     |
| 源文件删除 | 文件内容存在               | 失效      | 失效               | 失效         |
| 源文件替换 | 文件内容还是原始文件       | 新文件    | 新文件             | 新文件       |
| 局限       | 仅同一个分区               | -         | -                  | -            |
| 路径       | -                          | 相对路径  | 自动转化为绝对路径 | -            |

## 3. 拷贝文件 COPY/XCOPY

### 3.1. COPY

- 只能复制文件，不能复制文件夹

#### 选项

- `/B`: 合并文件
- `/Y`: 取消覆盖的确认

#### (1) 合并文件

-   合并两个二进制文件

```bat
COPY /B /Y [bin_file1] [bin_file2]
```

#### (2) 复制单个文件

```bat
:: 拷贝到目录下
COPY /Y C:\aaa.txt D:\
:: 拷贝到目录下换个名字
COPY /Y C:\aaa.txt D:\test.bat
```

#### (3) 批量复制文件

```bat
:: 目录下所有文件复制到另一个目录
COPY /Y C:\test\ D:\aaa\
:: 正则匹配
COPY /Y C:\test\*.txt D:\aaa\
```

### 3.2. XCOPY

#### (1) 选项

- `/S`: 复制目录和子目录，不包括空目录，不带此选项不复制子目录，需要和`/E`一起复制空目录
- `/T`: 只复制目录结构，不复制文件，不包括空目录，需要和`/E`一起复制空目录
- `/E`: 复制空目录
- `/Y`: 取消覆盖确认提示
- `/I`: 路径不存在的情况，原始路径是目录就自动创建目录，文件则还是会提示选择是否为文件
- `/K`: 保留只读属性
- `/V`: 验证文件完全相同

#### (2) 示例

- 拷贝文件到目录不能跟文件名，只需要父级目录就好，跟文件名会弹出确认是文件还是目录

```bat
xcopy /y /k /v C:\path\to\file.txt D:\path\to\
```

## 4. 注册表

### 4.1. 获取注册表的值

```bat
REG QUERY "HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\钉钉" /v "UninstallString"
```

### 4.2. 修改注册表的值

- 路径
- `/v`: key
- `/t`: type
- `/d`: value
- `/f`: 不显示确认输入

```bat
reg add HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\arch-work /v Colour0  /t REG_SZ /d 255,255,255   /f
```

## 5. for循环

### 5.1. 基本形态

- 在cmd中使用一个`%`: `for %i in (command1) do command2`
- 在batch脚本中使用两个`%%`: `for %%i in (command1) do command2`
- `%`后只能跟一个字母作为变量，不能跟多个字母

#### 几个示例

**(1) 数组遍历**

```bat
@echo off
for %%i in (A,B,C) do (
    echo s %%i e
)
pause
```

输出

```
s A e
s B e
s C e
```

### 5.2. for /r 递归遍历目录下所有文件，不包含目录

#### (1) 原型

```bat
FOR /R [[drive:]path] %%parameter IN (set) DO command
```

#### (2) 示例

**输出目录下包含子目录的所有文件**

```bat
@echo off
for /r D:\path\to\dir %%i in (*) do (
    echo s %%i e
)
pause
```

**输出目录下包含子目录的所有json文件**

```bat
@echo off
for /r D:\path\to\dir %%i in (*.json) do (
    echo s %%i e
)
pause
```

## 6. cd命令

### 6.1. 基本用法

```bat
:: 返回上一级
cd ..
:: 切换盘
cd D:
:: 切换到当前盘的相对目录
cd a\b\c
:: 切换到不同盘的绝对目录
cd /d C:\path\to\dir
```

## 7. 快捷键

### 7.1. 基本快捷键

- `F7`: 当次窗口执行过的历史记录，可以选择之前执行过的命令
- `Esc`: 清楚当前行

## 8. dir 列举当前目录

### 8.1. 选项

- `/B`: 仅显示名字，不显示其他信息，默认显示时间类型
- `/-C`: 大小不显示千位数分割符，默认`/C`显示
- `/Q`: 显示所有者，默认不显示
- `/S`: 显示子目录
- `/W`: 横向排列，目录两旁带`[]`
- `/D`: 纵向排列，目录两旁带`[]`
- `/A:[属性]`:
  - `d`: 目录
  - `r`: 只读文件
  - `s`: 系统文件
  - `h`: 隐藏文件

### 8.2. 示例

```bat
D:\path\to\dir>dir /d
 驱动器 D 中的卷是 新加卷
 卷的序列号是 641B-9BCA

 D:\path\to\dir 的目录

[.]                .cpplint-ignore    .gitmodules        [cmake]            [doc]              [prebuilt]         [script]           [tests]
[..]               .gitignore         [.vscode]          CMakeLists.txt     Doxyfile           README.md          [mobile]           [third-party]
.clang-format      [.gitlab]          [build]            cppcheck.xml       [LICENSE]          requirements.txt   [pc]               [tools]
.cppcheck-ignore   .gitlab-ci.yml     build.sh           CPPLINT.cfg        [package]          [res]              [src]              version
              14 个文件        341,406 字节
              18 个目录 42,100,445,184 可用字节

D:\path\to\dir>dir /w
 驱动器 D 中的卷是 新加卷
 卷的序列号是 641B-9BCA

 D:\path\to\dir 的目录

[.]                [..]               .clang-format      .cppcheck-ignore   .cpplint-ignore    .gitignore         [.gitlab]          .gitlab-ci.yml     .gitmodules        [.vscode]
[build]            build.sh           [cmake]            CMakeLists.txt     cppcheck.xml       CPPLINT.cfg        [doc]              Doxyfile           [LICENSE]          [package]
[prebuilt]         README.md          requirements.txt   [res]              [script]           [mobile]           [pc]               [src]              [tests]            [third-party]
[tools]            version
              14 个文件        341,406 字节
              18 个目录 42,100,436,992 可用字节


```

## 9. 字符串操作

### 9.1. 字符串替换

```bat
set str=aaaa bb cc
:: 所有b替换成t
echo %str:b=t%
:: 替换第一个b和之前的字符串
echo %str:*b=tt%
```

输出

```
aaaa tt cc
ttb cc
```

### 9.2. 字符串截取

**<font color="red">`,`后大于0为长度，小于0为索引</font>**

```bat
set abc=abcdefghijklmnopqrstuvwxyz

echo 原字符串为:%abc%
echo 截取前5个字符:%abc:~0,5%
echo 截取最后5个字符:%abc:~-5%
echo 截取第一个到倒数第6个字符:%abc:~1,-5%
echo 从第4个字符开始截取5个字符:%abc:~3,5%
echo 从倒数第14个字符开始截取5个字符:%abc:~-14,5%
```

输出

```
原字符串为:abcdefghijklmnopqrstuvwxyz
截取前5个字符:abcde
截取最后5个字符:vwxyz
截取第一个到倒数第6个字符:bcdefghijklmnopqrstu
从第4个字符开始截取5个字符:defgh
从倒数第14个字符开始截取5个字符:mnopq
```

## 10. 引号的作用

- 引号作为字符串插入，不作为字符串的标识

```bat
set str=aaa
set str1="aaa"

echo %str% 123
echo %str1% 123
```

输出

```
aaa 123
"aaa" 123
```

## 11. if判断

### 11.1. 基本形态

```bat
:: 基本状态是一行
if %a% == 123 (echo a==123) else if %a% == 1234 (echo a==123) else (echo %a% not valid)
:: 也可以分行写，但是处理时认为是一行
if %a% == 123 (
    echo a==123
) else if %a% == 1234 (
    echo a==123
) else (
    echo %a% not valid
)
```

## 12. move 移动文件

```bat
move [src] [dst]
```

## 13. tracert 跟踪路由节点

- 类似linux的traceroute

```bat
C:\Users\User>tracert 199.200.2.170

通过最多 30 个跃点跟踪到 199.200.2.170 的路由

  1    <1 毫秒   <1 毫秒   <1 毫秒 172.22.73.254
  2    <1 毫秒   <1 毫秒   <1 毫秒 10.10.110.132
  3     1 ms    <1 毫秒    3 ms  10.10.110.37
  4    <1 毫秒   <1 毫秒   <1 毫秒 199.200.2.170

跟踪完成。
```

## 14. netsh 网络相关命令工具

### 14.1. 通用命令

```bat
:: 重置网络配置
netsh winsock reset
```

### 14.2. ipv6

- ipv6使用需要使用`netsh interface ipv6`命令

#### 1) 路由相关

```bat
:::::::::: 查看路由 ::::::::::
C:\Users\User>netsh interface ipv6 show route

发布    类型     跃点数 前缀                     索引 网关/接口名称
------- -------- ---    ------------------------ --- ------------------------
否        系统        256  ::1/128                     1  Loopback Pseudo-Interface 1
否        系统        256  fe80::/64                  21  以太网 2
否        系统        256  fe80::/64                  27  vEthernet (Default Switch)
否        系统        256  fe80::5dee:8e08:f0bd:1fad/128   21  以太网 2
否        系统        256  fe80::6811:502a:c8dd:b09e/128   27  vEthernet (Default Switch)
否        系统        256  ff00::/8                    1  Loopback Pseudo-Interface 1
否        系统        256  ff00::/8                   21  以太网 2
否        系统        256  ff00::/8                   27  vEthernet (Default Switch)

:::::::::: 添加路由 ::::::::::
C:\Users\User>netsh interface ipv6 add route
一个或多个重要的参数没有输入。
请验证需要的参数，然后再次输入。
此命令提供的语法不正确。请查看帮助以获取正确的语法信息。

用法: add route [prefix=]<IPv6 address>/<integer> [interface=]<string>
             [[nexthop=]<IPv6 address>] [[siteprefixlength=]<integer>]
             [[metric=]<integer>] [[publish=]no|age|yes]
             [[validlifetime=]<integer>|infinite]
             [[preferredlifetime=]<integer>|infinite]
             [[store=]active|persistent]

参数:

       标记                值
       prefix            - 要为其添加路由的前缀。
       interface         - 接口名称或索引。
       nexthop           - 网关地址(如果前缀不在链路上)。
       siteprefixlength  - 整个站点的前缀长度(如果在链路上)。
       metric            - 路由跃点数。
       publish           - 下列其中一个值:
                           no: 未在路由播发中播发。
                               此为默认值。
                           age: 用有限生存时间在路由播发中播发。
                           yes: 用无限生存时间在路由播发中播发。
       validlifetime     - 路由有效的生存时间。
                           默认值是无限。
       preferredlifetime - 首选路由生存时间。
                           默认值为无限生存时间。
       store             - 下列其中一个值:
                           active: 更改仅持续到下一次启动。
                           persistent: 更改持久有效。此为默认值。

说明: 为给定前缀添加路由。

示例:

       add route 3ffe::/16 "Internet" fe80::1

:::::::::: 删除路由就是将add给成del ::::::::::
C:\Users\User>netsh interface ipv6 del route 3ffe::/16 "Internet" fe80::1
```

### 14.3. ipv4

#### 1) 端口转发

```bat
:: 将1234端口的tcp请求转发到445 ::
netsh interface portproxy add v4tov4 listenport=1234 listenaddress=0.0.0.0 connectport=445 connectaddress=127.0.0.1 protocol=tcp
```

#### 2) dns配置

```bat
:: 显示dns配置
C:\Users\User>netsh interface ip show dnsservers

接口 "以太网 2" 的配置
    通过 DHCP 配置的 DNS 服务器:      223.5.5.5
                                          114.114.114.114
    用哪个前缀注册:                   只是主要

接口 "Loopback Pseudo-Interface 1" 的配置
    静态配置的 DNS 服务器:            无
    用哪个前缀注册:                   只是主要

:: 配置为网卡为静态dns
C:\Users\User>netsh interface ip set dns "以太网 2" static 233.5.5.5

:: 添加新的静态dns
C:\Users\User>netsh interface ip add dns "以太网 2" 114.114.114.114

:: 恢复dns为dhcp获取
C:\Users\User>netsh interface ip set dns "以太网 2" source=dhcp
```

#### 3) 跃点数（优先级）

```bat
:: 查看跃点数
C:\Users\User>netsh interface ip show Interface

Idx     Met         MTU          状态                名称
---  ----------  ----------  ------------  ---------------------------
  1          75  4294967295  connected     Loopback Pseudo-Interface 1
 17           1        1400  connected     本地连接
 15          25        1500  connected     以太网 5

:: 设置某个网卡跃点数为特定值
C:\Users\User>netsh interface ip set interface "本地连接" metric=100
:: 也可以用idx
C:\Users\User>netsh interface ip set interface 17 metric=100
:: 设置为自动跃点
C:\Users\User>netsh interface ip set interface 17 metric=auto
```

#### 4) ip地址配置


### 14.4. 防火墙

#### 1) `advfirewall firewall`

```bat
:: 开放1234的tcp入站端口
netsh advfirewall firewall add rule name=shared_folder dir=in protocol=TCP localport=1234 action=allow
```

## 15. taskkill

- `/im [xxx.exe]`: 删除某exe的进程
- `/f`: 强制删除

```bat
C:\Users\User>taskkill /f /im appidcertstorecheck.exe
成功: 已终止进程 "appidcertstorecheck.exe"，其 PID 为 11804。
成功: 已终止进程 "appidcertstorecheck.exe"，其 PID 为 15924。
成功: 已终止进程 "appidcertstorecheck.exe"，其 PID 为 17120。
成功: 已终止进程 "appidcertstorecheck.exe"，其 PID 为 16996。
成功: 已终止进程 "appidcertstorecheck.exe"，其 PID 为 17436。
成功: 已终止进程 "appidcertstorecheck.exe"，其 PID 为 9724。
```

## 16. cmd 调用其他bat脚本

- `/c`: 执行后面的命令后结束

```bat
cmd /c D:\path\to\test.bat
```

## 17. tasklist显示进程列表

- `/fi`: 过滤器，具体语法查看`/?`
- `/v`: 显示详细信息

```bat
C:\Users\sangfor>tasklist /FI "imagename eq abcdefAgent.exe"

映像名称                       PID 会话名              会话#       内存使用
========================= ======== ================ =========== ============
abcdefAgent.exe               6172 Services                   0     20,600 K
abcdefAgent.exe               7532 RDP-Tcp#1                  1     59,368 K
abcdefAgent.exe               1424 RDP-Tcp#1                  1     37,668 K
```

## 18. net 启动停止服务

### 18.1. 启动停止服务

```bat
C:\Windows\system32>net start sshd
OpenSSH SSH Server 服务正在启动 .
OpenSSH SSH Server 服务已经启动成功。


C:\Windows\system32>net stop sshd

OpenSSH SSH Server 服务已成功停止。


```

## 19. wmic windows管理工具

### 19.1. process 查看进程信息

- `commandline`: 命令行参数
- `ProcessId`: 进程号
- `Name`: 名称，一般是`xxx.exe`
- `caption`: 好像和name一样

```bat
C:\Windows\system32>wmic process where caption="cmd.exe" get caption,commandline /value


Caption=cmd.exe
CommandLine="C:\Windows\system32\cmd.exe"


```

## 20. find/findstr

- `/i`: 不区分大小写
- `/v`: 不包含

```bat
C:\Windows\system32>wmic process where processid=4384 get commandline /value | find /i "atrustcore"
CommandLine=aTrustAgent --plugin plugins/aTrustCore --enable-http --enable-event-center

```

## 21. %ERRORLEVEL% 上一条命令的返回值

```bat
C:\Windows\system32>wmic process where processid=4384 get commandline /value | find /i "atrustcore"
CommandLine=aTrustAgent --plugin plugins/aTrustCore --enable-http --enable-event-center

C:\Windows\system32>echo %ERRORLEVEL%
0

C:\Windows\system32>wmic process where processid=4384 get commandline /value | find /i "atrustcore1
FIND: 参数格式不正确

C:\Windows\system32>echo %ERRORLEVEL%
2

```

## 22. route 添加路由

- `-p`: 永久生效，默认在重启后会失效
- `-6`: ipv6路由，默认ipv4

```bat
route add 1.1.1.1 mask 255.255.0.0 10.242.255.254 -p
```

## 23. shutdown 关机命令

```bat
:: 进入休眠状态
shutdown /h
:: 关机
shutdown /s /t 0
:: 重启
shutdown /r
```

## 24. powercfg 电源配置

```bat
:: 查看系统电源配置
C:\WINDOWS\system32>powercfg -a
此系统上没有以下睡眠状态:
    待机 (S1)
        虚拟机监控程序不支持此待机状态。

    待机 (S2)
        系统固件不支持此待机状态。
        虚拟机监控程序不支持此待机状态。

    待机 (S3)
        系统固件不支持此待机状态。
        虚拟机监控程序不支持此待机状态。

    休眠
        系统固件不支持休眠。
        该虚拟机监控程序不支持休眠。

    待机(S0 低电量待机)
        系统固件不支持此待机状态。

    混合睡眠
        待机(S3)不可用。
        休眠不可用。
        虚拟机监控程序不支持此待机状态。

    快速启动
        休眠不可用。

:: 开启/禁止系统休眠，只是配置开启，不是立马进入
C:\Windows\system32>powercfg -h on
C:\Windows\system32>powercfg -h off
```

## 25. ipconfig 网络配置

### 25.1. 刷新本地dns缓存

- 所有dns请求重新解析

```bat
C:\Windows\system32>ipconfig /flushdns

Windows IP 配置

已成功刷新 DNS 解析缓存。

```

## 26. 弹框获取管理员权限

```bat
set current_dir=%~dp0
echo %current_dir%

@echo off&color 17
if exist "%SystemRoot%\SysWOW64" path %path%;%windir%\SysNative;%SystemRoot%\SysWOW64;%~dp0
bcdedit >nul
if '%errorlevel%' NEQ '0' (goto UACPrompt) else (goto UACAdmin)
:UACPrompt
%1 start "" mshta vbscript:createobject("shell.application").shellexecute("""%~0""","::",,"runas",1)(window.close)&exit
exit /B
:UACAdmin

cd /d %current_dir%
```

## 27. windows特殊目录

- `%allUsersprofile%`: 打开所有用户的配置文件 `C:\ProgramData`
- `%appdata%`: 打开AppData文件夹 `C:\Users\{username}\AppData\Roaming`
- `%commonprogramFiles%`: `C:\Program Files\Common Files`
- `%commonprogramFiles（x86）%`: `C:\Program Files (x86)\Common Files`
- `%homedrive%`: 打开您的主驱动器 `C:\`
- `%localappdata%`: 打开本地AppData文件夹 `C:\Users\{username}\AppData\Local`
- `%ProgramData%`: `C:\ProgramData`
- `%ProgramFiles%`: `C:\Program Files`或`C:\Program Files (x86)`
- `%ProgramFiles（x86）%`: `C:\Program Files (x86)`
- `%Public%`: `C:\Users\Public`
- `%SystemDrive%`: `C:`
- `%SystemRoot%`: 打开Windows文件夹 `C:\Windows`
- `%Temp%`: 打开临时文件文件夹 `C:\Users\{Username}\AppData\Local\Temp`
- `%UserProfile%`: 打开用户的配置文件 `C:\Users\{username}`
- `%AppData%\ Microsoft \ Windows \ Start菜单\程序\启动`: 打开Windows 10启动位置以获取程序快捷方式

## 28. windows常用工具命令

### 打开管理软件

- `taskmgr`: 任务管理器
- `appwiz.cpl`: 卸载或更改程序
- `sysdm.cpl`: 系统属性（环境变量、远程等）
- `ncpa.cpl`: 网络连接属性（适配器属性）
- `devmgmt.msc`: 设备管理器
- `diskmgmt.msc`: 磁盘管理
- `services.msc`: 服务管理
- `desk.cpl`: 显示相关设置
- `CHARMAP`: 字符映射表
- `cleanmgr`: 磁盘清理器
- `comexp.msc`: 组件服务，包含事件查看器和服务管理
- `eventvwr.msc`: 事件查看器
- `lusrmgr.msc`: 本地用户和组
- `secpol.msc`: 本地安全策略
- `Dxdiag`: DirectX诊断工具
- `Explorer`: 文件资源管理器
- `wf.msc`: 防火墙管理器
- `msconfig`: 启动相关配置，开机启动服务、启动项、引导等
- `msinfo32`: 系统详细信息
- `regedit`: 注册表编辑器
- `gpedit`: 本地组策略编辑器
- `certmgr.msc`: 证书管理器
- `fsmgmt.msc`: 文件夹共享管理
- `compmgmt.msc`: 计算机管理
- `wmimgmt.msc`: wmi管理控件
- `perfmon.msc`: 性能监视器
- `powercfg.cpl`: 电源选项
- `inetcpl.cpl`: internel选项
- `main.cpl`: 鼠标属性
- `joy.cpl`: 游戏手柄属性
- `timedate.cpl`: 日期和时间
- `intl.cpl`: 区域设置
- `sndvol`: 音量控制
- `mmsys.cpl`: 声音和音频设备属性
- `telephon.cpl`: 电话和调制解调器选项
- `wscui.cpl`: 控制面板的安全和维护
- `firewall.cpl`: 控制面板的防火墙设置
- `Odbcad32`: ODBC数据源管理

### 好用的命令

- `systeminfo`: 列举出系统所有信息
- `Defrag`: 磁盘碎片整理工具
- `Ftp`: ftp.exe程序
- `netstat`: 网络连接状态
- `nslookup`: 域名解析
- `telnet`: 连接测试
- `taskkill`: 杀进程
- `regsvr32`: 注册一个dll
- `tracert`: 路由追踪
- `sfc`: 文件扫描并修复

### Microsoft Office套件

winword -Microsoft Word
excel -Microsoft Excel
powerpnt -Microsoft PowerPoint
msaccess -Microsoft Access
Outlook -Microsoft Outlook
ois -Microsoft Picture Manager

## 29. timeout 等待

```bat
:: 等待5秒后继续操作
timeout /t 5
```

## 30. netstat 网络状态查询

### 30.1. 选项解释

- `-a`: 显示所有连接和监听端口。
- `-b`: 显示正在使用端口的可执行程序
- `-e`: 显示以太网统计信息，如传输的字节数、错误数等。
- `-n`: 以数字形式显示地址和端口号，而不是使用名称。
- `-o`: 显示与每个连接关联的进程 ID。
- `-r`: 显示路由表。
- `-p <proto>`: 协议

### 30.2. 实例

```bat
:: 显示路由表
C:\Users\User>netstat -r
===========================================================================
接口列表
 18...fe fc fe fa 1d f2 ......Sangfor FastIO Ethernet Adapter
  1...........................Software Loopback Interface 1
===========================================================================

IPv4 路由表
===========================================================================
活动路由:
网络目标        网络掩码          网关       接口   跃点数
          0.0.0.0          0.0.0.0    172.22.73.254     172.22.72.56     15
        127.0.0.0        255.0.0.0            在链路上         127.0.0.1    331
        127.0.0.1  255.255.255.255            在链路上         127.0.0.1    331
      172.22.72.0    255.255.254.0            在链路上      172.22.72.56    271
     172.22.72.56  255.255.255.255            在链路上      172.22.72.56    271
    172.22.73.255  255.255.255.255            在链路上      172.22.72.56    271
===========================================================================
永久路由:
  无

IPv6 路由表
===========================================================================
活动路由:
 接口跃点数网络目标                网关
  1    331 ::1/128                  在链路上
  1    331 ff00::/8                 在链路上
===========================================================================
永久路由:
  无

:: 显示当前正在使用的tcp连接，显示使用的进程可执行文件和PID，显示连接状态
C:\Users\User> netstat -anbo -p tcp

活动连接

  协议  本地地址          外部地址        状态           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       512
  RpcSs
 [svchost.exe]
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
 无法获取所有权信息
  TCP    0.0.0.0:4489           0.0.0.0:0              LISTENING       1276
  TermService
 [svchost.exe]
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       9196
  CDPSvc
 [svchost.exe]
  TCP    0.0.0.0:5357           0.0.0.0:0              LISTENING       4
 无法获取所有权信息
  TCP    0.0.0.0:7171           0.0.0.0:0              LISTENING       1276
  TermService
 [svchost.exe]
  TCP    0.0.0.0:7172           0.0.0.0:0              LISTENING       4956
 [SRAPSrv.exe]
```

## 31. sc 服务配置和查询

- 此命令仅在cmd下有效，powershell不支持，使用Get-Service和Set-service替换

### 31.1. 参数

```bat
C:\Users\User>sc \?

错误:  未知命令

描述:
        SC 是用来与服务控制管理器和服务进行通信
        的命令行程序。
用法:
        sc <server> [command] [service name] <option1> <option2>...


        <server> 选项的格式为 "\\ServerName"
        可通过键入以下命令获取有关命令的更多帮助: "sc [command]"
        命令:
          query-----------查询服务的状态，
                          或枚举服务类型的状态。
          queryex---------查询服务的扩展状态，
                          或枚举服务类型的状态。
          start-----------启动服务。
          pause-----------向服务发送 PAUSE 控制请求。
          interrogate-----向服务发送 INTERROGATE 控制请求。
          continue--------向服务发送 CONTINUE 控制请求。
          stop------------向服务发送 STOP 请求。
          config----------更改服务的配置(永久)。
          description-----更改服务的描述。
          failure---------更改失败时服务执行的操作。
          failureflag-----更改服务的失败操作标志。
          sidtype---------更改服务的服务 SID 类型。
          privs-----------更改服务的所需特权。
          managedaccount--更改服务以将服务帐户密码
                          标记为由 LSA 管理。
          qc--------------查询服务的配置信息。
          qdescription----查询服务的描述。
          qfailure--------查询失败时服务执行的操作。
          qfailureflag----查询服务的失败操作标志。
          qsidtype--------查询服务的服务 SID 类型。
          qprivs----------查询服务的所需特权。
          qtriggerinfo----查询服务的触发器参数。
          qpreferrednode--查询服务的首选 NUMA 节点。
          qmanagedaccount-查询服务是否将帐户
                          与 LSA 管理的密码结合使用。
          qprotection-----查询服务的进程保护级别。
          quserservice----查询用户服务模板的本地实例。
          delete ----------(从注册表中)删除服务。
          create----------创建服务(并将其添加到注册表中)。
          control---------向服务发送控制。
          sdshow----------显示服务的安全描述符。
          sdset-----------设置服务的安全描述符。
          showsid---------显示与任意名称对应的服务 SID 字符串。
          triggerinfo-----配置服务的触发器参数。
          preferrednode---设置服务的首选 NUMA 节点。
          GetDisplayName--获取服务的 DisplayName。
          GetKeyName------获取服务的 ServiceKeyName。
          EnumDepend------枚举服务依赖关系。

        以下命令不需要服务名称:
        sc <server> <command> <option>
          boot------------(ok | bad)指示是否应将上一次启动另存为
                          最近一次已知的正确启动配置
          Lock------------锁定服务数据库
          QueryLock-------查询 SCManager 数据库的 LockStatus
示例:
        sc start MyService


QUERY 和 QUERYEX 选项:
        如果查询命令带服务名称，将返回
        该服务的状态。其他选项不适合这种
        情况。如果查询命令不带参数或
        带下列选项之一，将枚举此服务。
    type=    要枚举的服务的类型(driver, service, userservice, all)
             (默认 = service)
    state=   要枚举的服务的状态 (inactive, all)
             (默认 = active)
    bufsize= 枚举缓冲区的大小(以字节计)
             (默认 = 4096)
    ri=      开始枚举的恢复索引号
             (默认 = 0)
    group=   要枚举的服务组
             (默认 = all groups)

语法示例
sc query                - 枚举活动服务和驱动程序的状态
sc query eventlog       - 显示 eventlog 服务的状态
sc queryex eventlog     - 显示 eventlog 服务的扩展状态
sc query type= driver   - 仅枚举活动驱动程序
sc query type= service  - 仅枚举 Win32 服务
sc query state= all     - 枚举所有服务和驱动程序
sc query bufsize= 50    - 枚举缓冲区为 50 字节
sc query ri= 14         - 枚举时恢复索引 = 14
sc queryex group= ""    - 枚举不在组内的活动服务
sc query type= interact - 枚举所有不活动服务
sc query type= driver group= NDIS     - 枚举所有 NDIS 驱动程序
```

### 31.2. query/qc 查询服务信息

```bat
:: 查看服务状态
C:\Users\User>sc query Srv2

SERVICE_NAME: Srv2
        TYPE               : 2  FILE_SYSTEM_DRIVER
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 1077  (0x435)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

:: 查看服务配置
C:\Users\User>sc qc LanmanServer
[SC] QueryServiceConfig 成功

SERVICE_NAME: LanmanServer
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 4   DISABLED
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\system32\svchost.exe -k netsvcs -p
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Server
        DEPENDENCIES       : SamSS
                           : Srv2
        SERVICE_START_NAME : LocalSystem

```

### 31.3. start/stop 启用停止服务

```bat
C:\Users\User>sc start sshd

SERVICE_NAME: sshd
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x1
        WAIT_HINT          : 0x12c
        PID                : 9600
        FLAGS              :

C:\Users\User>sc stop sshd

SERVICE_NAME: sshd
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

```

### 31.4. config 配置

```bat
C:\Users\User>sc config \?
描述:
        在注册表和服务数据库中修改服务项。
用法:
        sc <server> config [服务名称] <option1> <option2>...

选项:
注意: 选项名称包括等号。
      等号和值之间需要一个空格。
      要删除依赖关系，请使用单个“/”表示依赖关系值。
 type= <own|share|interact|kernel|filesys|rec|adapt|userown|usershare>
 start= <boot|system|auto|demand|disabled|delayed-auto>
 error= <normal|severe|critical|ignore>
 binPath= <.exe 文件的 BinaryPathName>
 group= <LoadOrderGroup>
 tag= <yes|no>
 depend= <依赖关系(以 / (正斜杠)分隔)>
 obj= <AccountName|ObjectName>
 DisplayName= <显示名称>
 password= <密码>
```

*实例*

```bat
:: 配置服务启动方式为禁用
C:\Users\User>sc config sshd start=disabled
[SC] ChangeServiceConfig 成功

```

## 32. takeown 获取所有权

```bat
:: /r为递归
C:\Users\User>takeown /f 目录名 /r
```

## 小技巧和踩坑记

### 1. 逻辑判断中多行设置变量不符合预期

**示例**

```bat
set a=123
if %a% == 123 (
    set a=%a%4
    set a=%a%5
)
echo a=%a%
```

- 认为输出的是`a=12345`
- 实际输出的是`a=1235`

**原因**

- 虽然括号内写成了多行，实际认为是一行
- batch解析时，**<font color="red">每一行命令会先将变量替换成值，再进行处理</font>**
- 所以上述翻译过来就是

```bat
set a=123
:: 下面将 %a% 替换成123得到
if 123 == 123 (
    set a=1234
    set a=1235
)
:: 上面一行 a=1235，所以这里将%a%替换成1235
echo a=1235
```

**解决**

- 使用`setlocal enabledelayedexpansion`，启用延迟解析
- 即不提前将值替换进去
- 然后变量两边的`%`换成`!`即可

```bat
set a=123
setlocal enabledelayedexpansion
if %a% == 123 (
    set a=!a!4
    set a=!a!5
)
echo a=%a%
```

- 输出`a=12345`

# 二、powershell

## 1. 一些基本用法

```bat
:: 打印环境变量
PS> $env:xxx
:: 设置环境变量
PS> $env:TestVar1="This is my environment variable"
:: 枚举环境变量，注意后面有个冒号
PS> ls env:
:: 删除环境变量
PS> del env:windir

:::::::::: 文件操作 ::::::::::
:: 新建文件
PS> New-item xxx.txt -type file
```

## 2. 网络相关

### 2.1. 网卡信息

#### 1) 获取网卡信息

```bat
PS> Get-NetIPInterface

ifIndex InterfaceAlias                  AddressFamily NlMtu(Bytes) InterfaceMetric Dhcp     ConnectionState PolicyStore
------- --------------                  ------------- ------------ --------------- ----     --------------- -----------
27      vEthernet (Default Switch)      IPv6                  1500            5000 Enabled  Connected       ActiveStore
21      以太网 2                        IPv6                  1500              15 Enabled  Connected       ActiveStore
1       Loopback Pseudo-Interface 1     IPv6            4294967295              75 Disabled Connected       ActiveStore
27      vEthernet (Default Switch)      IPv4                  1500            5000 Disabled Connected       ActiveStore
21      以太网 2                        IPv4                  1500              15 Disabled Connected       ActiveStore
1       Loopback Pseudo-Interface 1     IPv4            4294967295              75 Disabled Connected       ActiveStore

```

#### 2) 设置网卡优先级

```bat
:: InterfaceIndex是Get-NetIPInterface获取到的ifIndex
:: InterfaceMetric越大优先级越低
PS> Set-NetIPInterface -InterfaceIndex 59 -InterfaceMetric 50
```

## 3. 软件安装

### 3.1. 内置安装器

#### 1) 安装msixbundle

```bat
PS> Add-AppxPackage xxx.msixbundle
```

### 3.2. winget

#### 1) 安装winget

- github仓库: https://github.com/microsoft/winget-cli
- 到上面的release下载一个msixbundle安装包
- 使用powershell执行`Add-AppxPackage xxx.msixbundle`就可用了

#### 2) install 安装软件

##### 选项

- `--location [dir]`: 指定路径，部分软件不支持
- `--scope <user/machine>`: 指定安装范围，部分软件不支持，用户安装到用户的AppData下，电脑则安装到`C:\Program Files`下

## 4. 服务相关

### 4.1. Get-service 查看服务

```bat
:: 查看服务特定几列
PS C:\Users\User> Get-service sshd | Select-Object -property Name, StartType, Status

Name StartType  Status
---- ---------  ------
sshd  Disabled Stopped


```

### 4.2. Set-service 设置服务

```bat
Set-Service
   [-Name] <String>
   [-DisplayName <String>]
   [-Credential <PSCredential>]
   [-Description <String>]
   [-StartupType <ServiceStartupType>]
   [-Status <String>]
   [-SecurityDescriptorSddl <String>]
   [-Force]
   [-PassThru]
   [-WhatIf]
   [-Confirm]
   [<CommonParameters>]
```

- `ServiceStartupType`: 启动类型
  - Manual 手动
  - Disabled 禁用
  - Auto 自动

**实例**

```bat
:: 设置服务启动方式为手动
PS C:\Users\User> Set-service -Name sshd -StartupType Manual
:: 启动服务
PS C:\Users\User> Set-service -Name sshd -Status Running
:: 查看服务状态
PS C:\Users\User> Get-service sshd | Select-Object -property Name, StartType, Status

Name StartType  Status
---- ---------  ------
sshd    Manual Running


:: 设置服务启动方式为禁用
PS C:\Users\User> Set-service -Name sshd -StartupType Disabled
:: 停止服务
PS C:\Users\User> Set-service -Name sshd -Status Stopped
:: 查看服务状态
PS C:\Users\User> Get-service sshd | Select-Object -property Name, StartType, Status

Name StartType  Status
---- ---------  ------
sshd  Disabled Stopped
```

# 小技巧

## 1. powershell 使用 utf-8 编码

-   启动 powershell 时输入

```bat
chcp 65001
```

## 2. windows 10 启用 sshd 服务

- 需要在功能中安装`OpenSSH Server`
- 在命令行中执行，登陆帐号密码和微软一致

```bat
:: 开启
net start sshd
:: 关闭
net stop sshd
```
