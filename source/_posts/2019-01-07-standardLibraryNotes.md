---
title: 标准库函数记录
date: 2019-01-07 14:35:37
tags: [C/C++]
categories: [notes, study]
---

# C标准库

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

# C++标准库

## stack栈

```C++
    #include <stack>

    using namespace std;

    int main() {
        stack<int> myStack;     //定义类型为int
        myStack.push(10);       //入栈
        myStack.push(50);       //入栈
        int a = myStack.top();  //返回栈顶元素的引用，不会出栈
        myStack.pop();          //出栈，void型
        bool isEmpty = myStack.empty();         //是否为空
        unsigned long size = myStack.size();    //栈的元素数量
    }
```

### 注意事项

- push()，实际上是调用的底层容器的push_back()函数，新元素的值是push函数参数的一个拷贝。
- emplace()，实际上是调用的底层容器的emplace_back()函数，新元素的值是在容器内部就地构造的，不需要移动或者拷贝。