---
title: 算法与数据结构学习
date: 2018-09-18 15:07:21
tags: [algorithm]
categories: [notes, study]
top: true
---

# 数学知识复习

## 对数

### 定理 1.1
$$ log_A{B} = \frac{log_C{B}}{log_C{A}} ; C > 0 $$

## 级数

### 几何级数公式

$$ \sum_{i = 0}^{N}2^i = 2^{N+1} - 1 $$

$$ \sum_{i = 0}^{N}A^i = \frac{A^{N+1} - 1}{A - 1} $$

当N趋于$\infty$时，该和趋于$\frac{1}{1-A}$。

# 算法分析

## 数学基础

### 四个定义

- 如果存在正常数 $c$ 和 $n_0$ 使得当 $N \geq n_0$ 时 $T(N) \leq cf(N)$ ，则即为 $T(N) = O(f(N))$ 。
- 如果存在正常数 $c$ 和 $n_0$ 使得当 $N \geq n_0$ 时 $T(N) \geq cg(N)$ ，则即为 $T(N) = \Omega(f(N))$ 。
- $T(N) = \Theta (h(N))$ 当且仅当 $T(N) = O(h(N))$ 且 $T(N) = \Omega(h(N))$ 。
- 如果 $T(N) = O(p(N))$ 且 $T(N) \neq \Theta(p(N))$ ，则 $T(N) = o(p(N))$

# 算法

## 排序算法

### 时间复杂度总结

<img src = "2018_09_26_01.jpg" width = "80%" />

### 插入排序

#### 原理

- 自我理解像扑克牌起牌一样，一张一张插入到已有的序列中

<img src = "2018_09_26_02.gif" width = "80%" />

```C++
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

### 希尔排序

- 带间隔的插入排序

<img src = "2018_09_26_03.gif" width = "80%" />

```C++
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

### 堆排序

-

<img src = "2018_09_26_04.gif" width = "80%" />

## 最短路径算法

### Dijkstra(迪杰斯特拉)算法

Dijkstra(迪杰斯特拉)算法是典型的单源最短路径算法，用于计算一个节点到其他所有节点的最短路径。主要特点是以起始点为中心向外层层扩展，直到扩展到终点为止。

#### 算法描述

<img src = "2019_02_23_08.png" width = "40%">

<img src = "2019_02_23_09.png" width = "80%">

示例，自己写的，没有考虑内存和时间，改了一下，变成有向路径算法。无向路径可以使用上述U集作为while判断，减少循环次数

```C++
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

```C++
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

# 数据结构

## 树

### 遍历

#### 二叉树前序遍历

- 指先访问根，然后访问子树的遍历方式

<img src = "2019_02_22_05.png" width = "80%">

```C
    void in_order_traversal(TreeNode *root) {
        // Do Something with root
        if (root->lchild != NULL)
            in_order_traversal(root->lchild);
        if (root->rchild != NULL)
            in_order_traversal(root->rchild);
    }
```

#### 二叉树中序遍历

- 指先访问左（右）子树，然后访问根，最后访问右（左）子树的遍历方式

<img src = "2019_02_22_06.png" width = "80%">

```C
    void in_order_traversal(TreeNode *root) {
        if (root->lchild != NULL)
            in_order_traversal(root->lchild);
        // Do Something with root
        if (root->rchild != NULL)
            in_order_traversal(root->rchild);
    }
```

#### 二叉树后序遍历

- 指先访问子树，然后访问根的遍历方式

<img src = "2019_02_22_07.png" width = "80%">

```C
    void in_order_traversal(TreeNode *root) {
        if (root->lchild != NULL)
            in_order_traversal(root->lchild);
        if (root->rchild != NULL)
            in_order_traversal(root->rchild);
        // Do Something with root
    }
```
