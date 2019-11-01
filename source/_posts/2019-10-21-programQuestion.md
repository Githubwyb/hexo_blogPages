---
title: 编程题思路记录
date: 2019-10-21 10:54:51
tags: 
categories: [Knowledge, Study]
---

# 二维数组查找

## 题目

在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

## 思路

从右侧最上方开始，查到小于后，竖着找大于的，然后横着，直到数组溢出或者找到为止

# <span id = "H2">斐波那切数列输出</span>

## 题目

大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。
n<=39

## 思路

### 硬解

保存前一次和前前一次，一步一步算

### 递归

- 每次计算中直接调用自己的`n - 1`和`n - 2`的值
- 防止递归过大，预留一个40的int数组，如果数组相应索引有值直接返回，没值递归算出保存返回

### 动态规划

- 两个变量，g保存当前和前一次，f保存前一次和前两次
- 计算方法

```c++
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

# <span id = "H4">青蛙跳台阶</span>

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

# 两个栈实现队列

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

# 旋转数组查找最小值

## 题目

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

## 思路

- 数组旋转后，最小值为分界点的值，利用二分查找方式最快到达最小值输出。

```c++
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

# 变态跳台阶的问题

## 题目

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

## 思路

- 数学统计题，$f(n) = f(n - 1) + f(n - 2) + ... + f(1) + 1 = 2^{n - 1}$
- 使用`1 << (number - 1)`计算2的`n - 1`次幂更快

# 重建二叉树

## 问题

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

## 思路

- 中序遍历的头结点左边为左子树，右边为右子树
- 前序遍历第一个为头结点
- 两个结合，找到头结点在中序遍历中的位置，左边递归出来为左子树，右边递归出来为右子树

```c++
    class Solution {
    public:
        static TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> vin) {
            if (pre.size() != vin.size()) {
                return nullptr;
            }
            return reConstructBinaryTree(pre.begin().base(), vin.begin().base(), pre.size());
        }

    private:
        static TreeNode* reConstructBinaryTree(const int *pre, const int *vin, int size) {
            if (size == 0) {
                return nullptr;
            }

            //头结点从pre第一个取
            auto head = new TreeNode(pre[0]);
            //找到和vin中和pre[0]相等的点，左边为左子树，右边为右子树
            for (int i = 0; i < size; ++i) {
                if (vin[i] == pre[0]) {
                    head->left = reConstructBinaryTree(pre + 1, vin, i);
                    head->right = reConstructBinaryTree(pre + i + 1, vin + i + 1, size - i - 1);
                    break;
                }
            }

            return head;
        }
    };
```

# 链表倒数第k个值

## 题目

输入一个链表，输出该链表中倒数第k个结点。

## 思路

- 两个指针，一个比另一个先走k步，直到遍历完整个链表返回第二个指针

# 链表翻转

## 题目

输入一个链表，反转链表后，输出新链表的表头。

## 思路

- 三个指针，一个遍历，一个翻转next，一个保存前一个节点

```c++
    class Solution {
    public:
        static ListNode* ReverseList(ListNode* pHead) {
            ListNode *p1 = nullptr;     //跟在后面翻转next的指针
            ListNode *p2 = nullptr;     //保存前一次

            while (pHead != nullptr) {
                p2 = p1;
                p1 = pHead;
                pHead = pHead->next;
                p1->next = p2;
            }
            
            return p1;
        }
    };
```

# 矩形覆盖

## 问题

我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？

## 思路

- 分析一下，相当于[青蛙跳台阶](#H4)，竖着放就是跳一阶，横着放就是跳两阶


# 调整数组中奇数位于偶数前面

## 题目

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

## 思路

### 硬解

- 查找到第一个偶数和后面第一个奇数
- 将中间的偶数后移一位，奇数前移到偶数的位置
- 编译直到完成

### 另加队列

- 遍历，偶数放入新队列，奇数前移
- 遍历完后拷贝新队列到老队列最后一个奇数后面

```c++
    class Solution {
    public:
        static void reOrderArray(vector<int> &array) {
            vector<int> Odd;     //偶数队列
            int size = array.size();
            int j = 0;              //保存最后待放偶数的位置
            for (int i = 0; i < size; ++i) {
                if ((array[i] % 2) == 0) {
                    //偶数加到新队列
                    Odd.emplace_back(array[i]);
                    continue;
                }

                if (i != j) {
                    //奇数前移
                    array[j] = array[i];
                }

                ++j;
            }

            memcpy(array.begin().base() + j, Odd.begin().base(), Odd.size() * sizeof(int));
        }
    };
```

### 标准库 stable_partition

```c++
    class Solution {
    public:
        static void reOrderArray(vector<int> &array) {
            // 奇数放前面，偶数放后面，两边分别的相对位置保持不变
            stable_partition(array.begin(), array.end(),
                                [](const int &value) { return (value % 2 == 1); });
        }
    };
```

# 数值的整数次幂

## 题目

给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
保证base和exponent不同时为0

## 思路

- 简单快速幂算法

```c++
    class Solution {
    public:
        static double Power(double base, int exponent) {
            unsigned int tmp = abs(exponent);
            double r = 1.0;
            while (tmp) {
                if (tmp & (unsigned) 1) {
                    //对应位是1时，乘以base
                    r *= base;
                }
                //tmp移位，base要平方
                base *= base;
                tmp >>= (unsigned) 1;
            }
            return exponent < 0 ? 1 / r : r;
        }
    };
```

# 合并两个排序的列表

## 题目

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

## 思路

### 非递归版本

```c++
    class Solution {
    public:
        static ListNode *Merge(ListNode *pHead1, ListNode *pHead2) {
            if (pHead1 == nullptr) {
                return pHead2;
            }

            if (pHead2 == nullptr) {
                return pHead1;
            }

            ListNode* result = nullptr;
            if (pHead1->val < pHead2->val) {
                result = pHead1;
                pHead1 = pHead1->next;
            } else {
                result = pHead2;
                pHead2 = pHead2->next;
            }

            ListNode* pTmp = result;
            while (true) {
                if (pHead1 == nullptr) {
                    pTmp->next = pHead2;
                    break;
                }
                
                if (pHead2 == nullptr) {
                    pTmp->next = pHead1;
                    break;
                }
                
                if (pHead1->val < pHead2->val) {
                    pTmp->next = pHead1;
                    pHead1 = pHead1->next;
                } else {
                    pTmp->next = pHead2;
                    pHead2 = pHead2->next;
                }
                pTmp = pTmp->next;
            }

            return result;
        }
    };
```

### 递归版本

- 合并链表选出较大的节点，下一跳时剩下两个链表再选

```c++
    class Solution {
    public:
        static ListNode *Merge(ListNode *pHead1, ListNode *pHead2) {
            if (pHead1 == nullptr) {
                return pHead2;
            }

            if (pHead2 == nullptr) {
                return pHead1;
            }

            ListNode* result = nullptr;
            if (pHead1->val < pHead2->val) {
                result = pHead1;
                result->next = Merge(pHead1->next, pHead2);
            } else {
                result = pHead2;
                result->next = Merge(pHead2->next, pHead1);
            }

            return result;
        }
    };
```