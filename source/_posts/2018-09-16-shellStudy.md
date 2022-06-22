---
title: shell学习笔记
date: 2018-09-16 13:23:55
tags: [Linux]
categories: [Program, Shell]
top: 99
---

# 一、语法相关

## 1. 进制数字表示

```shell
# 二进制转十进制输出，其他同理
echo $((2#100100))
```

## 2. 目录特殊符号

- 上一级 `..`
- 当前目录 `.`
- 当前用户目录 `~`
- 根目录 `/`

## 3. 常用语法详解

### 3.1. `<>` 输入出重定向

```shell
ls ./ >a.text           # 正确输出到a.txt(覆盖)
ls ./ 1>a.text          # 正确输出到a.txt(覆盖)
ls ./ 2>a.txt           # 错误输出到a.txt(覆盖)
ls ./ 2>a.txt           # 错误输出到a.txt(覆盖)
ls ./ >a.txt 2>&1       # 标准输出和标准错误输出到a.txt(覆盖)
ls ./ &>a.txt           # 标准输出和标准错误输出到a.txt(覆盖)
# 追加使用>>
ls ./ >>a.text          # 正确输出到a.txt(追加)

ls ./ 2>&1              # 错误输出到标准输出

echo "test" >&2         # 将字符串输出到stderr

# < 将文件内容作为标准输入
while read -r line; do
    echo "$line"
done < test.txt

# <<<的含义，相当于管道，把字符串当作标准输入传给命令
echo "$a" | grep "xxx"
grep "xxx" <<< "$a"
```

### 3.2. $ 意义

```shell
$$      # Shell本身的PID（ProcessID）
$!      # Shell最后运行的后台Process的PID
$UID    # 当前用户的id
$PPID   # 父进程id
$?      # 最后运行的命令的结束代码（返回值）
$-      # 使用Set命令设定的Flag一览
$*      # 所有参数列表。如*所有参数列表。如"*“用「”」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
$@      # 所有参数列表。如@所有参数列表。如"@“用「”」括起来的情况、以"$1 $2 … $n" 的形式输出所有参数。
$#      # 添加到Shell的参数个数
$0      # Shell本身的文件名
$1～n   # 添加到Shell的各参数值。$1是第1参数、$2是第2参数…
```

### 3.3. cd 跳转目录命令

**特殊跳转**

- 跳转上一级 `..`
- 跳转当前目录 `.`
- 跳转当前用户目录 `~`
- 跳转根目录 `/`
- 跳转上一个目录 `cd -`
- 跳转前n目录 `cd -n`
- 跳转后n目录 `cd +n`

### 3.4. if 条件判断

**文件表达式**

```shell
if [ -e file ]      # 如果文件或者目录存在，不管有没有权限
if [ -f file ]      # 如果文件是普通文件，不是目录或者设备文件
if [ -b file ]      # 如果文件是块设备文件
if [ -c file ]      # 如果文件是字符设备文件
if [ -L file ]      # 如果文件是符号文件
if [ -d ...  ]      # 如果目录存在
if [ -s file ]      # 如果文件存在且非空
if [ -r file ]      # 如果文件存在且可读
if [ -w file ]      # 如果文件存在且可写
if [ -x file ]      # 如果文件存在且可执行
```

**整数变量表达式**

```shell
if [ int1 -eq int2 ]        # 如果int1等于int2
if [ int1 -ne int2 ]        # 如果不等于
if [ int1 -ge int2 ]        # 如果>=
if [ int1 -gt int2 ]        # 如果>
if [ int1 -le int2 ]        # 如果<=
if [ int1 -lt int2 ]        # 如果<
```

**字符串变量表达式**

```shell
if  [ $a = $b ]                     # 如果string1等于string2
                                    # 字符串允许使用赋值号做等号
if  [ $string1 != $string2 ]        # 如果string1不等于string2
if  [ -n $string ]                  # 如果string 非空(非0），返回0(true)
if  [ -z $string ]                  # 如果string 为空
if  [ $sting ]                      # 如果string 非空，返回0 (和-n类似)
```

### 3.5. for 循环

#### (1) 文件遍历

```shell
# 此行代表以换行符为分割，默认为空格、\t和\n
IFS=$'\n'
# 将test.txt的内容按行分割到value输出
for value in $(cat test.txt)
do
    echo "$value"
done
```

#### (2) index递增形式

```shell
for ((i=0;i<2;i++)); do
    echo "$i"
done
```

### 3.6. while 循环

```shell
# 按行读取文件内容
while IFS= read -r value
do
    echo "$value"
done < "test.txt"

# 按行读取变量内容
while IFS= read -r value
do
    echo "$value"
done <<< "$test"
```

### 3.7. switch 语句

```shell
case "$1" in
start)
    check_start "$@"
    ;;
stop)
    check_stop "$@"
    ;;
check)
    check_service "$@"
    ;;
*)
    echo "Usage: check_handle_service.sh start|stop|check [service]"
    exit 1
    ;;
esac
```

## 4. 函数调用

```shell
abc() {
    # ...
}

abc
```

函数调用不加括号

## 5. 字符串操作

```shell
string="12348213"

# 字符串长度
echo ${#string}

# 字符串最后一个字符，注意中间有一个空格
echo "${string: -1}"
```

### 5.1. 字符串替换

```shell
echo ${string/23/bb}   # 替换一次
echo ${string//23/bb}  # 双斜杠替换所有匹配
```

### 5.2. 字符串截取

```shell
# 删除最后一个/及前面所有。作用是删除路径，只留文件名
echo ${string##*/}
# 删除最后一个/及后面所有。作用是删除文件名，只留路径名
echo ${string%/*}
```

- #:表示从左开始算起，并且截取第一个匹配的字符
- ##:表示从左开始算起，并且截取最后一个匹配的字符
- %:表示从右开始算起，并且截取第一个匹配的字符
- %%:表示从右开始算起，并且截取最后一个匹配的字符

```shell
# 截取给定一段，m开始截取n个
echo "${str:m:n}"
# 截取给定一段，m开始到结尾
echo "${str:m}"
```

### 5.3. 字符串遍历

```shell
string="abc
b c     d"

# 这样会根据空格、换行和制表符分开
for item in $string; do
    echo "s $item e"
done
# 加了引号会认为整个字符串为一个
for item in "$string"; do
    echo "s $item e"
done
```

## 6. 数组

```shell
# 数组声明
test_arr=(
    "abc"
    "dce"
)

# 数组长度，bash 5.0不用加[@]，但是4.2就需要
len="${#test_arr[@]}"
```

### 6.1. 数组遍历

数组遍历分为几种形式

```shell
service_list=(
    "abc"
    "ddd"
    "dd d"
)
# for in 形式，在脚本中，不加双引号，第三个元素会拆分成两个元素
for sv in "${service_list[@]}"; do
    echo "$sv"
done

# i遍历的形式
len="${#service_list[@]}"
for ((i=0;i<"$len";i++)); do
    echo "a ${service_list[i]} b"
done
```

- 低版本shell中，local不能修饰数组

### 6.2. 数组拷贝

```shell
service_list=(
    "abc"
    "ddd"
    "dd d"
)

# 这样拿到的是原数组
arr_cp=("${service_list[@]}")
# 这样拿到的是数组转字符串再次根据空格、换行和制表符再次分割的数组
arr1_cp=({service_list[@]})
# 这样拿到的是个字符串
arr2_cp="${service_list[@]}"
```

## 7. eval

- eval是将字符串转成命令执行
- 可用于在脚本中将字符串转成变量名执行

```shell
service_list=(
    "abc"
    "ddd"
    "dd d"
)

str="service_list"
# 下面这句话将转成 test=("${service_list[@]}") 执行，即拷贝数组
eval "test=(\"\${${str}[@]}\")"

for item in "${test[@]}"; do
    echo "s $item e"
done
```

## 8. bash和source、`.`的区别

- source和`.`是一样的
- 三者都可以不需要文件有可执行权限
- bash类似于fork了一个子进程，内部变量和父进程没关系，但是父进程会等待子进程结果
- source和`.`更像直接把文件内容拷贝到位置上执行，主要用于导入变量和函数

## 9. 算术表达式

### 9.1. i++表示

#### (1) 使用let表示

```shell
i=0

let i+=1
let 'i+=1'
```

#### (2) 用`(())`，这种用法常见于for循环中

```shell
((i++))
```

#### (3) 用expr

- 中间要有空格，否则就成字符串拼接了

```shell
i=0
i=`expr $i + 1`
echo "$i"       # 1
i=`expr $i+1`
echo "$i"       # 1+1
```

#### (4) 用如下模式

```shell
i=$[$i+1];      # 分号不能少
i=$(($i+1))     # 中间可以加空格
```

# 二、系统命令详解

## 1. find 文件查找

### 1.1. 选项

- `-type`: 类型，d为文件夹，f为文件
- `-name`: 文件名称，支持`*`
- `-atime`: 访问时间，后跟参数，查看示例
- `-mtime`: 修改时间，后跟参数，查看示例
- `-ctime`: 元数据修改时间，后跟参数，查看示例

### 1.2. 实例

```shell
# 查找当前目录下文件类型的，修改时间在0.5天以内的文件
find ./ -type f -mtime -0.5
# 查找当前目录下文件类型的所有php文件
find ./ -type f -name "*.php"
```

## 2. file

### 2.1. 实例

```shell
# 查看文件类型，不显示文件名
=> file -b test.txt
ASCII text
```

## 3. tr

### 3.1. 实例

```shell
# 大小写转化
echo "HELLO" | tr '[:upper:]' '[:lower:]'
echo "hello" | tr '[:lower:]' '[:upper:]'
```

## 4. watch

watch命令以周期性的方式执行给定的指令，指令输出以全屏方式显示。watch是一个非常实用的命令，基本所有的Linux发行版都带有这个小工具，如同名字一样，watch可以帮你监测一个命令的运行结果，省得你一遍遍的手动运行。

### 4.1. 选项

- -n：指定指令执行的间隔时间（秒）；
- -d：高亮显示指令输出信息不同之处；
- -t：不显示标题。

### 4.2. 参数
指令：需要周期性执行的指令。

### 4.3. 实例

```shell
watch uptime
watch -t uptime
watch -d -n 1 netstat -ntlp
watch -d 'ls -l | fgrep goface'     //监测goface的文件
watch -t -differences=cumulative uptime
watch -n 60 from            //监控mail
watch -n 1 "df -i;df"       //监测磁盘inode和block数目变化情况
```

## 5. tail 实时查看文件内容（可用于查看log文件）

```shell
tail -f (fileName)
```

### 5.1. tail显示高亮

```shell
tail -f sys.log | perl -pe 's/(关键词1)|(关键词2)|(关键词3)/\e[1;颜色1$1\e[0m\e[1;颜色2$2\e[0m\e[1;颜色3$3\e[0m/g'
```
**两个示例**

- 黄字，高亮加粗显示 `[1;33m`
- 红底黄字，高亮加粗显示 `[1;41;33m`

**效果配置列举**

```
前景色
30m：黑
31m：红
32m：绿
33m：黄
34m：蓝
35m：紫
36m：青
37m：白

背景色
40：黑
41：红
42：绿
43：黄
44：蓝
45：紫
46：青
47：白

动效设置
[1; 设置高亮加粗
[4; 下划线
[5; 闪烁
```

## 6. sort 排序

### 6.1. 一些基本操作

```shell
# 将test.txt中排序并去重，但是只输出到控制台，可以用>写到文件
cat test.txt | sort -u
# sort -V 可以排序版本号类型，如判断 2.1.9和2.1.10
xxx | sort -V
```

## 7. grep 查找内容

- `-r`: 遍历子目录
- `-n`: 遍历行数
- `-i`: 大小写无关
- `-v`: 排除
- `-E`: 衍生为正则表达式（用|代表或等）
- `-o`: 正则只输出PATTERN部分

### 7.1. 内容匹配

- 查找字符串用`^`代表开头，用`$`代表结束，可以使用`^xxx$`进行全匹配

```shell
# 匹配ab开头的字符串
cat test.txt | grep "^ab"
# 匹配bc结束的字符串
cat test.txt | grep "ab$"
# 全匹配abc
cat test.txt | grep "^abc$"
```

### 7.2. 查找进程排除grep

- 通常使用`ps + grep`查找进程，但是会多出一行当前查找的grep进程

```shell
ps -aux | grep xxx | grep -v grep
```

### 7.3. 查找包含内容的有几行

```shell
# 相当于 cat test.txt | grep "test" | wc -l
grep -c "test" < test.txt
```

### 7.4. 查找目录下所有文件匹配对应字符串

```shell
grep -nr "test" ./
```
### 7.5. 正则查找，只显示匹配段

```shell
# 显示 APPVERSION=abc 到|停止，即所有非|都匹配，匹配多个
echo "APPVERSION=abc|ddd" | grep -o "APPVERSION=[^|]*"
```

## 8. wc 查看文件或者内容行数

- 结合查找进程可以返回匹配的进程个数
- <font color="red">`wc -l`返回的是`\n`的个数，如果最后一行没有换行，将不会统计最后一行</font>

```shell
ps -aux | grep xxx | grep -v grep | wc -l
```

## 9. addr2line 根据地址定位代码位置

示例

```shell
=> addr2line 0x2026 -e run -f
main
/xxxx/xxxx/xxxx/xxxx/main.cpp:39
```

- `-e`: 指定可执行文件
- `-f`: 显示函数名

### 9.1. 注意事项

- gcc编译需要添加`-g`参数
- 不加`-g`放到生产环境，加`-g`的用于找代码位置

## 10. date 时间工具

```shell
########## 时间计算 ##########
# 时间戳转正常时间格式
=> date -d @1587536520
2020年 04月 22日 星期三 14:22:00 CST

# 时间字符串转时间戳
=> date -d "20200422" +%s
1587536520

########## 查看当前时间 ##########
# 时间格式化
=> date +'%Y-%m-%d %A %H:%M:%S'
2020-05-08 Friday 19:12:22
# 当前时间戳
=> date +'%s'
1644981572

########## 设置当前时间 ##########
=> date -s @1587536520
```

## 11. dd 批量拷贝命令

### 11.1. 实例

**1. 生成大文件**

```shell
# 生成指定大小文件，1M一个单位，120个单位，生成到/sftmpfs/test
dd if=/dev/zero of=/sftmpfs/test bs=1M count=120
```

## 12. watch 监听命令

### 12.1. 实例

```shell
# 监听内存变化，每秒刷新一次
watch -n 1 free -h
```

## 13. iptables 防火墙

### 13.1. 知识填充

#### 1) 表 table

nat

#### 2) 链 chain

- `INPUT`: 进来的数据包
- `OUTPUT`: 外出的数据包
- `FORWARD`: 转发数据包时
- `PREROUTING`: 对数据包作路由选择前
- `POSTROUTING`: 对数据包作路由选择后

### 13.2. 选项

- `-t [table]`: 查看具体表，表有`nat`等

### 13.3. 实例

```shell
########## 插入规则 ##########
# -I插入到前面，-A插入到后面
iptables -I INPUT -i eth0 -s 200.200.87.48 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

########## 查询规则 ##########
iptables -L INPUT --line-numbers

########## 删除规则 ##########
# line_number是上面查出来的
iptables -D INPUT [line_number]

```

## 14. ls 列举目录

### 13.1. 选项

- `-l`: 列表显示详细信息
- `-h`: 目录大小加上单位
- `-t`: 按照时间排序
- `-S`: 按照大小排序
- `-r`: 反向排序

### 13.2. 时间显示格式修改

```shell
=> export TIME_STYLE='+%Y-%m-%d %H:%M:%S'
=> ls -l test.js
-rw-r--r-- 1 xxx xxx 87 2022-03-07 16:02:09 test.js
```

## 15. netstat 网络状态查看

```
usage: netstat [-vWeenNcCF] [<Af>] -r         netstat {-V|--version|-h|--help}
       netstat [-vWnNcaeol] [<Socket> ...]
       netstat { [-vWeenNac] -i | [-cnNe] -M | -s [-6tuw] }

        -r, --route              display routing table
        -i, --interfaces         display interface table
        -g, --groups             display multicast group memberships
        -s, --statistics         display networking statistics (like SNMP)
        -M, --masquerade         display masqueraded connections

        -v, --verbose            be verbose
        -W, --wide               don't truncate IP addresses
        -n, --numeric            don't resolve names
        --numeric-hosts          don't resolve host names
        --numeric-ports          don't resolve port names
        --numeric-users          don't resolve user names
        -N, --symbolic           resolve hardware names
        -e, --extend             display other/more information
        -p, --programs           display PID/Program name for sockets
        -o, --timers             display timers
        -c, --continuous         continuous listing

        -l, --listening          display listening server sockets
        -a, --all                display all sockets (default: connected)
        -F, --fib                display Forwarding Information Base (default)
        -C, --cache              display routing cache instead of FIB
        -Z, --context            display SELinux security context for sockets

  <Socket>={-t|--tcp} {-u|--udp} {-U|--udplite} {-S|--sctp} {-w|--raw}
           {-x|--unix} --ax25 --ipx --netrom
  <AF>=Use '-6|-4' or '-A <af>' or '--<af>'; default: inet
  List of possible address families (which support routing):
    inet (DARPA Internet) inet6 (IPv6) ax25 (AMPR AX.25)
    netrom (AMPR NET/ROM) ipx (Novell IPX) ddp (Appletalk DDP)
    x25 (CCITT X.25)
```

- `-a`: 显示所有连接，包括正在连接的
- `-l`: 显示监听的连接，只显示被监听的端口和套接字文件
- `-p`: 显示进程名
- `-n`: 不把端口自动推测成服务，显示原始端口

### 实例

```shell
netstat -tnlp | grep 1234
```

## 16. sed 文件查找替换打印

- sed命令有点复杂，一直不会用，但是很强大

### 16.1. 选项

- `-i[suffix]`: 替换文件内容，如果定义了suffix，会备份一份到`xxxsuffix`

### 16.2. 示例

**1. 添加一行**

```shell
sed -i "/^start()/a\
	if ! "${script}" check "${proc_name}"; then exit 0; fi
" "${file_name}"
```

**2. 添加多行**

```shell
sed -i "/^start()/a\
	if ! \"${script}\" check \"${proc_name}\"; then\n\
		/bin/run.sh \"${file}\" &\n\
		exit 0\n\
	fi
" "${file_name}"
```

**3. 替换**

```shell
# 将aaa替换成bbb
sed -i "s/aaa/bbb/" "${file_name}"
```

**3. 删除所有的html标签**

```shell
sed -i 's/<[^>]*>//g' "${file_name}"
```

**4. 截取文件中间部分**

```shell
sed -n '15,20p' test.txt
```

### 16.3. 正则

- ` *`: 匹配空格零次或多次
- ` \?`: 匹配空格零次或一次
- ` \+`: 匹配空格一次或多次

### 特殊用法

**1. pattern带大括号，需要转义**

- 将大括号单独用单引号包裹，两边闭合单引号

```shell
sed -i 's/aaa''{''/bbb''{''/' aaa.txt
```

## 17. echo 输出

- `-n`: 不换行

## 18. diff & patch 差异输出和应用

**diff**

- `-r`: 递归对比文件夹改动
- `-u`: 合并的方式输出，类似git diff，用于生成patch

**git diff**

- `-w`: 忽略空格改动
- `--relative`: 相对目录，不使用git绝对目录

**patch**

- `-p<n>`: 裁剪前导`/`和目录，p1忽略第一个`/`，以此类推
- `-l`: 忽略空格
- `-i <patch_path>`: 输入patch文件
- `--no-backup-if-mismatch`: 如果改动不完全，不要备份文件

### 18.1 git改动应用到某个目录上（非git标准目录）

```shell
# 和某个提交做diff，输出到patch文件
git diff 09e99a0b6273c26007c16dae48822a6d107eadef -w --relative . > ~/temp/patch
# 将patch文件应用到当前目录下
patch -p1 -l --no-backup-if-mismatch -i ~/temp/patch
```

### 18.2 文件夹改动应用到另一个文件夹

```shell
# 两个同级目录对比（必须同级，不然应用差异时目录会有问题）
diff -ru dir1 dir2 > ~/temp/patch
# 进入到目录里面，不然patch删除目录会不识别
cd dir3
# 将patch文件应用到当前目录下
patch -p1 -l --no-backup-if-mismatch -i ~/temp/patch
```

## 19. fallocate 创建大文件

fallocate创建的文件仅仅是占用磁盘，没有内容，所以只有很少的I/O操作，比dd快很多

### 19.1. 几种基本用法

```shell
# 创建一个10G的文件
fallocate -l 10G test.img
```

**<font color="red">不知道为什么，fallocate不能重复对一个文件执行，想要重新分配需要手动删除前一个文件</font>**

## 20. lsof 查看系统文件占用（包括端口）

### 20.1. 几种基本用法

```shell
# 根据进程号查看端口占用
lsof -i | grep [pid]

# 根据文件名查看文件占用
lsof /path/to/file

# 根据路径查看目录下（不包含子目录）文件占用
lsof +d /path/to/dir/

# 根据路径查看目录下（包含子目录）文件占用
lsof +D /path/to/dir/
```

## 21. top 查看系统占用

### 21.1. 几个快捷键

- `Shift + E`: 调整内存单位
- `1`: 切换cpu统计模式，所有/详细
- `Shift + P`: cpu占用排序
- `Shift + M`: 内存占用排序
- `m`: 切换内存显示样式
- `c`: 显示进程详细命令

### 21.2. <span id="process_status">进程状态解析</span>

- R——Runnable（运行）: 正在运行或在运行队列中等待
- S——sleeping（中断）: 休眠中，受阻，在等待某个条件的形成或接收到信号
- D——uninterruptible sleep(不可中断): 收到信号不唤醒和不可运行，进程必须等待直到有中断发生
- Z——zombie（僵死）: 进程已终止，但进程描述还在，直到父进程调用wait4()系统调用后释放
- T——traced or stoppd(停止): 进程收到SIGSTOP,SIGSTP,SIGTOU信号后停止运行

**后缀表示**

- <: 优先级高的进程
- N: 优先级低的进程
- L: 有些页被锁进内存
- s: 进程的领导者（在它之下有子进程）
- l: ismulti-threaded (using CLONE_THREAD, like NPTL pthreads do)
- +: 位于后台的进程组

## 22. awk 逐行处理显示

### 22.1. 几种基本用法

### 22.2. 骚操作实例

**1. git计算代码行数**

```shell
git diff xxxx --numstat | awk '{add+=$1;del+=$2} END {print "Add =",add,"Delete =",del}'
```

## 23. md5sum 计算MD5

### 23.1. 几种基本用法

```shell
# 计算字符串的md5
echo -n "xxx" | md5sum
```

## 24. udevadm 系统usb管理

### 24.1 几种基本用法

```shell
# 监听usb事件，查看详细信息可以添加选项
udevadm monitor --environment --udev
# 设置日志级别
udevadm control --log-priority=debug
```

## 25. lsblk 树型查看硬盘分区信息

- `-d`: 只显示硬盘，不显示分区
- `-o xxx,xxx`: 指定显示列
  - `name`: 名字
  - `rota`: 是否是转动磁盘，也就是机械硬盘。1为机械硬盘；0为固态硬盘

```shell
=> lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 298.1G  0 disk
├─sda1   8:1    0 954.9M  0 part
├─sda2   8:2    0     1K  0 part
├─sda5   8:5    0 198.1G  0 part /home
├─sda6   8:6    0  11.2G  0 part [SWAP]
├─sda7   8:7    0  46.6G  0 part /
└─sda8   8:8    0  41.3G  0 part /opt
=> lsblk -d
NAME MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda    8:0    0 298.1G  0 disk
=> lsblk -d -o name,rota
NAME ROTA
sda     1
```

## 26. blkid 查看已挂载的硬盘的uuid信息

## 27. tune2fs 修改硬盘uuid

## 28. route 更新路由信息

### 28.1. 一些基本用法

```shell
########## 查看路由 ##########
route -n

########## 添加路由 ##########
route add -net 1.1.1.1/24 gw 1.1.1.254
route add -net 1.1.1.1/24 dev eth0

########## 删除路由 ##########
route del -net 1.1.1.1/24 gw 1.1.1.254
route del -net 1.1.1.1/24 dev eth0
```

## 29. ps 查看系统进程

### 29.1. 选项含义

**参数**

- `a`: 列出所有用户的程序
- `u`: 显示所属用户
- `x`: 显示所有程序，不加只显示终端控制的程序
- `c`: 只显示指令名称
- `f`: 使用ascii字符展示关系
- `e`: 显示环境变量

## 30. ip 系统网络配置

- 默认配置ipv4，配置ipv6需要加上`-6`

### 30.1. 路由相关

```shell
########### 查看路由 ##########
=> ip route show
default via 199.200.2.254 dev ens18 proto static metric 100
199.200.0.0/16 dev ens18 proto kernel scope link src 199.200.2.199 metric 100
255.253.254.0/24 dev ens19 proto kernel scope link src 255.253.254.33
# 根据访问ip查看路由
=> ip r get 199.200.2.170
199.200.2.170 via 10.240.255.254 dev ens18 src 10.240.17.101 uid 1000
    cache

########## 配置路由 ##########
# 默认路由
ip route add default via 192.168.0.150/24
# 静态路由
ip route add 172.16.32.32 via 192.168.0.150/24 dev enp0s3
```

### 30.2. 查看网卡ip

```shell
=> ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fe:fc:fe:07:fe:6a brd ff:ff:ff:ff:ff:ff
    inet 199.200.2.199/16 brd 199.200.255.255 scope global noprefixroute ens18
       valid_lft forever preferred_lft forever
    inet6 fe80::34c9:aa6d:8630:e99a/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: ens19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fe:fc:fe:79:83:d5 brd ff:ff:ff:ff:ff:ff
    inet6 2001::199/112 scope global noprefixroute
       valid_lft forever preferred_lft forever
    inet6 fe80::d501:6609:cfd1:9970/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

### 30.3. 配置网卡

```shell
# 启用/禁用网卡
ip link set eth0 up
ip link set eth0 down

# 分配ip
# brd + 根据掩码和ip配置广播地址
ip addr add 199.200.1.1/24 brd + dev eth0
# 单独配置广播地址
ip addr add broadcast 192.168.0.255 dev eth0

# 删除分配的ip
ip addr del 192.168.0.10/24 dev eth0
```

## 31. read 读取输入

### 31.1. 读取用户输入

```shell
read -s 5 -p "do you wan't to continue [y/n]" input
case "$input" in
case [Yy]*)
```

## 32. conntrack 连接跟踪

### 32.1. 选项解释

```shell
# 查看连接跟踪表
conntrack -L <options>
```

- `-p [protocol]`
- `-s [src_ip]`
- `-d [dst_ip]`
- `--sport [src_port]`
- `--dport [dst_port]`
- `--state [NONE | SYN_SENT | SYN_RECV | ESTABLISHED | FIN_WAIT | CLOSE_WAIT | LAST_ACK | TIME_WAIT | CLOSE | LISTEN]`: 过滤状态

## 33. ss 查看socket占用情况

### 33.1. 选项解释

- `-t`: tcp
- `-u`: udp
- `-x`: unix
- `-l`: listening
- `-p`: 展示进程信息
- `-m`: 展示socket内存占用情况
- `-n`: 不把端口自动推测成服务，显示原始端口
v9k
### 33.2. 使用实例

```shell
# 查看22端口占用
=> ss -tnlp | grep ':22'
LISTEN  0        128                    0.0.0.0:22               0.0.0.0:*       users:(("sshd",pid=25985,fd=3))
LISTEN  0        128                       [::]:22                  [::]:*       users:(("sshd",pid=25985,fd=4))

# 查看25985进程占用情况
=> ss -tnlp | grep 'pid=25985'
LISTEN  0        128                    0.0.0.0:22               0.0.0.0:*       users:(("sshd",pid=25985,fd=3))
LISTEN  0        128                       [::]:22                  [::]:*       users:(("sshd",pid=25985,fd=4))
```

## 34. pwd

### 34.1. 选项解释

- `-L`: 逻辑路径，软连接路径
- `-P`: 物理路径，软连接映射的真实路径

## 35. mknod

- 用于挂载设备节点

### 35.1. 示例

```shell
# 挂载字符型设备，主设备号1，次设备号8
# c     character special file
# 1 8   Device type: 1,8
mknod /dev/random c 1 8
```

## 36. sudo

### 36.1. 选项

- `-E`: 继承当前环境变量

## 37. sysctl 修改内核参数

### 37.1. 选项

- `-a`: 列举所有内核参数
- `-p`: 从`/etc/sysctl.conf`加载配置
- `-w`: 修改内核参数

#### 示例

```shell
# 查找当前配置
=> sudo sysctl -a | grep compa
net.ipv4.nexthop_compat_mode = 1
vm.compact_unevictable_allowed = 1
vm.compaction_proactiveness = 20
vm.mmap_rnd_compat_bits = 8
# 临时修改kernel配置
=> sudo sysctl -w vm.compaction_proactiveness=0
vm.compaction_proactiveness = 0
=> sudo sysctl -a | grep compa
net.ipv4.nexthop_compat_mode = 1
vm.compact_unevictable_allowed = 1
vm.compaction_proactiveness = 0
vm.mmap_rnd_compat_bits = 8
# 永久修改kernel配置
=> sudo bash -c "echo \"vm.compaction_proactiveness=0\" > /etc/sysctl.conf"
# 从配置文件读取配置
=> sudo sysctl -p
vm.compaction_proactiveness = 0
=> sudo sysctl -a | grep compa
net.ipv4.nexthop_compat_mode = 1
vm.compact_unevictable_allowed = 1
vm.compaction_proactiveness = 0
vm.mmap_rnd_compat_bits = 8
```

## 38. pmap查看进程内存情况

### 选项

- `-x`: 显示扩展格式
- `-d`: 显示设备格式
- `-q`: 不显示头尾行
- `-V`: 显示指定版本

### 示例

```shell
=> pmap -x 76381
76381:   build/output/run
Address           Kbytes     RSS   Dirty Mode  Mapping
0000560aff840000      44      44       0 r---- run
0000560aff84b000     484     484       0 r-x-- run
0000560aff8c4000     288      64       0 r---- run
0000560aff90c000      12      12      12 r---- run
0000560aff90f000       4       4       4 rw--- run
0000560aff910000       4       4       4 rw---   [ anon ]
0000560affa43000     132      16      16 rw---   [ anon ]
00007f31dae00000     160     160       0 r---- libc.so.6
00007f31dae28000    1512    1044       0 r-x-- libc.so.6
00007f31dafa2000     352     180       0 r---- libc.so.6
00007f31daffa000      16      16      16 r---- libc.so.6
00007f31daffe000       8       8       8 rw--- libc.so.6
00007f31db000000      52      20      20 rw---   [ anon ]
00007f31db200000     612     612       0 r---- libstdc++.so.6.0.30
00007f31db299000    1112     764       0 r-x-- libstdc++.so.6.0.30
00007f31db3af000     476     116       0 r---- libstdc++.so.6.0.30
00007f31db426000      52      52      52 r---- libstdc++.so.6.0.30
00007f31db433000       4       4       4 rw--- libstdc++.so.6.0.30
00007f31db434000      12      12      12 rw---   [ anon ]
00007f31db474000      20      16      16 rw---   [ anon ]
00007f31db479000      12      12       0 r---- libgcc_s.so.1
00007f31db47c000      92      64       0 r-x-- libgcc_s.so.1
00007f31db493000      16      16       0 r---- libgcc_s.so.1
00007f31db497000       4       4       4 r---- libgcc_s.so.1
00007f31db498000       4       4       4 rw--- libgcc_s.so.1
00007f31db499000      56      56       0 r---- libm.so.6
00007f31db4a7000     484     224       0 r-x-- libm.so.6
00007f31db520000     376       0       0 r---- libm.so.6
00007f31db57e000       4       4       4 r---- libm.so.6
00007f31db57f000       4       4       4 rw--- libm.so.6
00007f31db5af000       8       8       8 rw---   [ anon ]
00007f31db5b1000       8       8       0 r---- ld-linux-x86-64.so.2
00007f31db5b3000     156     156       0 r-x-- ld-linux-x86-64.so.2
00007f31db5da000      44      40       0 r---- ld-linux-x86-64.so.2
00007f31db5e6000       8       8       8 r---- ld-linux-x86-64.so.2
00007f31db5e8000       8       8       8 rw--- ld-linux-x86-64.so.2
00007ffe6b20a000     136      24      24 rw---   [ stack ]
00007ffe6b2c7000      16       0       0 r----   [ anon ]
00007ffe6b2cb000       8       4       0 r-x--   [ anon ]
ffffffffff600000       4       0       0 --x--   [ anon ]
---------------- ------- ------- -------
total kB            6804    4276     228
```

## 39. setcap 提权命令

参考 [Linux的capability深入分析](https://www.cnblogs.com/sky-heaven/p/12096758.html)

- 从2.1版开始,Linux内核有了能力(capability)的概念,即它打破了UNIX/LINUX操作系统中超级用户/普通用户的概念,由普通用户也可以做只有超级用户可以完成的工作
- 比如给kill命令杀死其他进程的能力，但是不能重启系统，也不能修改文件。仅拥有杀死其他进程的能力

### 39.1. 能力列举

- `CAP_CHOWN`: 修改文件属主的权限
- `CAP_DAC_OVERRIDE`: 忽略文件的DAC访问限制
- `CAP_DAC_READ_SEARCH`: 忽略文件读及目录搜索的DAC访问限制
- `CAP_FOWNER`: 忽略文件属主ID必须和进程用户ID相匹配的限制
- `CAP_FSETID`: 允许设置文件的setuid位
- `CAP_KILL`: 允许对不属于自己的进程发送信号
- `CAP_SETGID`: 允许改变进程的组ID
- `CAP_SETUID`: 允许改变进程的用户ID
- `CAP_SETPCAP`: 允许向其他进程转移能力以及删除其他进程的能力
- `CAP_LINUX_IMMUTABLE`: 允许修改文件的IMMUTABLE和APPEND属性标志
- `CAP_NET_BIND_SERVICE`: 允许绑定到小于1024的端口
- `CAP_NET_BROADCAST`: 允许网络广播和多播访问
- `CAP_NET_ADMIN`: 允许执行网络管理任务
- `CAP_NET_RAW`: 允许使用原始套接字
- `CAP_IPC_LOCK`: 允许锁定共享内存片段
- `CAP_IPC_OWNER`: 忽略IPC所有权检查
- `CAP_SYS_MODULE`: 允许插入和删除内核模块
- `CAP_SYS_RAWIO`: 允许直接访问/devport,/dev/mem,/dev/kmem及原始块设备
- `CAP_SYS_CHROOT`: 允许使用chroot()系统调用
- `CAP_SYS_PTRACE`: 允许跟踪任何进程
- `CAP_SYS_PACCT`: 允许执行进程的BSD式审计
- `CAP_SYS_ADMIN`: 允许执行系统管理任务，如加载或卸载文件系统、设置磁盘配额等
- `CAP_SYS_BOOT`: 允许重新启动系统
- `CAP_SYS_NICE`: 允许提升优先级及设置其他进程的优先级
- `CAP_SYS_RESOURCE`: 忽略资源限制
- `CAP_SYS_TIME`: 允许改变系统时钟
- `CAP_SYS_TTY_CONFIG`: 允许配置TTY设备
- `CAP_MKNOD`: 允许使用mknod()系统调用
- `CAP_LEASE`: 允许修改文件锁的FL_LEASE标志

### 39.2. 示例

```shell
# 给tcpdump设置允许使用原始套接字，使得普通用户可以tcpdump
=> sudo setcap 'CAP_NET_RAW+eip' /usr/sbin/tcpdump
# 删除tcpdump的权限
=> sudo setcap -r /usr/sbin/tcpdump
```

# 三、工具命令

## 1. 文件夹目录大小 du

当前目录下所有子文件夹和文件的大小

```shell
du ./ -h --max-depth=1
```

## 2. 抓包工具 tcpdump

### 2.1. 选项

- `-c [num]`: 抓取的包的数量
- `-i`: 要监听的网口，不给默认为第一个网口
- `-n`: 对地址以数字方式显式，否则显示为主机名，也就是说-n选项不做主机名解析
- `-nn`: 除了-n的作用外，还把端口显示为数值，否则显示端口服务名
- `-w xxx.pcap`: 保存到文件，pcap结尾，用于wirshark分析
- `-v`: 显示更多信息，如ip包的存活时间、标识号、总长度和选项，并且检查ip包或icmp包的头部checksum
- `-vv`: 除`-v`作用外，NFS回包和SMB包会被解码
- `-X`: 打印每一个包，以hex和ascii的形式打印

### 2.2. 过滤器

- `host [ip_addr]`: 源ip和目的ip为`ip_addr`
- `[src|dst] [ip_addr]`: 源ip/目的ip为`ip_addr`
- `[protocol]`: 协议
  - `tcp`
  - `udp`
  - `icmp`
  - `icmp6`: ping ipv6
- `port [port_num]`: 端口
- `[src|dst] port [port_num]`: 源/目的端口
- `[tcp|udp] port [port_num]`: tcp/udp端口
- `[tcp|udp] [src|dst] port [port_num]`: tcp/udp的源/目的端口
- `ip6`: ipv6

### 2.3. 示例

```shell
# 抓5个192.168.100.62到本机eth0的icmp（ping）包
tcpdump -c 5 -nnvv -i eth0 icmp and src 192.168.100.62
# 抓取10个本机ens33的目的端口为22的tcp包
tcpdump -c 10 -nnvv -i ens33 tcp dst port 22
```

## 3. 网络嗅探 nmap

[nmap使用](https://baike.baidu.com/item/nmap/1400075?fr=aladdin)

## 4. 远程连接 ssh

### 4.1. ssh生成公钥和私钥

```shell
ssh-keygen -t rsa -C "xxx@xxx.com"
```

### 4.2. ssh通过代理访问

```shell
# socks5代理
ssh -o "ProxyCommand=nc -x 127.0.0.1:1080 %h %p" wangyubo@172.22.2.108
```

### 4.3. 端口转发

- 手册解释如下

```
     -L [bind_address:]port:host:hostport
     -L [bind_address:]port:remote_socket
     -L local_socket:host:hostport
     -L local_socket:remote_socket
             Specifies that connections to the given TCP port or Unix socket on the local (client) host are to be forwarded to the given host and port, or Unix socket, on the remote
             side.  This works by allocating a socket to listen to either a TCP port on the local side, optionally bound to the specified bind_address, or to a Unix socket.  Whenever a
             connection is made to the local port or socket, the connection is forwarded over the secure channel, and a connection is made to either host port hostport, or the Unix
             socket remote_socket, from the remote machine.

             Port forwardings can also be specified in the configuration file.  Only the superuser can forward privileged ports.  IPv6 addresses can be specified by enclosing the address
             in square brackets.

             By default, the local port is bound in accordance with the GatewayPorts setting.  However, an explicit bind_address may be used to bind the connection to a specific address.
             The bind_address of “localhost” indicates that the listening port be bound for local use only, while an empty address or ‘*’ indicates that the port should be available from
             all interfaces.
    ...
     -R [bind_address:]port:host:hostport
     -R [bind_address:]port:local_socket
     -R remote_socket:host:hostport
     -R remote_socket:local_socket
     -R [bind_address:]port
             Specifies that connections to the given TCP port or Unix socket on the remote (server) host are to be forwarded to the local side.

             This works by allocating a socket to listen to either a TCP port or to a Unix socket on the remote side.  Whenever a connection is made to this port or Unix socket, the con‐
             nection is forwarded over the secure channel, and a connection is made from the local machine to either an explicit destination specified by host port hostport, or
             local_socket, or, if no explicit destination was specified, ssh will act as a SOCKS 4/5 proxy and forward connections to the destinations requested by the remote SOCKS
             client.

             Port forwardings can also be specified in the configuration file.  Privileged ports can be forwarded only when logging in as root on the remote machine.  IPv6 addresses can
             be specified by enclosing the address in square brackets.

             By default, TCP listening sockets on the server will be bound to the loopback interface only.  This may be overridden by specifying a bind_address.  An empty bind_address,
             or the address ‘*’, indicates that the remote socket should listen on all interfaces.  Specifying a remote bind_address will only succeed if the server's GatewayPorts option
             is enabled (see sshd_config(5)).

             If the port argument is ‘0’, the listen port will be dynamically allocated on the server and reported to the client at run time.  When used together with -O forward the al‐
             located port will be printed to the standard output.
```

- 用法

```shell
# 监听本地端口9229，转发给远程的9229端口
ssh -L 9229:127.0.0.1:9229 admin@199.200.2.170
# 监听远程端口1234，转发给本地的5678端口
ssh -R 1234:127.0.0.1:5678 admin@199.200.2.170
```

### 踩坑记

#### (1) 低版本ssh连接失败，提示key不支持

```
no matching host key type found. Their offer: ssh-rsa,ssh-dss
```

- 修改`/etc/ssh/ssh_config`，添加下面字段即可

```conf
Host *
    HostkeyAlgorithms +ssh-rsa
    PubkeyAcceptedAlgorithms +ssh-rsa
```

#### (2) ssh首次登录总要等一会才提示密码框

- 原因是ssh去找主机名的DNS
- 快速解决是修改服务端的`/etc/ssh/sshd_config`，然后重启服务

```conf
UseDNS no
```

## 5. 压缩和解压缩命令

### 5.1. tar命令

#### 5.1.1. 参数解析

- `-c`: 建立压缩档案
- `-x`: 解压
- `-t`: 查看内容
- `-r`: 向压缩归档文件末尾追加文件
- `-u`: 更新原压缩包中的文件

这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。

- `-z`: 有gzip属性的 tar.gz
- `-j`: 有bz2属性的 tar.bz2
- `-J`: tar.xz
- `-Z`: 有compress属性的 tar.Z
- `-v`: 显示所有过程
- `-O`: 将文件解开到标准输出

这几个根据需要在压缩或解压档案时可选的。

- `-f [tar_file_name]`: 必须，使用档案名字，多选项时，此参数为最后一个。
- `-C [dir_name]`: 解压到什么目录，压缩时的父目录

#### 5.1.2. 示例

```shell
tar -xzf xxx.tar.gz             # 解压tar.gz文件
tar -xzvf xxx.tar.gz -C tmp     # 解压tar.gz文件显示过程，解压到tmp目录
tar -czvf xxx.tar.gz -C tmp ./  # 将tmp目录下的所有文件压缩，以tmp为根目录
```

#### 5.1.3. 加密压缩

```shell
# 使用openssl命令进行加密压缩
tar -zvcf - -C xxx ./ | openssl des3 -salt -k "xxx" | dd of=xxx.des3
# 解压
dd if=xxx.des3 | openssl des3 -d -k 'xxx' | tar -zxf - -C xxx/
```

- <font color = "red">**openssl在1.1.0版本有修改导致不兼容，可以用下面的方法**</font>
```shell
在OpenSSL 1.1.0中，我们从MD5更改为SHA-256。我们这样做是作为整体改变的一部分，以摆脱现在不安全和破碎的MD5算法。如果您有旧文件，请使用“-md md5”标志对其进行解密
```

### 5.2. zip命令

#### 5.2.1. 压缩

压缩只会追加不会删除已经存在zip文件中的文件

- `-j`: 去除路径
- `-r`: 递归压缩所有文件
- `-m`: 打包后删除压缩的源文件

```shell
zip -r (filename.zip) (path)
# 去除路径
zip -jr (filename.zip) (path)
```

#### 5.2.2. 解压

```shell
unzip xxx.zip -d xxx
# 解压特定文件
unzip xxx.zip "b/c" -d xxx
# 查看文件列表
unzip -l xxx.zip
# 指定编码
unzip -O gbk (filename.zip) -d (path)
```

- 不加`-d`就解压到当前目录

## 6. e2label 修改卷标名称

分区为`ext2/ext3`类型使用

```shell
e2label /dev/(partition) "(name)"
```

## 7. readelf 查看二进制符号表

### 7.1. 查看符号表

- 查看有哪些二进制符号和对应的函数地址

```shell
# -s 符号表
# -W 符号名字显示全
readelf -Ws xxxx
```

### 7.2. 查看二进制依赖

- 会显示依赖的动态库路径

```shell
readelf -d xxx
```

## 8. objdump 导出汇编指令表

导出二进制的汇编指令表，如果没有strip掉符号，还能看到每个函数的汇编指令和对应地址

```shell
objdump -DS xxxx > xxx.dump
```

## 9. strace 查看系统调用

- `-t`: 打印时间
- `-tt`: 打印时间，精确到微秒
- `-T`: 打印系统调用占用时间
- `-f`: 打印进程号，可针对多进程程序
- `-v`: 参数打全，不省略
- `-o file`: 结果输出到文件
- `-p [PID]`: 挂载到一个进程

### 9.1. 查看单进程的系统调用，不重启进程

```shell
strace -p xxx
```

## 10. tree 查看文件树

- `-L n`: 目录深度

## 11. sshfs 挂载远程目录

```shell
# 和mount一样，将远程的/aaa挂载到本地~/xxx
sshfs xxx@xxx.xxx.xxx.xxx:/aaa ~/xxx
```

## 12. cmake 跨平台编译命令

CMakeLists.txt编写参见[CMakeLists.txt](/blogs/2019-06-03-makefile/#CMakeLists)

### 12.1 基本使用命令

```shell
# 指定build目录生成工程
cmake -DCMAKE_BUILD_TYPE:STRING=Debug -B build/
# vscode常用的编译命令
cmake --no-warn-unused-cli -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=TRUE -DCMAKE_BUILD_TYPE:STRING=Debug -H . -B build/ -G "Visual Studio 14 2015" -A win32
# 编译指定目标文件，会自动编译依赖，4线程编译
# 不指定target编译所有
cmake --build build/ --target xxx -j 4
# 清理工程，相当于make clean
cmake --build build/ --target clean -j 4
```

## 13. gdb c/c++单步调试工具

### 13.1 基本使用命令

```shell
start                       # 开始调试,停在第一行代码处,(gdb)start
l                           #list的缩写查看源代码,(gdb) l [number/function]
b <lines>                   # b: Breakpoint的简写，设置断点。(gdb) b 10
b <func>                    # b: Breakpoint的简写，设置断点。(gdb) b main
b filename:[line/function]  # b:在文件filename的某行或某个函数处设置断点
i breakpoints               # :info 的简写。(gdb)i breakpoints
d [bpNO]                    # d: Delete breakpoint的简写，删除指定编号的某个断点，或删除所有断点。断点编号从1开始递增。 (gdb)d 1
s                           # s: step执行一行源程序代码，如果此行代码中有函数调用，则进入该函数；(gdb) s
n                           # n: next执行一行源程序代码，此行代码中的函数调用也一并执行。(gdb) n
r                           # Run的简写，运行被调试的程序。如果此前没有下过断点，则执行完整个程序；如果有断点，则程序暂停在第一个可用断点处。(gdb) r
c                           # Continue的简写，继续执行被调试程序，直至下一个断点或程序结束。(gdb) c
finish                      # 函数结束
p [var]                     # Print的简写，显示指定变量（临时变量或全局变量 例如 int a）的值。(gdb) p a
display [var]               # display，设置想要跟踪的变量(例如 int a)。(gdb) display a
undisplay [varnum]          # undisplay取消对变量的跟踪，被跟踪变量用整型数标识。(gdb) undisplay 1
set args                    # 可指定运行时参数。(gdb)set args 10 20
show args                   # 查看运行时参数。
q                           # Quit的简写，退出GDB调试环境。(gdb) q
help [cmd]                  # GDB帮助命令，提供对GDB名种命令的解释说明。如果指定了“命令名称”参数，则显示该命令的详细说明；如果没有指定参数，则分类显示所有GDB命令，供用户进一步浏览和查询。(gdb)help
回车                        # 重复前面的命令，(gdb)回车
```

### 13.2. 可以看代码版本 cgdb

## 14. <span id = "tmux">tmux</span>

### 快捷键

- 工具特定命令前缀为`Ctrl + b`

**session操作**

- `tmux ls`: 查看session列表
- `prefix, d`: 离开当前session，但是session继续跑
- `tmux a -t <session-name>`: 重新回到一个session
- `tmux kill-session -t <session-name>`: 杀死一个session

**窗口操作**

- `prefix, c`: 创建新窗口
- `prefix, n`: 下一个窗口
- `prefix, p`: 上一个窗口
- `prefix, l`: 进入前一个操作的窗口
- `prefix, w`: 列出窗口进行选择
- `prefix, ,`: 重命名当前窗口
- `prefix, Shift + &`: 删除当前窗口

**窗格操作**

- `prefix, <方向键>`: 切换到相应窗格
- `prefix, Shift + "`: 纵向分屏
- `prefix, Shift + %`: 横向分屏
- `prefix, Shift + !`: 将当前窗格新起一个窗口存放
- `prefix, Ctrl + <方向键>`: 朝相应方向移动边界
- `prefix, x`: 关闭当前窗格
- `prefix, z`: 当前窗格放大（专注，切换窗格就恢复了）
- `prefix, ;`: 切换到上一个窗格
- `prefix, o`: 切换到下一个窗格
- `prefix, [`: 进入copy mode
- `prefix, Ctrl + o`: 顺时针切换各个窗格
- `prefix, m`: 标记窗格，配合下面命令交换窗格
- `prefix, :swapp`: 切换窗格

**copy mode**

- `Ctrl + s`: 向下查找，输入字符串回车后，使用n和N选择下一个或者上一个
- `Ctrl + r`: 向上查找

## 15. fzf

### 配置

1. 安装完成后，将下面配置加到`.zshrc`或者`.bashrc`中

```shell
# 40%屏显示非全屏，展示反向（输入框在上面，文件在下面正序），文件在右侧展示前50行
export FZF_DEFAULT_OPTS="--height 40% --layout=reverse --preview 'head -n 50 {} 2>/dev/null'"
```


## 16. <span id = "ctags">ctags</span>

### 16.1. C/C++

#### 基本选项

- `-I xxx`: 许多函数定义最后有一个`__THROW`类似的，ctags将解析出错，加上此选项会忽略xxx
- `--fields=+iaS`:
- `--extra=+q`
- `--c-kinds=+p`

#### 添加系统头文件支持

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

### 16.2. php支持

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

## 17. gcc

### 17.1. 选项解释

- `-s`: strip掉所有符号
- `-fsanitize=address`: 监听内存泄漏，需要同时加上`-lasan`并且保证已经安装libasan
- `-fno-omit-frame-pointer`
- `-Werror`: 所有warning当作error处理

## 18. firewalld 防火墙

### 18.1. 选项解释

- `--permanent`: 永久生效，加了这个选项，需要执行reload才能生效；不加，会立即生效，但是reload后会消失

### 18.2. 基本操作

```shell
# 重载防火墙规则
firewall-cmd --reload
# 重启防火墙服务
firewall-cmd --complete-reload
```

### 18.3. 区域操作

```shell
########## 查看区域 ##########
# 查看当前使用的区域，返回每个网卡对应的区域
firewall-cmd --get-active-zones
# 查看所有支持的区域
firewall-cmd --get-zones
# 查看网卡对应的区域
firewall-cmd --get-zone-of-interface=eth0
# 查看默认区域
firewall-cmd --get-default-zone
# 查看区域所有配置，不加zone返回默认区域
firewall-cmd --zone=public --list-all

########## 编辑区域 ##########
# 将一个接口加到public区域内，永久生效加上--permanent然后reload
firewall-cmd --zone=public --add-interface=eth0
# 设置默认区域，永久生效加上--permanent然后reload
firewall-cmd --get-default-zone
```

### 18.4. 服务操作

```shell
########## 查看服务 ##########
# 列举服务，不加zone列举默认区域
firewall-cmd --zone=work --list-services
# 查看支持的服务
firewall-cmd --get-services

########## 新增服务 ##########
# 允许smtp服务，永久生效加上--permanent然后reload
firewall-cmd --zone=work --add-service=smtp

########## 删除服务 ##########
# 删除smtp服务，永久生效加上--permanent然后reload
firewall-cmd --zone=work --remove-service=smtp
```

### 18.5. 端口操作

```shell
########## 查看服务 ##########
# 列举服务，不加zone列举默认区域
firewall-cmd --zone=work --list-ports

########## 新增服务 ##########
# 允许22端口，永久生效加上--permanent然后reload
firewall-cmd --zone=work --add-port=22/tcp

########## 删除服务 ##########
# 删除22端口，永久生效加上--permanent然后reload
firewall-cmd --zone=work --remove-port=22/tcp
```

### 18.6. 直接操作

- 这个操作和firewalld原生的不在一起，如果firewalld放开了，这里禁止了，无法访问
- 如果firewalld禁止了，这里放开了，同样无法访问

```shell
firewall-cmd --direct -add-rule ipv4 filter INPUT 0 -p tcp --dport 9000 -j accept
```

## 19. jq linux的json解析工具

### 1. 几种基本用法

```shell
# 检查key是否存在
=> echo '{"a":1}' | jq 'has("b")'
false
=> echo '{"a":1}' | jq 'has("a")'
true

# 取值
=> echo '{"a":1}' | jq '.'
1
=> echo '{"a":"1"}' | jq '.a'
"1"
```

## 20. iotop 磁盘占用统计工具

### 1. 几种基本用法

- 需要使用root权限
- 类似与top，不需要跟参数

**统计中快捷键**

- `o`: 仅显示占用io的进程
- `a`: 显示累计数据
- `shift + P`: 只显示进程，不显示线程，默认显示所有线程
- 使用左右方向键来选择排序的列

## 21. 图片操作工具 imagemagick

### 21.1. 图片转pdf

- 按照图片名字顺序生成pdf

```shell
convert *.jpg +compress foo.pdf
```

### 21.2. pdf转图片

- 执行了下面命令后，就会生成 image_name-0.jpg，image_name-1.jpg等等图片，数目由PDF页数决定。

```shell
convert pdf_name.pdf image_name.jpg
```

**额外的参数**

- `-resize 1800x`: 指定生成的像素大小，越大生成的图片越大，转化的时间越久
- `-density 150`: 参数指定密度，具体含义再查
- `-quality 100`: 指定生成图片的质量

```shell
convert -resize 1800x -density 150 -quality 100 pdf_name.pdf image_name.jpg
```

## 22. 模拟网络延迟 tc

- 局限就是只能对某一个网卡设置，不能设定规则来指针对ip和端口

### 22.1. 基本操作

```shell
########## 增加延迟 ##########
tc qdisc add dev  eth0 root netem delay 100ms 10ms
########## 删除规则 ##########
tc qdisc del dev eth0 root
########## 查询规则 ##########
tc -s qdisc ls dev eth0
```

## 23. ltrace 跟踪库函数调用

### 23.1 选项

- `-r`: 显示运行时间，`0.000123`
- `-t`: 和`-r`不能同时用，显示当前时间，`23:12:11`
- `-tt`: 和`-r`不能同时用，显示当前时间，`23:12:11.345678`
- `-ttt`: 和`-r`不能同时用，显示当前时间戳，`1645024757.123456`
- `-T`: 显示调用花费的时间，在函数调用返回值后面用`<>`包裹
- `-i`: 显示函数的地址
- `-c`: 显示统计数据，表格形式
- `-S`: 将系统调用显示出来，使用`SYS_`作为前缀
- `-p [PID]`: 挂载到一个进程

### 23.2 实例

```shell
# 监听一段时间的5623的函数调用次数，需要ctrl c停止监听后才显示结果
=> ltrace -p 5623 -c
^C% time     seconds  usecs/call     calls      function
------ ----------- ----------- --------- --------------------
 36.20    0.151489        2913        52 __errno_location
 29.99    0.125516        5229        24 epoll_wait
 24.01    0.100467        2954        34 clock_gettime
  5.33    0.022294        2786         8 pthread_mutex_lock
  4.47    0.018724        2340         8 pthread_mutex_unlock
------ ----------- ----------- --------- --------------------
100.00    0.418490                   126 total

# 显示系统调用并显示各个函数的调用耗时
＝> ltrace -t -S -T ./run
# 省略若干行
...
23:36:12 _ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev(0x5714c8aed8, 0, 0, 0x5714c79010) = 0x5714c8aee8 <0.001116>
23:36:12 _ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev(0x5714c8aeb8, 0x5714c8aee8, 0, 0x5714c79010) = 0x7abb140060 <0.001756>
23:36:12 _ZdlPvm(0x5714c8aeb0, 224, 0, 0x5714c79010)                 = 0x7abb140060 <0.001005>
23:36:12 _ZdlPvm(0x5714c8b110, 720, 1, 0x7fe51ecda0)                 = 0x7abb140060 <0.000966>
23:36:12 _ZdlPvm(0x5714c8b740, 16, 1, 0x5714c8b740)                  = 0x7abb140060 <0.000864>
23:36:12 _ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev(0x5714c8b668, 0, 0, 0x5714c79010) = 0x5714c8b678 <0.001675>
23:36:12 _ZdlPvm(0x5714c8b650, 232, 1, 0x7fe51ecfc0)                 = 0x7abb140060 <0.000936>
23:36:12 _ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev(0x5714c8b768, 0, 0, 0x5714c79010) = 0x5714c8b778 <0.000870>
23:36:12 _ZdlPvm(0x5714c8b760, 64, 1, 0x5714c79010)                  = 0x7abb140060 <0.000934>
23:36:12 memset(0x5714c8b7b0, '\0', 104)                             = 0x5714c8b7b0 <0.001023>
23:36:12 _ZdlPvm(0x5714c8b7b0, 104, 13, 0x5714c8b7e0)                = 0x7abb140060 <0.000877>
23:36:12 SYS_exit_group(0 <no return ...>
23:36:12 +++ exited (status 0) +++
```

## 24. dmidecode 获取硬件信息

### 24.1. 获取内存信息

- 可以获取到有几个插槽，每个插槽内存信息（包括频率、大小、电压、类型等）

```shell
=> sudo dmidecode -t memory
```

## 25. nc 网络链接工具

## 26. curl 网络请求工具

### 26.1. 下载文件

```shell
curl http://1.1.1.1/download/aaa.txt -o aaa.txt
```

# 四、小技巧

## 1. swap临时空间（可用于编译时内存不足问题）

```shell
# 查看内存文件
sudo swapon --show

# 一般内存不够了，可以用这种方式增加交换内存。
dd if=/dev/zero of=/var/swapfile1 bs=1024 count=6000000
mkswap /var/swapfile1
swapon /var/swapfile1

# 不需要交换内存的时候，取消磁盘占用
swapoff /var/swapfile1
rm -f /var/swapfile1
```

## 2. <span id="openssl_diy">自颁发证书（不受信）</span>

在开发过程中，https请求需要配置ssl证书，这个证书需要到部分供应商申请（阿里云、腾讯云等），自然不是免费的。开发过程中并不想这么麻烦，可以使用shell中的openssl进行自颁发证书。

1. 颁发证书需要先生成一个颁发机构，也就是CA

```shell
# 新建目录防止混乱
mkdir -p ssl_diy/private && cd ssl_diy
# 生成ca的私钥
openssl genrsa -out private/cakey.pem 2048
# 根据key生成CA的证书文件
openssl req -new -x509 -key private/cakey.pem -out cacert.pem
# 将有一些配置需要填写，可以随意填写，因为是自颁发
# Country Name (2 letter code) [AU]: 国家名，自然CN
# State or Province Name (full name) [Some-State]: 省份名称，比如Hunan
# Locality Name (eg, city) []: 城市名称，比如changsha
# Organization Name (eg, company) [Internet Widgits Pty Ltd]: 公司名称
# Organizational Unit Name (eg, section) []: 部门名称
# Common Name (eg, YOUR name) []: 颁发者的名字
# Email Address []: 邮件地址
```

2. 生成证书

```shell
# 同样生成私钥key
openssl genrsa -out domain.key 2048
# 根据key生成证书文件
openssl req -new -key domain.key -out domain.csr
# 将有一些配置需要填写，可以随意填写，因为是自颁发
# Country Name (2 letter code) [AU]: 国家名，自然CN
# State or Province Name (full name) [Some-State]: 省份名称，比如Hunan
# Locality Name (eg, city) []: 城市名称，比如changsha
# Organization Name (eg, company) [Internet Widgits Pty Ltd]: 公司名称
# Organizational Unit Name (eg, section) []: 部门名称
# Common Name (eg, server FQDN or YOUR name) []: 填写网站的域名
# Email Address []: 邮件地址
```

3. 使用ca给证书签名

```shell
# 签名前需要有几个准备文件
mkdir -p demoCA/newcerts
touch demoCA/index.txt
echo 01 > demoCA/serial
# 签名，可以自定义有效天数-days
openssl ca -in domain.csr -cert cacert.pem -keyfile private/cakey.pem -days 365 -out domain.crt
```

4. 将生成的domain.crt和domain.key配置到服务器即可

## 3. 快捷键

- `Ctrl + a` / `home`: 移到行首
- `Ctrl + e` / `end`: 移到行首
- `Ctrl + r`: 历史命令搜索，不支持模糊搜索，只支持连续字符
- `Ctrl + k`: 清空光标后面的字符
- `Ctrl + u`: 清空光标前面的字符

## 4. 安装软件

### 4.1. 二进制安装

- 一般二进制都会自带依赖，可以直接使用
- linux根目录有一个opt目录，大致是让用户自定义软件放在这里
- /usr/local/bin目录下放的是用户自定义应用，所以移过去后可以软链过去

```shell
sudo mv xxx /opt/xxx
sudo ln -s /opt/xxx/bin/xxx /usr/local/bin/xxx
```

## 5. 代码扫描

- github搜索shellcheck软件，根据系统安装
- vscode搜索shellcheck插件，安装
- 不在PATH中则手动配置路径即可完成扫描

## 6. 软件运行

### 6.1. 临时加载库so

- 只想要当前运行加载so，可以执行下面命令

```shell
export LD_LIBRARY_PATH="xxx"
```

## 7. 脚本技巧

### 7.1. 获取脚本所在目录

```shell
#!/bin/bash
# $0是脚本目录，dirname获取目录
scriptDir="$(dirname "$0")"
```

## 8. 有用的软件

### 8.1. 登陆显示系统信息

安装`landscape-common`，可以在登陆时显示下面的信息

```
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.6.0-25-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 15 Jan 2021 10:08:49 AM CST

  System load:  0.25               Processes:              356
  Usage of /:   28.5% of 77.43GB   Users logged in:        0
  Memory usage: 10%                IPv4 address for ens18: 192.168.0.1
  Swap usage:   0%

5 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Tue Jan 5 10:13:40 2021 from 192.168.0.2
```

### 8.2. x11vnc linux远程桌面

**设置密码**

- 默认储存到`/home/<username>/.vnc/passwd`

```shell
x11vnc -storepasswd
```

**设置开机启动**

- 写一个service文件到`/usr/lib/systemd/system/x11vnc.service`
- 要修改passwd的路径

```ini
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/<username>/.vnc/passwd -rfbport 5900 -shared -capslock

[Install]
WantedBy=multi-user.target
```

- `systemctl enable x11vnc`，加到开机启动项中
- `service x11vnc start`，当前开启服务

**无显示器模拟显示器**

- 在没有显示器的情况下x11vnc特别慢，插上显示器就快了，因为没有显示器显卡不工作
- 安装`xserver-xorg-video-dummy`模拟显示器
- 新增配置文件`/usr/share/X11/xorg.conf.d/xorg.conf`

```ini
Section "Device"
    Identifier  "Configured Video Device"
    Driver      "dummy"
EndSection

Section "Monitor"
    Identifier  "Configured Monitor"
    HorizSync 31.5-48.5
    VertRefresh 50-70
    Modeline "1920x1080"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
    VideoRam 256000
EndSection

Section "Screen"
    Identifier  "Default Screen"
    Monitor     "Configured Monitor"
    Device      "Configured Video Device"
    DefaultDepth 24
    SubSection "Display"
        Depth 24
        Modes "1920x1080"
    EndSubSection
EndSection
```

- 重启设备就好了

**sddm管理桌面x11vnc起不来**

- 可能需要单独写一个脚本处理，上面service的exec就指定这个脚本的目录即可
- sddm使用单独的auth模式

```shell
#!/bin/bash

while [ -z "$(ls -t /run/sddm)" ]; do
    sleep 1
done
authFile="$(ls -t /run/sddm | head -n 1)"

/usr/bin/x11vnc -auth "/run/sddm/$authFile" -forever -loop -noxdamage -repeat -rfbauth "/home/xxx/.vnc/passwd" -rfbport 5900 -shared -capslock
```

**gdm3管理桌面x11vnc起不来**

- 同上，不过脚本变成下面这样
- 尝试的出现一个问题，当使用gdm3启动kde时，在桌面登陆前，没有`/run/user/1000/gdm/Xauthority`，需要先登录再启动
- 但是存在一个`/run/user/129/gdm/Xauthority`，这个可以打开登陆界面，登陆完这个就只有黑屏，需要使用上面的配置，并且`-dislay`需要设置成`:1`

```shell
#!/bin/bash

auth_file="/run/user/1000/gdm/Xauthority"

while [ ! -f "$auth_file" ]; do
    sleep 1
done

/usr/bin/x11vnc -auth "$auth_file" -forever -loop -noxdamage -repeat -rfbauth "/home/xxx/.vnc/passwd" -rfbport 5900 -shared -capslock
```

## 9. 查看电量

```shell
cat /sys/class/power_supply/battery/capacity
```

## 10. 当前运行命令转后台

1. `Ctrl + z`: 暂停当前程序
2. `bg`: 后台继续运行暂停的程序

**其他命令**

- `fg`: 后台运行程序转前台
- `jobs`: 查看所有程序

## 12. shell脚本单例

```shell
## 进程单例判断
is_single_running() {
    local lockfile=$1

    exec 100> "$lockfile"
    if /usr/bin/flock -xw 3 100
    then
        return 0
    else
        return 1
    fi
}

## 用法
if ! is_single_running /tmp/xxx; then
    echo "has been running"
    exit 1
fi
...
```

## 13. 挂载windows共享文件夹

- `uid`和`gid`需要根据需要设置成对应的id，为挂在过来的所属用户和用户组

```shell
# 查看windows共享文件夹
smbclient -U User -L //xxx.xxx.xxx.xxx/
# 挂载
mount -t cifs -o user=share,rw,uid=0,gid=0 //192.168.1.120/share /root/share
```

# 踩坑记

## 1. ssh用rsakey无法免密登陆

- sshd要求使用rsakey需要有以下权限限制
    - `/home/xxx`需要700权限，不允许用户组编辑
    - `/home/xxx/.ssh`需要700权限
    - `/home/xxx/authorized_keys`需要600权限

