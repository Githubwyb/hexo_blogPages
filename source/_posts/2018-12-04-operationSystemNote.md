---
title: 操作系统笔记
date: 2018-12-04 16:57:43
tags: [study, notes, os]
categories: [notes, study]
---

# 线程

## 创建线程

- 创建函数

```C
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
    /*
     * @description 申请锁
     * @param mutex 线程互斥锁
     * @return 0，创建成功；其他，错误码
     */
    int pthread_mutex_lock(pthread_mutex_t *mutex);
```

- 解锁函数

```C
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

## 线程相关知识

- 线程内部创建变量是不共享的
- 全局变量线程内部共享

# 进程
