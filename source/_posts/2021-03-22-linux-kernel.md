---
title: linux kernel源码学习记录
date: 2021-03-22 16:55:36
tags: [Linux]
categories: [Program, C/C++]
---

# 一、前言

本文为研究linux kernel源码所记录的一些笔记

源码下载路径
```
https://mirrors.edge.kernel.org/pub/linux/kernel/
```

由于篇幅问题，后续更新放在 [linux内核源码分析记录](/bookPages/docs/linux-kernel/)

# 二、linux启动过程

## 1. 从引导加载程序内核

### 1.1. cpu上电

1. 主板通电后，会启动cpu
2. cpu启动复位后，开始在实模式下工作
  - 所有x86兼容处理器均支持实模式
3. 启动后开始从地址`0xfffffff0`执行第一条指令，这个地址被放置了BIOS的入口
4. 但是实模式下只有16位寄存器，不能索引到上面的地址。其实这个地址被映射到了rom中
5. ROM中存放各个硬件厂商定制的BIOS或者UEFI的启动代码，用于找到硬盘并引导系统启动

### 1.2. bios启动

**UEFI和BIOS的区别**

BIOS启动流程：

- 系统开机
- 上电自检（Power On Self Test 或 POST）。POST过后初始化用于启动的硬件（磁盘、键盘控制器等）
- BIOS会运行BIOS磁盘启动顺序中第一个磁盘的首440bytes（MBR启动代码区域）内的代码。
- 启动引导代码从BIOS获得控制权，然后引导启动下一阶段的代码（如果有的话）（一般是系统的启动引导代码）。
- 再次被启动的代码（二阶段代码）（即启动引导）会查阅支持和配置文件。根据配置文件中的信息，启动引导程序会将内核和initramfs文件载入系统的RAM中，然后开始启动内核。

UEFI启动流程：

- 系统开机
- 上电自检（Power On Self Test 或 POST）。UEFI 固件被加载，并由它初始化启动要用的硬件。
- 固件读取其引导管理器以确定从何处（比如，从哪个硬盘及分区）加载哪个 UEFI 应用。
- 固件按照引导管理器中的启动项目，加载UEFI 应用。
- 已启动的 UEFI 应用还可以启动其他应用（对应于 UEFI shell 或 rEFInd 之类的引导管理器的情况）或者启动内核及initramfs（对应于GRUB之类引导器的情况），这取决于 UEFI 应用的配置。

**MBT和GPT**

MBR

1. bios在初始化和检查硬件之后，需要找一个可引导设备
   1. 初始可引导设备列表存在bios配置中，根据顺序一个一个找
2. 对于硬盘，引导扇区在第一个扇区（512字节）的头446字节，并且引导扇区最后必须是`0x55`和`0xaa`，这两个字节可以称为魔术字节，如果bios看到这两个字节，认为这个设备是可引导设备，这个也是MBR硬盘的第一个扇区的构成
3. 在实模式下，内存的组成如下，所以我们写bootloader程序需要加载到`0x7c00`，如果需要操作显示，需要写入到`0xa0000 ~ 0xbffff`

```
0x00000000 - 0x000003FF - Real Mode Interrupt Vector Table
0x00000400 - 0x000004FF - BIOS Data Area
0x00000500 - 0x00007BFF - Unused
0x00007C00 - 0x00007DFF - Our Bootloader
0x00007E00 - 0x0009FFFF - Unused
0x000A0000 - 0x000BFFFF - Video RAM (VRAM) Memory
0x000B0000 - 0x000B7777 - Monochrome Video Memory
0x000B8000 - 0x000BFFFF - Color Video Memory
0x000C0000 - 0x000C7FFF - Video ROM BIOS
0x000C8000 - 0x000EFFFF - BIOS Shadow Area
0x000F0000 - 0x000FFFFF - System BIOS
```

- linux kernel启动函数（main函数）并不是常规的main，是`init/main.c`里面的`start_kernel()`函数

# 三、数据结构

## 1. 公共机制

### 1.1. `container_of` 根据数据结构节点找value

[container_of](/bookPages/docs/linux-kernel/data-structures/container_of/)

## 2. rbtree

[rbtree](/bookPages/docs/linux-kernel/data-structures/rbtree/)

## 3. rcu 读拷贝更新

[rcu](/bookPages/docs/linux-kernel/data-structures/rcu/)

# 四、系统调用

## 1. 网络相关

### 1.1. epoll

[epoll](/bookPages/docs/linux-kernel/net/epoll/)

### 1.2. bind 绑定地址到socket

#### 1) 接口定义

```cpp
// net/socket.c
SYSCALL_DEFINE3(bind, int, fd, struct sockaddr __user *, umyaddr, int, addrlen)
{
	return __sys_bind(fd, umyaddr, addrlen);
}
```

### 1.3. unix套接字

源码主要看`net/unix/af_unix.c`

- unix套接字仅支持`SOCK_STREAM`、`SOCK_RAW`、`SOCK_DGRAM`、`SOCK_SEQPACKET`这几种type，源码看`unix_create()`
- 使用unix套接字，protocol参数必须为`0`，原因查看`unix_create()`的第一个判断

## 2. 文件相关

### 2.1. 监听文件变化 inotify

- inotify是linux内核给用户态提供的一个监听文件变化的接口

```cpp
class FileMonitor {
public:
    FileMonitor(const std::string &path) : m_filePath(path) {
        std::promise<void> p;
        m_monitorFuture = std::async(std::launch::async, [&p, path, this]() {
            p.set_value();
            const std::string tag = "file monitor loop";
            LOGI(WHAT("{} begin", tag));

            auto fd = inotify_init();
            if (fd < 0) {
                LOGE(WHAT("{}, inotify_init failed", tag),
                     REASON("ec: {}", std::to_string(std::error_code(errno, std::system_category()))),
                     WILL("exit thread"));
                return;
            }

            {
                std::lock_guard<std::mutex> lock(m_fdMutex);
                m_monitorFD = fd;
            }

            // 添加文件监听的函数
            auto addFileWatch = [&tag, this]() {
                int watchFD = -1;
                while (true) {
                    // 监听所有事件，除了访问，打开和关闭，主要监听修改删除
                    watchFD = inotify_add_watch(m_monitorFD, m_filePath.c_str(),
                                                IN_ALL_EVENTS ^ (IN_ACCESS | IN_OPEN | IN_CLOSE));
                    if (watchFD > 0) {
                        break;
                    }
                    LOGW(WHAT("{}, inotify_add_watch failed", tag),
                         REASON("ec: {}", std::to_string(std::error_code(errno, std::system_category()))),
                         WILL("retry after 1 second"));
                    std::this_thread::sleep_for(std::chrono::seconds(1));
                }

                // 这里赋值成成员变量，用于析构时可以退出
                {
                    std::lock_guard<std::mutex> lock(m_fdMutex);
                    m_watchFD = watchFD;
                }
            };
            addFileWatch();
            while (true) {
                char buffer[(sizeof(struct inotify_event) + NAME_MAX + 1)] = {0};
                // 阻塞读取event事件
                auto len = read(fd, buffer, sizeof(struct inotify_event));
                // 监听的fd被关掉，也就是外部调用了inotify_rm_watch。当前仅析构函数会调用，直接退出
                auto err = errno;
                if (len < 0 && err == EBADF) {
                    LOGI(WHAT("{}, stop monitor file {}", tag, m_filePath));
                    break;
                }
                /* Some signal, likely ^Z/fg's STOP and CONT interrupted the inotify read, retry */
                if (len < 0 && err != EINTR && err != EAGAIN) {
                    LOGW(WHAT("{}, read get err", tag),
                         REASON("get error {}", std::to_string(std::error_code(err, std::system_category()))),
                         WILL("retry after 1 second"));
                    std::this_thread::sleep_for(std::chrono::seconds(1));
                    continue;
                }

                if (len < (int)sizeof(struct inotify_event)) {
                    LOGW(WHAT("{}, read len error", tag), REASON("len {}, need {}", len, sizeof(struct inotify_event)),
                         WILL("retry after 1 second"));
                    std::this_thread::sleep_for(std::chrono::seconds(1));
                    continue;
                }

                int index = 0;
                while (index < len) {
                    auto event = reinterpret_cast<struct inotify_event *>(buffer + index);
                    index += sizeof(struct inotify_event) + event->len;
                    if (event->wd != m_watchFD) {
                        continue;
                    }
                    if (event->mask & (IN_IGNORED)) {
                        continue;
                    }

                    // 这个文件被删除或重命名，需要重新进行添加
                    if (event->mask & (IN_DELETE_SELF | IN_MOVE_SELF)) {
                        LOGI(WHAT("{}, file {} was renamed or deleted, retry open", tag, m_filePath));
                        {
                            std::lock_guard<std::mutex> lock(m_fdMutex);
                            inotify_rm_watch(fd, m_watchFD);
                            m_watchFD = -1;
                        }
                        addFileWatch();
                        // 到这里说明文件重新创建，同样认为是修改，调用修改回调
                    }
                    // 文件发生变化
                    LOGI(WHAT("file {} change", m_filePath));
                }
            }

            {
                std::lock_guard<std::mutex> lock(m_fdMutex);
                if (m_monitorFD >= 0) {
                    if (m_watchFD >= 0) {
                        inotify_rm_watch(m_monitorFD, m_watchFD);
                        m_watchFD = -1;
                    }
                    close(m_monitorFD);
                    m_monitorFD = -1;
                }
            }

            LOGI(WHAT("{} end", tag));
        });
        p.get_future().wait();
    }

    ~FileMonitor() {
        do {
            std::lock_guard<std::mutex> lock(m_fdMutex);
            if (m_monitorFD >= 0) {
                if (m_watchFD >= 0) {
                    inotify_rm_watch(m_monitorFD, m_watchFD);
                    m_watchFD = -1;
                }
                close(m_monitorFD);
                m_monitorFD = -1;
            }
            // 可能还没打开就关闭了，这里确保一下future是正常的，防止死锁
        } while (m_monitorFuture.wait_for(std::chrono::milliseconds(1)) != std::future_status::ready);
    }

    void registerMonitor(std::function<void()> callback) {
        std::lock_guard<std::mutex> lockGuard(m_callbackMutex);
        m_monitors.emplace_back(callback);
    }

private:
    std::string m_filePath;
    std::future<void> m_monitorFuture;
    std::mutex m_fdMutex;
    int m_monitorFD = -1;  // inotify的句柄
    int m_watchFD = -1;    // 被添加的文件句柄
    std::mutex m_callbackMutex;
    std::vector<std::function<void()>> m_monitors;
};
```

# 五、底层的几个机制

## 1. 惊群现象和处理

参考 [深入浅出 Linux 惊群：现象、原因和解决方案](https://zhuanlan.zhihu.com/p/385410196)

- 在linux的2.6.x已经解决，底层仅会唤起一个进程进行处理
