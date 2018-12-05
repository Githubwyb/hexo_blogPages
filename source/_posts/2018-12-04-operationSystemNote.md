---
title: 操作系统笔记（C语言）
date: 2018-12-04 16:57:43
tags: [study, notes, os, C/C++]
categories: [notes, study]
---

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

## 进程锁（共享内存）

进程相互之间的变量包括指针指向的地址都是不共享的，进程间通信需要使用共享内存。

### 获取或创建共享内存

```C
    #include <sys/ipc.h>
    #include <sys/shm.h>
    #include <errno>
    
    /*
     * @description 获取或者创建共享内存
     * @param key 共享内存ID
     * @param size 共享内存大小
     * @param shmflg 标示属性，使用时需要与IPC对象存取权限（如0600）进行|运算来确定信号量集的存取权限
     * @return 共享内存ID；-1，错误，原因存于error中
     */
    int shmget(key_t key, size_t size, int shmflg)

    errno               //错误码
    strerror(errno)     //错误信息字符串
```

- 函数传入值对应的操作

| key            | size | shmflg | 描述                 |
| -------------- | ---- | ------ | -------------------- |
| 0(IPC_PRIVATE) | x    |    IPC_CREAT    | 建立新的共享内存对象，不存在与key相同的则创建，存在返回key |
| 0(IPC_PRIVATE) | x    |    IPC_CREAT\|IPC_EXCL    | 建立新的共享内存对象，不存在与key相同的则创建，存在报错 |
| x           | 0    | 0      | 获取共享内存         |

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

## 进程相关知识

- 进程除了创建的共享变量，所有变量包括全局变量和初始创建的变量均是不共享的
- 两个进程中地址相同的指针指向的是不同的位置，即malloc后fork出来的是两个malloc出来的指针，地址虽然打印相同，地址是逻辑地址，对两个进程是不同的。