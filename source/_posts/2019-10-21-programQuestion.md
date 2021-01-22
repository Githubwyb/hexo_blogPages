---
title: 编程题思路记录
date: 2019-10-21 10:54:51
tags:
categories: [Knowledge, Study]
top: 15
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

```cpp
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

```cpp
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

```cpp
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

```cpp
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

```cpp
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

我们可以用$2 \times 1$的小矩形横着或者竖着去覆盖更大的矩形。请问用n个$2 \times 1$的小矩形无重叠地覆盖一个$2 \times n$的大矩形，总共有多少种方法？

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

```cpp
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

```cpp
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

```cpp
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

```cpp
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

```cpp
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

# 二叉树的镜像

## 题目

操作给定的二叉树，将其变换为源二叉树的镜像。

输入描述:

```
二叉树的镜像定义：源二叉树
                    8
                   /  \
                  6   10
                 / \  / \
                 5  7 9 11
    	        镜像二叉树
    	            8
    	           /  \
    	          10   6
    	         / \  / \
    	        11 9 7  5
```

## 思路

### 递归版本

- 二叉树遍历，前序和后序都行，做操作即可
- 详见: [二叉树遍历](/blogs/2018-09-18-algorithmStudy/#towTree)

### 非递归版本

- 进行广度优先遍历或者深度优先遍历
- 弹出一个节点就交换其左右子树
- 详见: [广度优先和深度优先遍历](/blogs/2018-09-18-algorithmStudy/#treeSpan)

# 树的子结构

## 题目

输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

## 思路

直接递归实现，查找根节点，然后递归查找子节点

```C++
    /*
    struct TreeNode {
        int val;
        struct TreeNode *left;
        struct TreeNode *right;
        TreeNode(int x) :
                val(x), left(NULL), right(NULL) {
        }
    };*/
    class Solution {
    public:
        static bool HasSubtree(TreeNode* pRoot1, TreeNode* pRoot2)
        {
            if (pRoot1 == nullptr || pRoot2 == nullptr) {
                return false;
            }

            return HasSubtree1(pRoot1, pRoot2);
        }

    private:
        static bool HasSubtree1(TreeNode* pRoot1, TreeNode* pRoot2)
        {
            if (pRoot1 == nullptr || pRoot2 == nullptr) {
                return pRoot2 == nullptr;
            }

            if (pRoot1->val == pRoot2->val &&
                HasSubtree1(pRoot1->right, pRoot2->right) &&
                HasSubtree1(pRoot1->left, pRoot2->left)) {
                return true;
            }

            return HasSubtree1(pRoot1->right, pRoot2) || HasSubtree1(pRoot1->left, pRoot2);
        }
    };
```

# 从上往下打印二叉树

## 题目

从上往下打印出二叉树的每个节点，同层节点从左至右打印。

## 思路

二叉树的广度优先遍历，参考[广度优先遍历](/blogs/2018-09-18-algorithmStudy/#treeSpan)

# 栈的压入、弹出序列

## 题目

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

## 思路

模拟这个过程

```C++
    class Solution {
    public:
        static bool IsPopOrder(vector<int> pushV, vector<int> popV) {
            int size = pushV.size();
            if (size == 0 || size != popV.size()) {
                return false;
            }

            stack<int> test;
            int j = 0;
            int i = 0;
            while (i < size) {
                test.push(pushV[i++]);
                while (!test.empty() && test.top() == popV[j]) {
                    test.pop();
                    j++;
                }
            }

            return test.empty();
        }
    };
```

# 包含min函数的栈

## 题目

定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。

## 思路

两个栈，一个放数据，一个放最小值的一个栈

```C++
    class Solution {
    public:
        void push(int value) {
            if (m_minData.empty() || value <= m_minData.top()) {
                m_minData.push(value);
            }

            m_data.push(value);
        }

        void pop() {
            if (m_minData.top() == m_data.top()) {
                m_minData.pop();
            }
            m_data.pop();
        }

        int top() { return m_data.top(); }
        int min() {
            if (m_minData.empty()) {
                return -1;
            }
            return m_minData.top();
        }

    private:
        stack<int> m_data;
        stack<int> m_minData;
    };
```

# 顺时针打印矩阵

## 题目

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

## 思路

强解，直接使用四个变量保存起始位置，不断循环打印即可

# 数组中出现次数超过一半的数字

## 题目

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

## 思路

### 统计

使用map储存，遍历将数组每个值出现次数统计出来，找最多的对比size

### 排序取中值

将数组排序，中位数即为出现超过一半的，找出验证一下出现次数是否超过一半

### 相消

由于出现次数超过一半，所以不断排除两个不一样的数字，超过一半的数字一定会留在最后一个，验证一下输出

```C++
    class Solution {
    public:
        //巧解，排除相同的数，留下的即为所求
        static int MoreThanHalfNum_Solution2(vector<int> numbers) {
            int size = numbers.size();

            int times = 0;
            int result = 0;
            for (auto &tmp : numbers) {
                if (times == 0) {
                    result = tmp;
                }

                if (result == tmp) {
                    times++;
                    continue;
                }

                times--;
            }

            if (count(numbers.begin(), numbers.end(), result) > size / 2) {
                return result;
            }

            return 0;
        }
    };
```

# 二叉搜索树后序遍历数组校验

## 题目

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。

## 思路

- 二叉树搜索树满足根节点比左边大，比右边小
- 后序遍历，根节点在最后
- 从后向前，判断最后一个树比数组前一部分都大，比后一部分都小
- 找到分割点，递归判断前一部分和后一部分分别满足二叉搜索树后序遍历

```C++
    class Solution {
    public:
        static bool VerifySquenceOfBST(vector<int> sequence) {
            return !sequence.empty() &&
                VerifySquenceOfBST(sequence.begin().base(), sequence.size());
        }

    private:
        static bool VerifySquenceOfBST(const int arr[], int length) {
            if (length <= 0) {
                return true;
            }

            bool flag = false;
            int result = length;
            int i = 0;
            for (i = 0; i < length - 1; i++) {
                //异或，当false时，最后一位要比前面大，否则进判断；true时最后一位要比前面小，否则进逻辑
                if (flag ^ (arr[i] > arr[length - 1])) {
                    if (flag) {
                        //找到中间点后，出现小的，返回false
                        return false;
                    }
                    //中间点转判断
                    flag = true;
                    result = i;
                }
            }
            return (i == length - 1) ||
                (VerifySquenceOfBST(arr, result) &&
                    VerifySquenceOfBST(arr + result, length - result - 1));
        }
    };
```

# <span id="minKNumber">最小k个数</span>

## 题目

输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

## 思路

### 排序后取前k个值

- 将数组排序后取前k个值返回

### 维护一个k个值的数组，找到最小的k个值

```cpp
    class Solution {
    public:
        // 遍历，找到最小的k个值
        static vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
            int inputSize = input.size();
            if (k == 0 || k > inputSize) {
                return {};
            }
            vector<int> result(k);
            int size = 1;
            result[0] = input[0];
            // 遍历数组
            for (int i = 1; i < inputSize; i++) {
                // 不到k，且大于最后一位，直接放到最后一位
                if (size < k && input[i] > result[size - 1]) {
                    result[size] = input[i];
                    size++;
                    continue;
                }

                // 数组个数不到k，或者tmp小于数组最大值
                LOG_DEBUG("size %d, result[size - 1] %d, input[i] %d", size,
                        result[size - 1], input[i]);
                if (size < k || input[i] < result[size - 1]) {
                    if (size < k) {
                        // 不到k，最后一位在上面判断了肯定大于输入
                        result[size] = result[size - 1];
                        size++;
                    }
                    // 从最后一位开始，如果前一位大于tmp，后移，直到不大于跳出
                    int j = size - 1;
                    for (; j > 0 && result[j - 1] > input[i]; --j) {
                        LOG_DEBUG("j %d, result[j - 1] %d", j, result[j - 1]);
                        result[j] = result[j - 1];
                    }
                    // 跳出后，j指向要放的位置
                    result[j] = input[i];
                }
            }
            return result;
        }
    };
```

### 大根堆

- 维护一个k个值的大根堆
- 遍历和堆顶比，小于则弹出堆顶，插入新元素排序

```cpp
    class Solution {
    public:
        // 最小堆解问题，C++标准接口
        static vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
            int inputSize = input.size();
            if (k == 0 || k > inputSize) {
                return {};
            }

            vector<int> result(k + 1);
            // 插入前k个值并构造最大堆
            for (int i = 0; i < k; i++) {
                result[i] = input[i];
                // 插入构建大根堆
                push_heap(result.begin(), result.begin() + i + 1);
            }

            // 判断，如果大于最大节点，替换并重构最大堆
            for (int i = k; i < inputSize; i++) {
                if (input[i] < result[0]) {
                    result[k] = input[i];
                    // 弹出堆顶，重排序大根堆
                    pop_heap(result.begin(), result.end());
                }
            }

            // 多余一个删掉
            result.pop_back();
            return result;
        }
    };
```

# 明明的随机数

## 题目

明明想在学校中请一些同学一起做一项问卷调查，为了实验的客观性，他先用计算机生成了N个1到1000之间的随机整数（N≤1000），对于其中重复的数字，只保留一个，把其余相同的数去掉，不同的数对应着不同的学生的学号。然后再把这些数从小到大排序，按照排好的顺序去找同学做调查。请你协助明明完成“去重”与“排序”的工作(同一个测试用例里可能会有多组数据，希望大家能正确处理)。

## 思路

- C++标准库有两个接口可以直接实现，一个排序，一个去重

```cpp
    int main() {
        int num;
        while (cin >> num) {
            vector<int> input;
            for (int i = 0; i < num; i++) {
                int tmp = 0;
                cin >> tmp;
                input.push_back(tmp);
            }

            sort(input.begin(), input.end());
            input.erase(unique(input.begin(), input.end()), input.end());
            for (auto &tmp : input) {
                cout << tmp << endl;
            }
        }
    }
```

# 连续子数组的最大和

## 题目

HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。给一个数组，返回它的最大连续子序列的和，你会不会被他忽悠住？(子向量的长度至少是1)

## 思路

- 动态规划
- 找函数，以第$i$个元素结尾的最大和是$f(i)$，则第$i + 1$个元素结尾的最大和应为$max(f(i) + array[i - 1], array[i - 1])$
- 所以可以表示为$f(i + 1) = max(f(i) + array[i - 1], array[i - 1])$

```cpp
    class Solution {
    public:
        static int FindGreatestSumOfSubArray(vector<int> array) {
            int maxTail = -1;
            int max = 0x80000000;
            for (auto &tmp : array) {
                // f(i + 1) = max(f(i) + array[i - 1], array[i - 1])
                maxTail = (maxTail + tmp) > tmp ? (maxTail + tmp) : tmp;
                max = max < maxTail ? maxTail : max;
            }
            return max;
        }
    };
```

# 二叉树中和为某一值的路径

## 题目

输入一颗二叉树的根节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)

## 思路

- 从根节点向下递归
- 维持一个vector放入经过的路径，返回时弹出最后一个元素
- 到根部则判断是否为所求，是则插入结果中

```cpp
    class Solution {
    public:
        static vector<vector<int>> FindPath(TreeNode *root, int expectNumber) {
            if (root == nullptr) {
                return {};
            }
            vector<vector<int>> result;
            vector<int> road;
            FindPath(root, expectNumber, result, road);
            sort(result.begin(), result.end(), comp);
            return result;
        }

    private:
        static bool comp(vector<int> a, vector<int> b) {
            return a.size() > b.size();
        }

        static void FindPath(TreeNode *root, int expectNumber,
                            vector<vector<int>> &result, vector<int> &road) {
            if (root == nullptr) {
                return;
            }

            // 不到根部，sum增加，road放入此节点，继续递归
            expectNumber -= root->val;
            road.emplace_back(root->val);

            if (root->right == nullptr && root->left == nullptr &&
                expectNumber == 0) {
                // 查到根部，判断是否相等，相等则存入结果
                result.emplace_back(road);
            }

            FindPath(root->right, expectNumber, result, road);
            FindPath(root->left, expectNumber, result, road);

            expectNumber += root->val;
            road.pop_back();
        }
    };
```

# 二叉树的深度

## 题目

输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。

## 思路

### 递归

- 返回左节点最大深度和右节点最大深度的最大值加一

```cpp
    class Solution {
    public:
        // 递归实现
        static int TreeDepth(TreeNode *pRoot) {
            if (pRoot == nullptr) {
                return 0;
            }
            return max(TreeDepth(pRoot->left) + 1, TreeDepth(pRoot->right) + 1);
        }
    };
```

### 非递归

- 类似广度优先遍历，不过是一次取出同一级别所有节点

```cpp
    class Solution {
    public:
        // 非递归实现，广度优先遍历
        static int TreeDepth(TreeNode *pRoot) {
            if (pRoot == nullptr) {
                return 0;
            }

            int maxDeep = 0;
            queue<TreeNode *> traversal;
            traversal.push(pRoot);
            while(!traversal.empty()) {
                // 获取当前层的节点数，此循环全部取出
                int length = traversal.size();
                for (int i = 0; i < length; i++)
                {
                    auto &tmp = traversal.front();
                    if (tmp->left != nullptr) traversal.push(tmp->left);
                    if (tmp->right != nullptr) traversal.push(tmp->right);
                    traversal.pop();
                }

                maxDeep++;
            }
            return maxDeep;
        }
    };
```

# 第一个只出现一次的字符

## 题目

在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置, 如果没有则返回 -1（需要区分大小写）.

## 思路

- 由于是字符，所以只有128位甚至更少，asscii可以显示的最大数
- 使用一个`int [128]`来统计各个字符出现次数
- 再次遍历找到第一个出现次数为1的字符

# 硬币方案数

## 题目

有数量不限的硬币，币值为25分、10分、5分和1分，请编写代码计算n分有几种表示法。
给定一个int n，请返回n分有几种表示法。保证n小于等于100000，为了防止溢出，请将答案Mod 1000000007。

## 思路

n种硬币凑成sum的方案数，可以假设有$i$个第n种硬币，然后剩下的则是用n - 1种硬币凑成$sum - i * coins[n]$的最大方案数，即$dp[n][sum] = \sum_{i = 0}^{sum / coins[n]}dp[n - 1][sum - i * coins[n]]$，经推导可得

$$ \begin{aligned}
dp[n][sum] &= dp[n - 1][sum - 0 * coins[n]] + dp[n - 1][sum - 1 * coins[n]] +...+ dp[n - 1][sum - sum / coins[n] * coins[n]] \\\\
    & = \sum_{i = 0}^{sum / coins[n]}dp[n - 1][sum - i * coins[n]] \\\\
    & = dp[n - 1][sum] + \underbrace{\sum_{i = 1}^{sum / coins[n]}dp[n - 1][sum - i * coins[n]]}\_{令sum1 = sum - coins[n]} \\\\
    & = dp[n - 1][sum] + \sum_{i = 1}^{sum1 / coins[n] + 1}dp[n - 1][sum1 - (i - 1) * coins[n]] \\\\
    & = dp[n - 1][sum] + \underbrace{\sum_{i = 0}^{sum1 / coins[n]}dp[n - 1][sum1 - i * coins[n]]}_{dp[n][sum1]} \\\\
    & = dp[n - 1][sum] + dp[n][sum - coins[n]]
\end{aligned} $$

所以$dp[n][sum] = dp[n - 1][sum] + dp[n][sum - coins[n]]$，代码实现上用一个一维数组去累加即可

```cpp
    class Coins {
       public:
        static int countWays(int n) {
            int coins[] = {1, 5, 10, 25};
            // 初始化为1，只有1种硬币时，所有都只有一种方案
            vector<int> dp(n + 1, 1);
            // 从2种硬币开始
            for (unsigned int i = 1; i < sizeof(coins) / sizeof(coins[0]); i++) {
                // dp[i][j] = dp[i - 1][j] + dp[i][j - coins[i]];，从coins[i]开始，coins[i]以内可coins[i - 1]一致
                for (int j = coins[i]; j <= n; j++) {
                    dp[j] += dp[j - coins[i]];
                    dp[j] %= 1000000007;
                }
            }
            return dp[n];
        }
    };
```

# 计算1 + 2 + 3 + ... + n

## 题目

求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

## 思路

- 主要运用逻辑与运算的短路特性，前一个false后一个将不会执行
- 计算可以强行累加或者运用公式$\frac{n(n + 1)}{2}$
- 运用公式则使用了乘法的位运算

$$ x \times y =
((y >> 0) \& 1) \times x \times 2^{0} + ((y >> 1) \& 1) \times x \times 2^{1} +...+ ((y >> 31) \& 1) \times x \times 2^{31} \\\\
x \times 2^n = x << n $$

```C++
    class Solution {
       public:
        static int Sum_Solution(int n) {
            int sum = 0;
            int num = 1 + n;
            Sum_Solution(sum, num, n);
            return sum >> 1;
        }

       private:
        static bool Sum_Solution(int &sum, int &num, int &n) {
            // 某一位为1时，加上数值num * 2^x
            (n & 0x01) && (sum += num);
            // num *= 2
            num <<= 1;
            n >>= 1;
            // n为0则结束
            return n && (Sum_Solution(sum, num, n));
        }
    };
```

# 链表的公共节点

## 题目

输入两个链表，找出它们的第一个公共结点。（注意因为传入数据是链表，所以错误测试数据的提示是用其他方式显示的，保证传入数据是正确的）

## 思路

### 暴力破解

1. 一个一个找节点是否相同
2. 可递归可循环

```cpp
    class Solution {
       public:
        ListNode *FindFirstCommonNode(ListNode *pHead1, ListNode *pHead2) {
            auto p1 = pHead1;
            auto p2 = pHead2;
            // 要么相等，要么同时为nullptr
            while (p1 != p2) {
                // p1到尾部，p2后移
                p2 = (p1 == nullptr ? p2 : p2->next);
                // p1到尾部，从头再来
                p1 = (p1 == nullptr ? pHead1 : p1->next);
            }
            return p1;
        }
    };
```

### 有点思考的解法

1. 如果有相同节点，必有公共尾部
2. 从中间某个节点开始相同
3. 先计算长度，然后从尾部公共长度开始找
4. 一共遍历三次

### 最骚解法

```cpp
    class Solution {
       public:
        ListNode *FindFirstCommonNode(ListNode *pHead1, ListNode *pHead2) {
            auto p1 = pHead1;
            auto p2 = pHead2;
            // 要么相等，要么同时为nullptr
            while (p1 != p2) {
                p1 = (p1 == nullptr ? pHead2 : p1->next);
                p2 = (p2 == nullptr ? pHead1 : p2->next);
            }
            return p1;
        }
    };
```

1. 如果长度相同，第一遍出结果
2. 长度不相同，第一遍找到差值，第二遍出结果
    - 假设长度分别为`a、b，a > b`
    - 第一遍p2先跑完指向第一个链表，p1走到第一个链表`a - b`
    - 在p1走完时，p2走到第一个链表的`a - (a - b) = b`长度位置
    - p1指向第二个链表，这时两个指针指向相同长度的链表
    - 一起走，找到公共节点则返回，找不到会同时到nullptr，返回

# 数组中重复的数字

## 题目

在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。

## 思路

### 常规思路

1. 搞一个bool型数组，访问过就写成true，找到第一个为true的

```cpp
    class Solution {
       public:
        // Parameters:
        //        numbers:     an array of integers
        //        length:      the length of array numbers
        //        duplication: (Output) the duplicated number in the array number
        // Return value:       true if the input is valid, and there are some
        // duplications in the array number
        //                     otherwise false
        static bool duplicate(int numbers[], int length, int* duplication) {
            vector<bool> count(length, false);
            for (int i = 0; i < length; i++) {
                if (count[numbers[i]]) {
                    *duplication = numbers[i];
                    return true;
                }
                count[numbers[i]] = true;
            }
            return false;
        }
    };
```

### 不骚不行，就是要骚

1. 就用原数组搞事情，访问某个数，以它为索引的地方加上n
2. 找到以数索引大于n的，返回这个数

```cpp
    class Solution {
       public:
        // Parameters:
        //        numbers:     an array of integers
        //        length:      the length of array numbers
        //        duplication: (Output) the duplicated number in the array number
        // Return value:       true if the input is valid, and there are some
        // duplications in the array number
        //                     otherwise false
        static bool duplicate(int numbers[], int length, int* duplication) {
            for (int i = 0; i < length; i++) {
                // 获取原始数据
                auto tmp = numbers[i] < length ? numbers[i] : numbers[i] - length;
                // 如果对应位小于length证明没有被访问过
                if (numbers[tmp] < length) {
                    // 访问它，增加length
                    numbers[tmp] += length;
                    continue;
                }
                // 访问过，就是它了
                *duplication = tmp;
                return true;
            }
            return false;
        }
    };
```

# 数字在排序数组中出现的次数

## 题目

统计一个数字在排序数组中出现的次数。

## 思路

- 排序数组二分查找即可
- 当然用标准库啊

```cpp
    class Solution {
       public:
        static int GetNumberOfK(vector<int> data, int k) {
            auto tmp =
                equal_range(data.begin(), data.end(), k);
            return tmp.second - tmp.first;
        }
    };
```

# 数组中只出现一次的数字

## 题目

一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

## 思路

1. 都出现了两次，说明异或为0
2. 只有两个出现了一次，那么异或结果就是他们俩的异或
3. `0 ^ 1 = 1`，为1的是两个数字不同的位，以此为标准分开两个数组分别异或
4. 两个结果就为异或结果

```cpp
    class Solution {
       public:
        static void FindNumsAppearOnce(vector<int> data, int* num1, int* num2) {
            int nor = 0;
            // 先把所有数字异或，结果为两个单独出现的数字的异或
            for (auto &tmp : data) {
                nor ^= tmp;
            }

            int bitNum = 0;
            // 找到第一个不同的位
            for (int i = 0; i < 32; ++i) {
                if (nor & 0x01) {
                    bitNum = i;
                    break;
                }
                nor >>= 1;
            }
            bitNum = 1 << bitNum;

            // 根据此位将所有数字分成两组进行异或，最终得到结果
            *num1 = 0;
            *num2 = 0;
            for (auto &tmp : data) {
                if (tmp & bitNum) {
                    *num1 ^= tmp;
                    continue;
                }

                *num2 ^= tmp;
            }
        }
    };
```