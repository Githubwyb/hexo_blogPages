---
title: 编程题思路记录
date: 2019-10-21 10:54:51
tags: 
categories: [Knowledge, Study]
---

# 1. 二维数组查找

## 题目

在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

## 思路

从右侧最上方开始，查到小于后，竖着找大于的，然后横着，直到数组溢出或者找到为止

# <span id = "H2">2. 斐波那切数列输出</span>

## 题目

大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。
n<=39

## 思路

### 2.1. 硬解

保存前一次和前前一次，一步一步算

### 2.2. 递归

- 每次计算中直接调用自己的`n - 1`和`n - 2`的值
- 防止递归过大，预留一个40的int数组，如果数组相应索引有值直接返回，没值递归算出保存返回

### 2.3. 动态规划

- 两个变量，g保存当前和前一次，f保存前一次和前两次
- 计算方法

```C++
    class Solution {
    public:
        int Fibonacci(int n) {
            int f = 0, g = 1;
            while(n--) {
                g += f;     //从前一次到当前
                f = g - f;  //从前两次算到前一次
            }
            return f;
        }
    };
```

# 3. 青蛙跳台阶

## 题目

一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

## 思路

青蛙跳n阶台阶，从第一跳来算:

- 第一次跳1阶，剩下种类为$f(n - 1)$
- 第一次跳2阶，剩下种类为$f(n - 2)$

则

$$f(n) = f(n - 1) + f(n - 2)$$

其中

$$f(0) = 0, f(1) = 1，f(2) = 2$$

分析可以看出和[斐波那切数列输出](#H2)除了初始值基本一致

# 4. 两个栈实现队列

## 题目

用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

## 思路

- stack1为入队列栈，stack2为出队列栈
- stack2不为空时直接出栈，为空则从stack1取所有元素

```c++
    class Solution
    {
    public:
        void push(int node) {
            stack1.push(node);
        }

        int pop() {
            //为空才从stack1拿数据
            if (stack2.empty()) {
                //stack1也是空的，就返回-1
                if (stack1.empty()) {
                    return -1;
                }

                while (!stack1.empty()) {
                    stack2.push(stack1.top());
                    stack1.pop();
                }
            }

            int result = stack2.top();
            stack2.pop();
            return result;
        }

    private:
        stack<int> stack1;  //入队列
        stack<int> stack2;  //出队列
    };
```

# 5. 旋转数组查找最小值

## 题目

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

## 思路

- 数组旋转后，最小值为分界点的值，利用二分查找方式最快到达最小值输出。

```C++
    class Solution {
    public:
        int minNumberInRotateArray(vector<int> rotateArray) {
            if (rotateArray.empty()) {
                return 0;
            }

            int left = 0;
            int right = rotateArray.size() - 1;

            //另类的二分查找，比左边的大，左边等，比右边的小，右边等
            while (true) {
                int tmp = (left + right) / 2;

                if (tmp == left) {
                    break;
                }

                if (rotateArray[tmp] >= rotateArray[left]) {
                    left = tmp;
                } else {
                    right = tmp;
                }
            }

            if (rotateArray[left] < rotateArray[right]) {
                return rotateArray[left];
            }

            return rotateArray[right];
        }
    };
```

# 6. 变态跳台阶的问题

## 题目

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

## 思路

- 数学统计题，$f(n) = f(n - 1) + f(n - 2) + ... + f(1) + 1 = 2^{n - 1}$
- 使用`1 << (number - 1)`计算2的`n - 1`次冪更快