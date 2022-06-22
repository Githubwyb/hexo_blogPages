---
title: 算法与数据结构学习
date: 2018-09-18 15:07:21
tags: [算法]
categories: [Knowledge, Study]
top: 19
---

# 一、数学知识复习

## 对数

### 定理 1.1
$$ log_A{B} = \frac{log_C{B}}{log_C{A}} ; C > 0 $$

## 级数

### 几何级数公式

$$ \sum_{i = 0}^{N}2^i = 2^{N+1} - 1 $$

$$ \sum_{i = 0}^{N}A^i = \frac{A^{N+1} - 1}{A - 1} $$

当N趋于$\infty$时，该和趋于$\frac{1}{1-A}$。

# 二、算法分析

## 数学基础

### 四个定义

- 如果存在正常数 $c$ 和 $n_0$ 使得当 $N \geq n_0$ 时 $T(N) \leq cf(N)$ ，则即为 $T(N) = O(f(N))$ 。
- 如果存在正常数 $c$ 和 $n_0$ 使得当 $N \geq n_0$ 时 $T(N) \geq cg(N)$ ，则即为 $T(N) = \Omega(f(N))$ 。
- $T(N) = \Theta (h(N))$ 当且仅当 $T(N) = O(h(N))$ 且 $T(N) = \Omega(h(N))$ 。
- 如果 $T(N) = O(p(N))$ 且 $T(N) \neq \Theta(p(N))$ ，则 $T(N) = o(p(N))$

# 三、算法

## 1. 排序算法

### 时间复杂度总结

<img src = "2018_09_26_01.jpg" width = "80%" />

### 1.1. 插入排序

#### 原理

- 自我理解像扑克牌起牌一样，一张一张插入到已有的序列中

<img src = "2018_09_26_02.gif" width = "80%" />

```cpp
    /*
     * @function 用插入排序数组
     * @param data 数组首地址
     * @param length 数组长度
     * @param order 排列顺序：顺序，true；倒序，false
     */
    template<typename T>
    void insertionSort(T *data, int length, bool order = true) {
        T tmp = 0;
        for (int i = 1; i < length; ++i) {
            tmp = data[i];
            int j = 0;
            for (j = i; j > 0 && ((data[j - 1] > tmp) ^ !order); --j) {
                data[j] = data[j - 1];
            }
            data[j] = tmp;
        }
    }
```

#### 定理

- $N$个互异数的数组的平均逆序数是$N(N-1)/4$。
- 通过交换相邻元素进行排序的任何算法平均需要$\Omega(N^2)$时间。

### 1.2. 希尔排序

- 带间隔的插入排序

<img src = "2018_09_26_03.gif" width = "80%" />

```cpp
/*
 * @function 用希尔增量的希尔排序数组
 * @param data 数组首地址
 * @param length 数组长度
 * @param order 排列顺序：顺序，true；倒序，false
 */
template<typename T>
void shellSort(T *data, int length, bool order = true) {
    T tmp = 0;
    for (int increment = length / 2; increment > 0; increment /= 2) {
        for (int i = increment; i < length; ++i) {
            tmp = data[i];
            int j = 0;
            for (j = i; j >= increment && ((data[j - increment] > tmp) ^ !order); j -= increment) {
                data[j] = data[j - increment];
            }
            data[j] = tmp;
        }
    }
}
```

#### 定理

- 使用希尔增量时希尔排序的最坏情形的运行时间为$\Theta(N^2)$。
- 使用Hibbard增量的希尔排序的最坏情形运行时间为$\Theta(N^{\frac{3}{2}})$

### 1.3. 堆排序

<img src = "2018_09_26_04.gif" width = "80%" />

- 大根堆或小根堆，保证父节点一定大于（小于）子节点，头节点为最大（最小）的

### 1.4. 快速排序（分治思想）

<img src = "2021_03_26_01.gif" width = "80%" />

- 选定一个中轴数，将数组分为大于和小于的部分，中轴放中间
- 对大于部分和小于部分分别做同样的事

## 2. 最短路径算法

### Dijkstra(迪杰斯特拉)算法

Dijkstra(迪杰斯特拉)算法是典型的单源最短路径算法，用于计算一个节点到其他所有节点的最短路径。主要特点是以起始点为中心向外层层扩展，直到扩展到终点为止。

#### 算法描述

<img src = "2019_02_23_08.png" width = "40%">

<img src = "2019_02_23_09.png" width = "80%">

示例，自己写的，没有考虑内存和时间，改了一下，变成有向路径算法。无向路径可以使用上述U集作为while判断，减少循环次数

```cpp
#include "log.hpp"

#include <iostream>
#include <memory>
#include <vector>
#include <fstream>
#include <map>

#define InputFileName "../input.txt"

using namespace std;

//按行返回数据
int readFile(string &input) {
    input = "";
    static ifstream inFile(InputFileName);
    if (!inFile) {
        LOG_ERROR("Failed to open file %s", InputFileName);
        return -1;
    }

    while (!inFile.eof()) {
        char buf[128];
        inFile.getline(buf, 128);
        input = string(buf);
        return 0;
    }

    return -1;
}

//将字母转成数字
int getIndex(char a) {
    switch (a) {
        case 'A':
            return 0;

        case 'B':
            return 1;

        case 'C':
            return 2;

        case 'D':
            return 3;

        case 'E':
            return 4;

        case 'F':
            return 5;

        default:
            return -1;
    }
}

//将数字转成字母
char getChar(int a) {
    switch (a) {
        case 0:
            return 'A';

        case 1:
            return 'B';

        case 2:
            return 'C';

        case 3:
            return 'D';

        case 4:
            return 'E';

        case 5:
            return 'F';

        default:
            return -1;
    }
}

int main() {
    //由于已知点数量，第一行废掉
    string inputStr = "";
    if (readFile(inputStr) != 0) {
        LOG_ERROR("Read file error");
        return -1;
    }

    //定义距离矩阵
    int distance[6][6] = {0};
    for (int i = 0; i < 6; ++i) {
        for (int j = 0; j < 6; ++j) {
            //初始化所有距离为-1
            distance[i][j] = -1;
        }
        //自己到自己为0
        distance[i][i] = 0;
    }

    map<char, string> road;
    while (readFile(inputStr) == 0) {
        int a = getIndex(inputStr[0]);      //第一个点
        int b = getIndex(inputStr[2]);      //第二个点
        int c = stoi(inputStr.substr(3));   //距离
        //赋值给距离矩阵
        LOG_INFO("%c--%c: %d", getChar(a), getChar(b), c);
        distance[a][b] = c;
    }

    while (true) {
        bool change = false;
        for (int i = 0; i < 6; ++i) {
            //U集合中A点可到达的点i
            if (distance[0][i] != -1) {
                //将路径加进去
                if (road.count(getChar(i)) == 0) {
                    string roadStr = "A";
                    road[getChar(i)] = roadStr + getChar(i);
                }

                for (int j = 0; j < 6; ++j) {
                    //筛选出i点可到达的j点
                    if (distance[i][j] != -1) {
                        //A点不可到达j点或者当前已计算的距离大于A从i到达j的距离
                        if (distance[0][j] == -1 ||
                            distance[0][j] > (distance[0][i] + distance[i][j])) {
                            change = true;
                            //更新距离表
                            distance[0][j] = distance[0][i] + distance[i][j];
                            //更新路径表
                            string tmpStr = road[getChar(i)];
                            road[getChar(j)] = tmpStr + getChar(j);
                        }
                    }
                }
            }
        }

        //一轮没有改变则证明已经计算完毕
        if (!change) {
            break;
        }
    }

    for (int i = 0; i < 6; ++i) {
        for (int j = 0; j < 6; ++j) {
            PRINT("%d\t", distance[i][j]);
        }
        PRINT("\r\n");
    }

    //打印结果
    PRINT("\r\n");
    LOG_INFO("Result:");
    for (int m = 0; m < 6; ++m) {
        LOG_INFO("%c %s: %d", getChar(m), road[getChar(m)].c_str(), distance[0][m]);
    }
}
```

### Floyd算法

Floyd-Warshall算法（Floyd-Warshall algorithm）是解决任意两点间的最短路径的一种算法，可以正确处理有向图或负权的最短路径问题，同时也被用于计算有向图的传递闭包。Floyd-Warshall算法的时间复杂度为O(N3)，空间复杂度为O(N2)。

#### 算法描述

使用距离矩阵，从第一个顶点开始，计算所有两个点通过1点的路径是否是最短。然后再从第二个顶点开始，同样计算所有两个点的路径是否是最短，直到所有点计算完毕，得到的距离矩阵即为最短路径。

示例

```cpp
#include "log.hpp"

#include <iostream>
#include <memory>
#include <vector>
#include <fstream>
#include <map>

#define InputFileName "../input.txt"

using namespace std;

//按行返回数据
int readFile(string &input) {
    input = "";
    static ifstream inFile(InputFileName);
    if (!inFile) {
        LOG_ERROR("Failed to open file %s", InputFileName);
        return -1;
    }

    while (!inFile.eof()) {
        char buf[128];
        inFile.getline(buf, 128);
        input = string(buf);
        return 0;
    }

    return -1;
}

//将字母转成数字
int getIndex(char a) {
    switch (a) {
        case 'A':
            return 0;

        case 'B':
            return 1;

        case 'C':
            return 2;

        case 'D':
            return 3;

        case 'E':
            return 4;

        case 'F':
            return 5;

        default:
            return -1;
    }
}

//将数字转成字母
char getChar(int a) {
    switch (a) {
        case 0:
            return 'A';

        case 1:
            return 'B';

        case 2:
            return 'C';

        case 3:
            return 'D';

        case 4:
            return 'E';

        case 5:
            return 'F';

        default:
            return -1;
    }
}

int main() {
    //由于已知点数量，第一行废掉
    string inputStr = "";
    if (readFile(inputStr) != 0) {
        LOG_ERROR("Read file error");
        return -1;
    }

    //定义距离矩阵
    int distance[6][6] = {0};
    for (int i = 0; i < 6; ++i) {
        for (int j = 0; j < 6; ++j) {
            //初始化所有距离为-1
            distance[i][j] = -1;
        }
        //自己到自己为0
        distance[i][i] = 0;
    }

    map<string, string> road;
    while (readFile(inputStr) == 0) {
        int a = getIndex(inputStr[0]);      //第一个点
        int b = getIndex(inputStr[2]);      //第二个点
        int c = stoi(inputStr.substr(3));   //距离
        //赋值给距离矩阵
        LOG_INFO("%c--%c: %d", getChar(a), getChar(b), c);
        distance[a][b] = c;
        //添加到路由表
        char tmp[2] = {getChar(a), getChar(b)};
        road[tmp] = string(tmp);
    }

    for (int k = 0; k < 6; ++k) {
        for (int i = 0; i < 6; ++i) {
            for (int j = 0; j < 6; ++j) {
                //添加key
                char tmp[3] = {getChar(i), getChar(j), 0x00};
                //防止路径遗漏，补充路由表
                if (distance[i][j] != -1 && road.count(tmp) == 0) {
                    road[tmp] = tmp;
                }
                //可通过k到达i且可通过k到达j
                if (distance[i][k] != -1 && distance[k][j] != -1) {
                    //ij之间没有通路或者当前计算的最短路径大于通过k到达的距离
                    if (distance[i][j] == -1 ||
                        distance[i][j] > distance[i][k] + distance[k][j]) {
                        distance[i][j] = distance[i][k] + distance[k][j];
                        //更新路由表
                        char tmp1[3] = {getChar(i), getChar(k), 0x00};
                        char tmp2[3] = {getChar(k), getChar(j), 0x00};
                        string tmpRoad = road[tmp1];
                        tmpRoad.pop_back();     //防止重复字母
                        road[tmp] = tmpRoad + road[tmp2];
                    }
                }
            }
        }
    }

    for (int i = 0; i < 6; ++i) {
        for (int j = 0; j < 6; ++j) {
            PRINT("%d\t", distance[i][j]);
        }
        PRINT("\r\n");
    }

    //打印结果
    PRINT("\r\n");
    LOG_INFO("Result:");
    for (int m = 0; m < 6; ++m) {
        char tmp[3] = {getChar(m), 0x00, 0x00};
        for (int i = 0; i < 6; ++i) {
            tmp[1] = getChar(i);
            LOG_INFO("%c->%c %s: %d", tmp[0], tmp[1], road[tmp].c_str(), distance[m][i]);
        }
    }
}
```

## 其他算法

### 3.1. 全排列算法（回溯法）

<img src = "2020_04_29_01.png">

- 原理是遍历替换首字母和后面的字符，替换到最后输出

```C++
    class Solution {
       public:
        vector<string> Permutation(string str) {
            int len = str.length();
            if (len == 0) {
                return {};
            }
            m_result.clear();
            Permutation(str, 0, len);
            // 这里做字典排序
            sort(m_result.begin(), m_result.end());
            m_result.erase(unique(m_result.begin(), m_result.end()),
                           m_result.end());
            return m_result;
        }

       private:
        // index为交换的头指针
        void Permutation(string str, int index, int len) {
            if (len == index) {
                m_result.emplace_back(str);
                return;
            }

            // 不交换的情况
            Permutation(str, index + 1, len);
            for (int i = index + 1; i < len; i++) {
                // 交换首字符和后面的字符，继续遍历
                swap(str[index], str[i]);
                Permutation(str, index + 1, len);
                swap(str[index], str[i]);
            }
        }

        vector<string> m_result;
    };
```

### 3.2. 动态规划

- 动态规划一般是推导真实场景的某一状态和下一状态之间的关系
- 根据两个状态的关系列出方程，然后从第一个状态不断向状态n靠近

# 四、数据结构

## 1. 树

### 1.1. 遍历

#### 1) <span id = "treeSpan">树的深度优先和广度优先遍历</span>

定义树结构

<img src = "2019_02_27_10.png" width = "40%">

##### 深度优先遍历

深度优先遍历（Depth First Search），简称DFS，其原则是，沿着一条路径一直找到最深的那个节点，当没有子节点的时候，返回上一级节点，寻找其另外的子节点，继续向下遍历，没有就向上返回一级，直到所有的节点都被遍历到，每个节点只能访问一次。

###### 算法步骤：

使用栈的数据结构实现

1. 首先将根节点1压入栈中【1】
2. 将1节点弹出，找到1的两个子节点3，2，首先压入3节点，再压入2节点（后压入左节点的话，会先取出左节点，这样就保证了先遍历左节点），2节点再栈的顶部，最先出来【2，3】
3. 弹出2节点，将2节点的两个子节点5，4压入【4，5，3】
4. 弹出4节点，将4的子节点9，8压入【8，9，5，3】
5. 弹出8，8没有子节点，不压入【9，5，3】
6. 弹出9，9没有子节点，不压入【5，3】
7. 弹出5，5有一个节点，压入10，【10，3】
8. 弹出10，10没有节点，不压入【3】
9. 弹出3，压入3的子节点7，6【6，7】
10. 弹出6，没有子节点【7】
11. 弹出7，没有子节点，栈为空【】，算法结束

出栈顺序【1，2，4，8，9，5，10，3，6，7】

```cpp
    #include <stack>
    #include <memory>
    #include <vector>

    //树结构
    typedef struct Node {
        int value;
        vector<shared_ptr<Node>> pChild;
        weak_ptr<Node> pParent;
    } Node;

    //深度优先打印节点
    void printNodesDeepFirst(const shared_ptr<Node> &node) {
        stack<shared_ptr<Node>> myStack;
        myStack.push(node);
        while (myStack.size() > 0) {
            shared_ptr<Node> pTmp = myStack.top();
            myStack.pop();
            PRINT("%d ", pTmp->value);

            //由于栈的后进先出特性，需要倒序使左节点先出
            for (int i = pTmp->pChild.size(); i != 0; --i) {
                myStack.push(pTmp->pChild[i - 1]);
            }
        }
        PRINT("\r\n");
    }
```

##### 广度优先遍历

广度优先遍历（Breadth First Search），简称BFS；广度优先遍历的原则就是对每一层的节点依次访问，一层访问结束后，进入下一层，直到最后一个节点，同样的，每个节点都只访问一次。

###### 算法步骤：

使用队列的数据结构实现

1. 节点1，插入队列【1】
2. 取出节点1，插入1的子节点2，3 ，节点2在队列的前端【2，3】
3. 取出节点2，插入2的子节点4，5，节点3在队列的最前端【3，4，5】
4. 取出节点3，插入3的子节点6，7，节点4在队列的最前端【4，5，6，7】
5. 取出节点4，插入3的子节点8，9，节点5在队列的最前端【5，6，7，8，9】
6. 取出节点5，插入5的子节点10，节点6在队列的最前端【6，7，8，9，10】
7. 取出节点6，没有子节点，不插入，节点7在队列的最前端【7，8，9，10】
8. 取出节点7，没有子节点，不插入，节点8在队列的最前端【8，9，10】
9. 取出节点8，没有子节点，不插入，节点9在队列的最前端【9，10】
10. 取出节点9，没有子节点，不插入，节点10在队列的最前端【10】
11. 取出节点10，队列为空，算法结束

我们看一下节点出队的顺序【1，2，3，4，5，6，7，8，9，10】

```cpp
    #include <queue>
    #include <memory>
    #include <vector>

    //树结构
    typedef struct Node {
        int value;
        vector<shared_ptr<Node>> pChild;
        weak_ptr<Node> pParent;
    } Node;

    //广度优先打印节点
    void printNodesWidthFirst(const shared_ptr<Node> &node) {
        queue<shared_ptr<Node>> myQueue;
        myQueue.push(node);
        while (myQueue.size() > 0) {
            shared_ptr<Node> pTmp = myQueue.front();
            myQueue.pop();
            PRINT("%d ", pTmp->value);

            for (auto tmp : pTmp->pChild) {
                myQueue.push(tmp);
            }
        }
        PRINT("\r\n");
    }
```

#### 2) <span id = "towTree">二叉树前中后序遍历</span>

- 前序遍历指先访问根，然后访问子树的遍历方式

<img src = "2019_02_22_05.png" width = "40%">

```cpp
    void in_order_traversal(TreeNode *root) {
        // Do Something with root
        if (root->lchild != NULL)
            in_order_traversal(root->lchild);
        if (root->rchild != NULL)
            in_order_traversal(root->rchild);
    }
```

- 中序遍历指先访问左（右）子树，然后访问根，最后访问右（左）子树的遍历方式

<img src = "2019_02_22_06.png" width = "40%">

```cpp
    void in_order_traversal(TreeNode *root) {
        if (root->lchild != NULL)
            in_order_traversal(root->lchild);
        // Do Something with root
        if (root->rchild != NULL)
            in_order_traversal(root->rchild);
    }
```

- 后序遍历指先访问子树，然后访问根的遍历方式

<img src = "2019_02_22_07.png" width = "40%">

```cpp
    void in_order_traversal(TreeNode *root) {
        if (root->lchild != NULL)
            in_order_traversal(root->lchild);
        if (root->rchild != NULL)
            in_order_traversal(root->rchild);
        // Do Something with root
    }
```

### 1.2. 二叉搜索树

二叉查找树（Binary Search Tree），（又：二叉搜索树，二叉排序树）它或者是一棵空树，或者是具有下列性质的二叉树：
**若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值； 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值；**
它的左、右子树也分别为二叉排序树。

#### 性质

二叉排序树的查找过程和次优二叉树类似，通常采取二叉链表作为二叉排序树的存储结构。**中序遍历**二叉排序树可得到一个关键字的有序序列，一个无序序列可以通过构造一棵二叉排序树变成一个有序序列，构造树的过程即为对无序序列进行排序的过程。每次插入的新的结点都是二叉排序树上新的叶子结点，在进行插入操作时，不必移动其它结点，只需改动某个结点的指针，由空变为非空即可。搜索,插入,删除的复杂度等于树高，O(log(n)).

### 1.3. 大（小）根堆（优先队列）

大（小）根堆是堆的两种形式之一。根结点（亦称为堆顶）的关键字是堆里所有结点关键字中最大（小）者，称为大（小）根堆，又称最大（小）堆、大（小）顶堆。大（小）根堆要求根节点的关键字既大（小）于或等于左子树的关键字值，又大（小）于或等于右子树的关键字值。
普通的队列是一种先进先出的数据结构，元素在队列尾追加，而从队列头删除。在优先队列中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出 （first in, largest out）的行为特征。通常采用堆数据结构来实现。

#### 性质和应用

- 最大或最小值在堆顶，可以用于排序数组
- 可以用队列表示，第$i$个元素大于或者小于第$i * 2$和第$i * 2 + 1$个元素
- 最基本的应用，查找[数组最小（大）k个值](/blogs/2019-10-21-programQuestion/#minKNumber)
- [C++标准库有接口可以直接应用](/blogs/2018-07-06-CppStudy/#bigHeap)

### 1.4. 红黑树

参考 [红黑树(R-B tree)原理图文详解](https://zhuanlan.zhihu.com/p/78152265)

#### 1) 红黑树的特性和应用

- 红黑树的查找和插入时间复杂度都是 $O(\log n)$，相比hash表更加稳定

**应用**

- std::map和std::set使用的是红黑树
- epoll的底层实现是用红黑树组织fd

## 2. hashTable 哈希表

### 2.1. 为什么哈希表的除数要用素数

参考自 [腾讯面试真题：证明为什么哈希表除m取余法的被除数为什么用素数比较好](https://blog.csdn.net/w_y_x_y/article/details/82288178)

- 使用合数可能和等差数列的差值含有公因数，导致碰撞概率增大

#### 1) 假设

- 传入的key是等差数列: 首项1，差值从2到5，长度10
- 两个hash表，一个使用6取模，一个使用7取模

#### 2) 效果

**差值2**

| 余数 | 0   | 1   | 2   | 3   | 4   | 5   |
| ---- | --- | --- | --- | --- | --- | --- |
| 首次 |     | 1   |     | 3   |     | 5   |
| 碰撞 |     | 7   |     | 9   |     | 11  |
| 碰撞 |     | 13  |     | 15  |     | 17  |
| 碰撞 |     | 19  |     |     |     |     |

| 余数 | 0   | 1   | 2   | 3   | 4   | 5   | 6   |
| ---- | --- | --- | --- | --- | --- | --- | --- |
| 首次 | 7   | 1   | 9   | 3   | 11  | 5   | 13  |
| 碰撞 |     | 15  |     | 17  |     | 19  |     |

**差值3**

| 余数 | 0   | 1   | 2   | 3   | 4   | 5   |
| ---- | --- | --- | --- | --- | --- | --- |
| 首次 |     | 1   |     |     | 4   |     |
| 碰撞 |     | 7   |     |     | 10  |     |
| 碰撞 |     | 13  |     |     | 16  |     |
| 碰撞 |     | 19  |     |     | 22  |     |
| 碰撞 |     | 25  |     |     | 28  |     |

| 余数 | 0   | 1   | 2   | 3   | 4   | 5   | 6   |
| ---- | --- | --- | --- | --- | --- | --- | --- |
| 首次 | 7   | 1   | 16  | 10  | 4   | 19  | 13  |
| 碰撞 | 28  | 22  |     |     | 25  |     |     |

**差值4**

| 余数 | 0   | 1   | 2   | 3   | 4   | 5   |
| ---- | --- | --- | --- | --- | --- | --- |
| 首次 |     | 1   |     | 9   |     | 5   |
| 碰撞 |     | 13  |     | 21  |     | 17  |
| 碰撞 |     | 25  |     | 33  |     | 29  |
| 碰撞 |     | 37  |     |     |     |     |

| 余数 | 0   | 1   | 2   | 3   | 4   | 5   | 6   |
| ---- | --- | --- | --- | --- | --- | --- | --- |
| 首次 | 21  | 1   | 9   | 17  | 25  | 5   | 13  |
| 碰撞 |     | 29  | 37  |     |     | 33  |     |

**差值5**

| 余数 | 0   | 1   | 2   | 3   | 4   | 5   |
| ---- | --- | --- | --- | --- | --- | --- |
| 首次 | 6   | 1   | 26  | 21  | 16  | 11  |
| 碰撞 | 36  | 31  |     |     | 46  | 41  |

| 余数 | 0   | 1   | 2   | 3   | 4   | 5   | 6   |
| ---- | --- | --- | --- | --- | --- | --- | --- |
| 首次 | 21  | 1   | 16  | 31  | 11  | 26  | 6   |
| 碰撞 |     | 36  |     |     | 46  |     | 41  |

#### 3) 结论

- 如果差值和被除数之间不含有公因数，效果一样
- 如果含有公因数，碰撞概率会变高
