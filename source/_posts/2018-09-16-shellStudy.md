---
title: shell学习笔记
date: 2018-09-16 13:23:55
tags: [Linux]
categories: [Program, Shell]
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

#### 文件遍历

```shell
# 此行代表以换行符为分割，默认为空格、\t和\n
IFS=$'\n'
# 将test.txt的内容按行分割到value输出
for value in $(cat test.txt)
do
    echo "$value"
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
echo ${string/#abc/bb} # #以什么开头来匹配，根php中的^有点像
echo ${string/%41/bb}  # %以什么结尾来匹配，根php中的$有点像
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

### 4.1. 语法

    watch(选项)(参数)

### 4.2. 选项

- -n：指定指令执行的间隔时间（秒）；
- -d：高亮显示指令输出信息不同之处；
- -t：不显示标题。

### 4.3. 参数
指令：需要周期性执行的指令。

### 4.4. 实例

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

## 6. sort 排序

### 6.1. 去重

```shell
    # 将test.txt中排序并去重，但是只输出到控制台，可以用>写到文件
    cat test.txt | sort -u
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

## 10. date 时间转换工具

```shell
# 时间戳转正常时间格式
=> date -d @1587536520
2020年 04月 22日 星期三 14:22:00 CST

# 时间字符串转时间戳
=> date -d "20200422" +%s
1587536520

# 时间格式化
=> date +'%Y-%m-%d %A %H:%M:%S'
2020-05-08 Friday 19:12:22
```

## 11. dd 占用空间

### 11.1. 实例

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

### 13.1. 实例

```shell
# -I插入到前面，-A插入到后面
sudo iptables -I INPUT -i eth0 -s 200.200.87.48 -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

## 14. ls 列举目录

### 13.1. 选项

- `-l`: 列表显示详细信息
- `-h`: 目录大小加上单位
- `-t`: 按照时间排序
- `-S`: 按照大小排序
- `-r`: 反向排序

## 15. netstat 网络状态查看

```shell
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

## 16. sed 文件查找替换打印

- sed命令有点复杂，一直不会用，但是很强大

### 16.1. 选项

- `-i[suffix]`: 替换文件内容，如果定义了suffix，会备份一份到`xxxsuffix`

### 16.2. pattern

就是sed命令的字符串段，上面是选项

- `s/aaa/bbb/`: 将aaa换成bbb

### 16.3. 添加

**添加一行**

```shell
sed -i "/^start()/a\
	if ! "${script}" check "${proc_name}"; then exit 0; fi
" "${file_name}"
```

**添加多行**

```shell
sed -i "/^start()/a\
	if ! \"${script}\" check \"${proc_name}\"; then\n\
		/bin/run.sh \"${file}\" &\n\ 
		exit 0\n\ 
	fi
" "${file_name}"
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

# 三、工具命令

## 1. 文件夹目录大小 du

当前目录下所有子文件夹和文件的大小

```shell
    du ./ -h --max-depth=1
```

## 2. 抓包工具 tcpdump

### 2.1. 选项

```shell
    -c: 抓取的包的数量
    -i: 要监听的网口，不给默认为第一个网口
    -n: 对地址以数字方式显式，否则显示为主机名，也就是说-n选项不做主机名解析
    -nn: 除了-n的作用外，还把端口显示为数值，否则显示端口服务名
    -w: 保存的文件，cap结尾，用于wirshark分析
```

### 2.2. 示例

```shell
    # 抓5个192.168.100.62到本机eth0的icmp（ping）包
    tcpdump -c 5 -nn -i eth0 icmp and src 192.168.100.62
    # 抓取10个本机ens33的目的端口为22的tcp包
    tcpdump -c 10 -nn -i ens33 tcp dst port 22
```

## 3. 网络嗅探 nmap

[nmap使用](https://baike.baidu.com/item/nmap/1400075?fr=aladdin)

## 4. ssh生成公钥和私钥

```shell
    ssh-keygen -t rsa -C "xxx@xxx.com"
```

## 5. 压缩和解压缩命令

### 5.1. tar命令

#### 5.1.1. 参数解析

```shell
    -c: 建立压缩档案
    -x：解压
    -t：查看内容
    -r：向压缩归档文件末尾追加文件
    -u：更新原压缩包中的文件
```

这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。

```shell
    -z：有gzip属性的 tar.gz
    -j：有bz2属性的 tar.bz2
    -J：tar.xz
    -Z：有compress属性的 tar.Z
    -v：显示所有过程
    -O：将文件解开到标准输出
```

这几个根据需要在压缩或解压档案时可选的。

```
    -f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。
```

参数-f是必须的

```shell
    -C: 解压到什么目录，压缩时的父目录
```

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

### 5.2. zip格式

#### 5.2.1. 压缩

压缩目录和目录下所有文件

```shell
zip -r (filename.zip) (path)
```

#### 5.2.2. 解压

```shell
unzip (filename.zip) -d (path)
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

一般针对编译生成的二进制文件，可以用此方式导出二进制符号表，查看有哪些二进制符号和对应的函数地址

```shell
readelf -s xxxx > xxx.elf
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

### 9.1. 查看单进程的系统调用，不重启进程

```shell
strace -p xxx
```

## 10. tree 查看文件树

- `-L n`: 目录深度

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

**x11vnc服务启动无法连接**

- gdm3的桌面管理器没有使用xserver，所以无法使用x11vnc，见 [XServer基本概念 + x11vnc配置远程桌面](https://blog.csdn.net/lovewangtaotao/article/details/102907540)
- 切换为lightdm就好了

```shell
sudo dpkg-reconfigure lightdm
```

**sddm管理桌面x11vnc起不来**

- 可能需要单独写一个脚本处理，上面service的exec就指定这个脚本的目录即可
- sddm使用单独的auth模式

```shell
#!/bin/bash

while [ -z "$(ls -t /run/sddm)" ]; do
    sleep 1
done
authFile="$(ls -t /run/sddm | head -n 1)"

/usr/bin/x11vnc -auth "/run/sddm/$authFile" -forever -loop -noxdamage -repeat -rfbauth "/home/wangyubo/.vnc/passwd" -rfbport 5900 -shared -capslock
```

## 9. 查看电量

```shell
cat /sys/class/power_supply/battery/capacity
```

## 10. 当前运行命令转后台

1. `Ctrl + z`: 暂停当前程序
2. `bg`: 后台继续运行暂停的程序

- `fg`: 后台运行程序转前台
- `jobs`: 查看所有程序

# 踩坑记

## 1. ssh用rsakey无法免密登陆

- sshd要求使用rsakey需要有以下权限限制
    - `/home/xxx`需要700权限，不允许用户组编辑
    - `/home/xxx/.ssh`需要700权限
    - `/home/xxx/authorized_keys`需要600权限

