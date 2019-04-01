---
title: 操作系统笔记（C语言）
date: 2018-12-04 16:57:43
tags: 
categories: [Program, C/C++]
---

操作系统课程设计总结出来的笔记，源码见: https://github.com/Githubwyb/operation_system_homework

# 环境

- 操作系统: Linux
- 编译器: make

# 线程

## 创建线程

- 创建函数

```C
    #include <pthread.h>

    /*
     * @description 线程创建函数
     * @param tidp 线程标识符
     * @param attr 线程属性指针
     * @param start_rtn 线程执行函数(void *fun(void *))
     * @param arg 线程执行函数的参数
     * @return 0，创建成功；其他，错误码
     */
    int pthread_create(pthread_t *tidp,
                        const pthread_attr_t *attr,
                        (void*)(*start_rtn)(void*),
                        void *arg);
```

- 等待线程结束

```C
    #include <pthread.h>

    /*
     * @description 等待线程结束函数
     * @param thread 线程标识符
     * @param retval 获取线程结束返回值
     * @return 0，创建成功；其他，错误码
     */
    int pthread_join(pthread_t thread,
                        void **retval);
```

- 示例

```C
    #include <pthread.h>

    //线程执行函数
    void *threadHandler(void *param) {
        while (1) {
            ...
        }
    }

    int main(void) {
        pthread_t threadID;

        //创建线程
        int code = pthread_create(&threadID, NULL, threadHandler, NULL);
        ...
        //等待线程结束
        code = pthread_join(threadID, NULL);
        ...
    }
```

## 线程锁

- 上锁函数，被占用将阻塞线程

```C
    #include <pthread.h>

    /*
     * @description 申请锁
     * @param mutex 线程互斥锁
     * @return 0，创建成功；其他，错误码
     */
    int pthread_mutex_lock(pthread_mutex_t *mutex);
```

- 解锁函数

```C
    #include <pthread.h>

    /*
     * @description 解锁
     * @param mutex 线程互斥锁
     * @return 0，创建成功；其他，错误码
     */
    int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

- 示例

```C
    #include <pthread.h>

    pthread_mutex_t mutex;
    //线程执行函数
    void *threadHandler(void *param) {
        while (1) {
            ...
            //申请锁
            pthread_mutex_lock(&mutex);
            ...
            //释放锁
            pthread_mutex_unlock(&mutex);
            ...
        }
    }
```

## 原子操作（GCC）

同一个进程中，原子操作是不可被线程间抢占的。一个线程中的原子操作可以实现同步，加快线程间的协调作用，进行无锁化编程。

```C
    type __sync_fetch_and_add (type *ptr, type value);  //获取值后加上value
    type __sync_fetch_and_sub (type *ptr, type value);
    type __sync_fetch_and_or (type *ptr, type value);
    type __sync_fetch_and_and (type *ptr, type value);
    type __sync_fetch_and_xor (type *ptr, type value);
    type __sync_fetch_and_nand (type *ptr, type value);
    type __sync_add_and_fetch (type *ptr, type value);
    type __sync_sub_and_fetch (type *ptr, type value);
    type __sync_or_and_fetch (type *ptr, type value);
    type __sync_and_and_fetch (type *ptr, type value);
    type __sync_xor_and_fetch (type *ptr, type value);
    type __sync_nand_and_fetch (type *ptr, type value);
```

## 线程相关知识

- 线程内部创建变量是不共享的
- 全局变量线程内部共享

# 进程

## 创建进程

- 创建函数

```C
    #include <sys/types.h>
    #include <unistd.h>

    /*
     * @description 创建一个进程
     * @return pid_t实质是int，包含在<sys/types.h>中，子进程返回0；父进程返回子进程号；-1为错误
     */
    pid_t fork(void);
```

- 等待线程结束

当有子进程结束或僵尸进程时，立刻返回第一个结束的子进程ID。如果有子进程在运行，阻塞父进程。如果没有子进程在运行，返回-1。

```C
    #include <sys/wait.h>
    #include <sys/types.h>
    #include <unistd.h>

    /*
     * @description 等待进程结束函数
     * @param status 获取进程程结束状态值
     * @return 第一个结束的子进程进程号；-1，没有子进程在运行
     */
    pid_t wait(int *status);

    //子进程的结束状态返回后存于status，底下有几个宏可判别结束情况
    WIFEXITED(status);      //如果子进程正常结束则为非0值。
    WEXITSTATUS(status);    //取得子进程exit()返回的结束代码，一般会先用WIFEXITED来判断是否正常结束才能使用此宏。
    WIFSIGNALED(status);    //如果子进程是因为信号而结束则此宏值为真
    WTERMSIG(status);       //取得子进程因信号而中止的信号代码，一般会先用WIFSIGNALED来判断后才使用此宏。
    WIFSTOPPED(status);     //如果子进程处于暂停执行情况则此宏值为真。一般只有使用WUNTRACED 时才会有此情况。
    WSTOPSIG(status);       //取得引发子进程暂停的信号代码，一般会先用WIFSTOPPED来判断后才使用此宏。
```

- 示例

```C
    #include <sys/wait.h>
    #include <sys/types.h>
    #include <unistd.h>

    int main(void) {
        ...
        //创建进程
        pid_t processID = fork();
        if (processID == 0) {
            //子进程执行代码
            ...
        } else if (processID > 0){
            //父进程执行代码
            ...
            //等待子进程结束
            int status = 0;
            pid_t waitProcessID = wait(&status);
            if (waitProcessID == processID) {
                if (WIFEXITED(status) != 0) {
                    //子进程正常结束
                    if (WEXITSTATUS(status) != 0) {
                        LOG_DEBUG("the return code is %d.", WEXITSTATUS(status));
                    }
                }
                else {
                    //子进程非正常结束
                    LOG_DEBUG("the child process %d exit abnormally.", waitProcessID);
                }
                //子进程结束执行代码
                ...
            } else if (processID == -1){
                //没有子进程在运行
                ...
            } else {
                //其他子进程结束代码
            }
        } else {
            //创建错误代码
        }
        ...
    }
```

## 共享内存

进程相互之间的变量包括指针指向的地址都是不共享的，进程间通信需要使用共享内存。

### <span id="ftok">获取ID（ftok）</span>

```C
    #include <sys/ipc.h>
    #include <sys/types.h>

    /*
     * @description 获取一个key值用于共享内存或消息队列
     * @param fname 指定一个文件名
     * @param id 子序号
     * @return 共享内存ID；-1，错误，原因存于error中
     */
    key_t ftok(const char *fname, int id);
```

### 获取或创建共享内存（shmget）

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>

    /*
     * @description 获取或者创建共享内存
     * @param key 共享内存ID，一般为ftok获取的ID
     * @param size 共享内存大小
     * @param shmflg 标示属性，使用时需要与IPC对象存取权限（如0600）进行|运算来确定信号量集的存取权限
     * @return 共享内存ID；-1，错误，原因存于error中
     */
    int shmget(key_t key, size_t size, int shmflg)

    errno               //错误码

    #include <string.h>
    
    strerror(errno)     //错误信息字符串
```

- 函数传入值对应的操作

| key | size | shmflg                                | 描述                                                       |
| --- | ---- | ------------------------------------- | ---------------------------------------------------------- |
| x   | x    | IPC_CREAT $\mid$ 0600                 | 建立新的共享内存对象，不存在与key相同的则创建，存在返回key |
| x   | x    | IPC_CREAT $\mid$ IPC_EXCL $\mid$ 0600 | 建立新的共享内存对象，不存在与key相同的则创建，存在报错    |
| x   | 0    | 0                                     | 获取共享内存                                               |

- 错误代码

```shell
    EINVAL：参数size小于SHMMIN或大于SHMMAX
    EEXIST：预建立key所指的共享内存，但已经存在
    EIDRM：参数key所指的共享内存已经删除
    ENOSPC：超过了系统允许建立的共享内存的最大值(SHMALL)
    ENOENT：参数key所指的共享内存不存在，而参数shmflg未设IPC_CREAT位
    EACCES：没有权限
    ENOMEM：核心内存不足
```

### 链接共享内存到进程中（shmat）

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>

    /*
     * @description 链接共享内存到当前进程
     * @param shmid 共享内存标识符
     * @param shmaddr 指定链接为内存的哪一块地址，NULL让操作系统自己选择
     * @param shmflg SHM_RDONLY，为只读模式；其他为读写模式
     * @return 共享内存地址
     */
    void *shmat(int shmid, const void *shmaddr, int shmflg)

    errno               //错误码
    strerror(errno)     //错误信息字符串
```

- 错误代码

```shell
    EACCES：无权限以指定方式连接共享内存
    EINVAL：无效的参数shmid或shmaddr
    ENOMEM：核心内存不足
```

### 取消链接共享内存到进程中（shmdt）

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>

    /*
     * @description 链接共享内存到当前进程
     * @param shmaddr 共享内存地址
     * @return 0，成功；-1，错误，原因存于error中
     */
    int shmdt(const void *shmaddr)

    errno               //错误码
    strerror(errno)     //错误信息字符串
```

- 错误代码

```shell
    EINVAL：无效的参数shmaddr
```

### 共享内存管理

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>

    /*
     * @description 链接共享内存到当前进程
     * @param shmid 共享内存标识符
     * @param cmd 操作命令
     * @param buf 管理结构体
     * @return 0，成功；-1，错误，原因存于error中
     */
    int shmctl(int shmid, int cmd, struct shmid_ds *buf)

    errno               //错误码
    strerror(errno)     //错误信息字符串
```

- cmd

```
    IPC_STAT：得到共享内存的状态，把共享内存的shmid_ds结构复制到buf中
    IPC_SET：改变共享内存的状态，把buf所指的shmid_ds结构中的uid、gid、mode复制到共享内存的shmid_ds结构内
    IPC_RMID：删除这片共享内存
```

- 错误代码

```shell
    EACCESS：参数cmd为IPC_STAT，确无权限读取该共享内存
    EFAULT：参数buf指向无效的内存地址
    EIDRM：标识符为msqid的共享内存已被删除
    EINVAL：无效的参数cmd或shmid
    EPERM：参数cmd为IPC_SET或IPC_RMID，却无足够的权限执行
```

### 示例代码

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>

    int main(int argc, char const *argv[]) {
        key_t key = ftok(".", 0x01);
        if (key < 0) {
            LOG_ERROR("ftok failed, key %d", key);
            return -1;
        }

        //获取共享内存
        int shmid = shmget(key, sizeof(int), 0666 | IPC_CREAT);
        if (shmid == -1) {
            LOG_ERROR("shmget failed, %d, %s", errno, strerror(errno));
            return -1;
        }

        while ((pid = fork()) == -1);

        //将共享内存连接到当前进程的地址空间
        int *data = (int *) shmat(shmid, NULL, 0);
        if (data == (int *) -1) {
            LOG_ERROR("shmat fail, %d, %s", errno, strerror(errno));
            return -1;
        }

        while (1) {
            ...
        }

        if (pid != 0) {
            //父进程
            ...
            //把共享内存从当前进程中分离
            if (shmdt(data) == -1) {
                LOG_ERROR("shmdt failed, %d, %s", errno, strerror(errno));
                return -1;
            }

            //删除共享内存
            if (shmctl(shmid, IPC_RMID, 0) == -1) {
                LOG_ERROR("shmctl(IPC_RMID) failed, %d, %s", errno, strerror(errno));
                return -1;
            }
        } else {
            //子进程
            ...
            //把共享内存从当前进程中分离
            if (shmdt(data) == -1) {
                LOG_ERROR("shmdt failed, %d, %s", errno, strerror(errno));
                return -1;
            }
        }

        return 0;
    }
```

## 进程锁（信号量）

进程相互之间的变量包括指针指向的地址都是不共享的，进程间通信需要使用共享内存。

### 获取ID（ftok）

[同上](#ftok)

### 获取或创建信号量（shmget）

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>

    /*
     * @description 获取或者创建信号量
     * @param key 信号量ID，一般为ftok获取的ID
     * @param num_sems 信号量个数
     * @param sem_flags 标示属性，使用时需要与IPC对象存取权限（如0600）进行|运算来确定信号量集的存取权限
     * @return 共享内存ID；-1，错误，原因存于error中
     */
    int semget(key_t key, int num_sems, int sem_flags);

    errno               //错误码
    strerror(errno)     //错误信息字符串
```

- 函数传入值对应的操作

| key | size | shmflg                        | 描述                                                 |
| --- | ---- | ----------------------------- | ---------------------------------------------------- |
| x   | x    | IPC_CREAT $\mid$ 0600             | 建立新的信号量，不存在与key相同的则创建，存在返回key |
| x   | x    | IPC_CREAT $\mid$ IPC_EXCL $\mid$ 0600 | 建立新的信号量，不存在与key相同的则创建，存在报错    |
| x   | 0    | 0                             | 获取信号量                                           |

- 错误代码

```shell
    EACCES：没有访问该信号量集的权限
    EEXIST：信号量集已经存在，无法创建
    EINVAL：参数nsems的值小于0或者大于该信号量集的限制；或者是该key关联的信号量集已存在，并且nsems
    大于该信号量集的信号量数
    ENOENT：信号量集不存在，同时没有使用IPC_CREAT
    ENOMEM ：没有足够的内存创建新的信号量集
    ENOSPC：超出系统限制
```

### 信号量操作（PV操作）

此操作会导致进程阻塞，用于进程间加锁使用

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>

    /*
     * @description 信号量操作
     * @param semid 信号量ID
     * @param sops 操作结构体
     * @param nsops 操作的信号量的个数
     * @return 0，成功；-1，错误，原因存于error中
     */
    int semop(int semid, struct sembuf *sops, size_t nsops)；

    /*
     * @description 信号量操作
     * @param semid 信号量ID
     * @param sops 操作结构体
     * @param nsops 操作的信号量的个数
     * @param timespec 等待时间
     * @return 0，成功；-1，错误，原因存于error中
     */
    int semtimedop(int semid, struct sembuf *sops, unsigned nsops, struct timespec *timeout);

    errno               //错误码
    strerror(errno)     //错误信息字符串
```

- sembuf结构

```C
    struct sembuf{
        short sem_num;  // 信号量序号，从0开始
        short sem_op;   // 信号量在一次操作中需要改变的数据，通常是两个数，一个是-1，即P（等待）操作，
                        // 一个是+1，即V（发送信号）操作。
        short sem_flg;  //信号量操作标示
    };
```

sem_flg

```C
    IPC_NOWAIT  //对信号的操作不能满足时，semop()不会阻塞，并立即返回，同时设定错误信息。
    SEM_UNDO    //程序结束时(不论正常或不正常)，保证信号值会被重设为semop()调用前的值。这样做的目的在于避免程序在异常情况下结束时未将锁定的资源解锁，造成该资源永远锁定。
```

- 错误代码

```shell
    E2BIG：一次对信号的操作数超出系统的限制
    EACCES：调用进程没有权能执行请求的操作，并且不具有CAP_IPC_OWNER权能
    EAGAIN：信号操作暂时不能满足，需要重试
    EFAULT：sops或timeout指针指向的空间不可访问
    EFBIG：sem_num指定的值无效
    EIDRM：信号集已被移除
    EINTR：系统调用阻塞时，被信号中断
    EINVAL：参数无效
    ENOMEM：内存不足
    ERANGE：信号所允许的值越界
```

### 信号量管理

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>

    /*
     * @description 信号量管理
     * @param sem_id 信号量ID
     * @param sem_num 信号量序号，从0开始
     * @param command 操作符
     * @param ... 操作参数
     * @return 0，成功；-1，错误，原因存于error中
     */
    int semctl(int sem_id, int sem_num, int command, ...);

    errno               //错误码
    strerror(errno)     //错误信息字符串
```

- command

```shell
    SETVAL：用来把信号量初始化为一个已知的值。p 这个值通过union semun中的val成员设置，其作用是在信号量第一次使用前对它进行设置。
    IPC_RMID：用于删除一个已经无需继续使用的信号量标识符。
```

- 错误代码

```shell
    EACCES(权限不够)
    EFAULT(arg指向的地址无效)
    EIDRM(信号量集已经删除)
    EINVAL(信号量集不存在，或者semid无效)
    EPERM(EUID没有cmd的权利)
    ERANGE(信号量值超出范围)
```

### 示例代码

```C
    #include <sys/sem.h>
    #include <sys/ipc.h>
    #include <errno.h>
    ...

    int main(int argc, char const *argv[]) {
        key_t key = ftok(".", 0x01);
        if (key < 0) {
            LOG_ERROR("ftok failed, key %d", key);
            return -1;
        }

        //获取信号量
        int semId = semget(key, 1, IPC_CREAT | 0600);
        if (semId == -1) {
            LOG_ERROR("semget failed, %d, %s", errno, strerror(errno));
            return -1;
        }

        //设置初始值
        int code = semctl(semId, 0, SETVAL, 1);
        if (code == -1) {
            LOG_ERROR("semctl failed, %d, %s", errno, strerror(errno));
            return -1;
        }

        while ((pid = fork()) == -1);

        while (1) {
            //申请信号量，上锁
            struct sembuf signal;
            signal.sem_op = -1;
            signal.sem_flg = SEM_UNDO;
            signal.sem_num = 0;
            code = semop(semId, &signal, 1);
            if (code == -1) {
                LOG_ERROR("semop failed, %d, %s", errno, strerror(errno));
                return -1;
            }
            ...
            //释放信号量，解锁
            signal.sem_op = 1;
            code = semop(semId, &signal, 1);
            if (code == -1) {
                LOG_ERROR("semop failed, %d, %s", errno, strerror(errno));
                return -1;
            }
        }

        if (pid != 0) {
            //父进程
            ...
            //删除信号量
            int code = semctl(semId, 0, IPC_RMID);
            if (code == -1) {
                LOG_ERROR("semctl failed, %d, %s", errno, strerror(errno));
                return -1;
            }
        } else {
            //子进程
            ...
        }

        return 0;
    }
```

## 进程相关知识

- 进程除了创建的共享变量，所有变量包括全局变量和初始创建的变量均是不共享的
- 两个进程中地址相同的指针指向的是不同的位置，即malloc后fork出来的是两个malloc出来的指针，地址虽然打印相同，地址是逻辑地址，对两个进程是不同的。