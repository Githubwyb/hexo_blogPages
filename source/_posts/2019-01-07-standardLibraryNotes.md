---
title: 标准库函数记录
date: 2019-01-07 14:35:37
tags:
categories: [Program, C/C++]
---

# C标准库

## string.h

### 初始化函数 memset()

```C
    #include <string.h>

    /*
     * @description 将n bytes起始地址为s的内存全部初始化为c
     * @param __s 初始化地址
     * @param __c 初始化的值
     * @param __n 初始化的大小
     * @return __s的地址
     */
    void *memset (void *__s, int __c, size_t __n);
```

**函数是按byte(8 bits)进行初始化的，每个字节均会被初始化为__c取前八位的值。**

## setjmp

### 非局部跳转函数 setjmp()/longjmp()

```C
    #include <setjmp.h>

    /*
     * @description 设置返回位置
     * @param __env 保存状态信息的缓存地址
     * @return 直接调用返回，0；调用longjmp后返回longjmp给入参数__val
     */
    int setjmp (jmp_buf __env);

    /*
     * @description 跳转到setjmp的位置
     * @param __env 跳转位置的状态信息
     * @param __val 跳转位置返回值
     */
    void longjmp (jmp_buf __env, int __val);
```

实例

```C++
    #include <iostream>
    #include <csetjmp>

    using namespace std;

    class Rainbow {
    public:
        Rainbow() {
            cout << "Rainbow()" << endl;
        }

        ~Rainbow() {
            cout << "~Rainbow()" << endl;
        }
    };

    jmp_buf kansas;

    void oz() {
        Rainbow rb;
        longjmp(kansas, 47);
    }

    int main() {
        int code = setjmp(kansas);
        if ((code = setjmp(kansas)) == 0) {
            cout << "Setjmp code " << code << endl;
            oz();
        } else {
            cout << "Setjmp code " << code << endl;
        }
    }
```

输出

```shell
    Setjmp code 0
    Rainbow()
    Setjmp code 47
```

#### 注意事项

- 在上述实例中发现类没有被析构，这两个函数可以实现跳转但是不会检测类相关，所以类不会被析构
