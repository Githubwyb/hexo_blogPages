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
netsh interface portproxy add v4tov4 listenport=1234 listenaddress=127.0.0.1 connectport=445 connectaddress=127.0.0.1 protocol=tcp
```

### 14.4. 防火墙

#### 1) `advfirewall firewall`

```bat
:: 开放1234的tcp端口 ::
netsh advfirewall firewall add rule name=shared_folder action=allow protocol=TCP localport=1234
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
