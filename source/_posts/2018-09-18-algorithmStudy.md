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

## 1. 数学基础

### 1.1. 四个定义

- 如果存在正常数 $c$ 和 $n_0$ 使得当 $N \geq n_0$ 时 $T(N) \leq cf(N)$ ，则即为 $T(N) = O(f(N))$ 。
- 如果存在正常数 $c$ 和 $n_0$ 使得当 $N \geq n_0$ 时 $T(N) \geq cg(N)$ ，则即为 $T(N) = \Omega(f(N))$ 。
- $T(N) = \Theta (h(N))$ 当且仅当 $T(N) = O(h(N))$ 且 $T(N) = \Omega(h(N))$ 。
- 如果 $T(N) = O(p(N))$ 且 $T(N) \neq \Theta(p(N))$ ，则 $T(N) = o(p(N))$

## 2. 排列组合

### 2.1. 公式记录

- n个不同元素取m个，关心顺序。为排列，使用A表示

$$ A_n^m = \frac{n!}{(n-m)!} $$

- n个不同元素取m个，不关心顺序。为组合，使用C表示
  - 排列的方式除以顺序造成的多余方案数

$$ C_n^m = \frac{A_n^m}{m!} = \frac{n!}{m!(n-m)!} $$
$$ C_n^m = C_n^{n-m} $$

## 3. 质数

使用欧拉线性筛计算出来的 $10^n$ 以内的质数的数量

```
10 4
100 25
1000 168
10000 1229
100000 9592
1000000 78498
10000000 664579
100000000 5761455
```

### 3.1. 埃氏筛

- 埃氏筛是将一个合数分成一个质数乘以一个整数，那么循环所有的数，将他们乘以小于自己的质数，得到的结果一定不是质数
- 那么循环到某个数时，如果其没有被小于自己的数排掉，那么一定是质数

#### 证明

- 反证法，如果有一个合数x没有被排掉，可以分为 $x = a \times b，a为因数中最小的质数$
- b为合数一定大于a，因为b也可以分为质因数相乘，a为其中最小的；b为质数同样大于a
- 在循环到b时，乘以小于自己的所有质数，到a时一定会把x排除掉，假设不成立，证明是可行的

### 3.2. 欧拉线性筛：使用O(n)的时间复杂度找出n以内的所有质数

- 核心思想是埃氏筛，但是使用一个提前退出将时间复杂度降到O(n)
- 当一个合数 $x = a \times b$，其中b为a的倍数，a为质数，遍历到x时提前退出的唯一顾虑是 $n = x \times c = a \times b \times c，c为质数$ 没有做排除
- 当遍历到 $b \times c$ 时，会将n进行排除，所以此顾虑消失，提前退出减少循环计算次数

```go
const mx int = 1e8

var primes []int = make([]int, 0, 5*1e7)

func main() {
	flag := make([]bool, mx+1) // 标记数有没有被筛掉，false就是没有
	check := 10                // 打印使用
	for i := 2; i < mx+1; i++ {
		if !flag[i] {
			// 数没有被比自己小的数筛掉，就代表是质数
			primes = append(primes, i)
		}
		for _, v := range primes {
			if i*v > mx {
				break
			}
			// 每一个数都作为因子乘以比自己小的素数筛掉后面的数
			flag[i*v] = true
			if i%v == 0 {
				// 减少时间复杂度的关键算法
				// 12 = 2 * 3 * 2，i = 4时，只排了8就退出了，因为6会将12排除
				// 也就是，假设v可以整除i即i = kv，有某个数为x = mi = kmv
				//        那么存在一个数 i < km < x可以把x排掉，用i乘以所有的质数去排除就没什么意义了，提前退出减少时间复杂度
				break
			}
		}
		// 仅打印使用
		if i == check {
			fmt.Println(i, len(primes))
			check *= 10
		}
	}
}
```

# 三、数据结构

## 1. 树

### 1.1. 公共部分

#### 1) 遍历

##### (1) <span id = "treeSpan">树的深度优先和广度优先遍历</span>

定义树结构

<img src = "2019_02_27_10.png" width = "40%">

###### 深度优先遍历

深度优先遍历（Depth First Search），简称DFS，其原则是，沿着一条路径一直找到最深的那个节点，当没有子节点的时候，返回上一级节点，寻找其另外的子节点，继续向下遍历，没有就向上返回一级，直到所有的节点都被遍历到，每个节点只能访问一次。

**算法步骤：**

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

###### 广度优先遍历

广度优先遍历（Breadth First Search），简称BFS；广度优先遍历的原则就是对每一层的节点依次访问，一层访问结束后，进入下一层，直到最后一个节点，同样的，每个节点都只访问一次。

**算法步骤：**

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

#### 2) 度

- 孩子结点个数就是结点的度，0度就是没有孩子结点
- 树的度就是结点中最大的度

### 1.2. 二叉树

#### 1) 性质和算法

##### (1) <span id = "towTree">二叉树前中后序遍历</span>

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

##### (2) 完全二叉树和满二叉树

- 满二叉树，只有最后一行是叶子节点，其他的节点都是度为2的节点
- 完全二叉树指的是除了最后一行，整体是满二叉树，最后一行叶子节点从左到右依次排列

#### 2) 二叉搜索树

二叉查找树（Binary Search Tree），（又：二叉搜索树，二叉排序树）它或者是一棵空树，或者是具有下列性质的二叉树：
**若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值； 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值；**
它的左、右子树也分别为二叉排序树。

##### 性质

二叉排序树的查找过程和次优二叉树类似，通常采取二叉链表作为二叉排序树的存储结构。**中序遍历**二叉排序树可得到一个关键字的有序序列，一个无序序列可以通过构造一棵二叉排序树变成一个有序序列，构造树的过程即为对无序序列进行排序的过程。每次插入的新的结点都是二叉排序树上新的叶子结点，在进行插入操作时，不必移动其它结点，只需改动某个结点的指针，由空变为非空即可。搜索,插入,删除的复杂度等于树高，O(log(n)).

#### 3) 大（小）根堆（优先队列）

大（小）根堆是堆的两种形式之一。根结点（亦称为堆顶）的关键字是堆里所有结点关键字中最大（小）者，称为大（小）根堆，又称最大（小）堆、大（小）顶堆。大（小）根堆要求根节点的关键字既大（小）于或等于左子树的关键字值，又大（小）于或等于右子树的关键字值。
普通的队列是一种先进先出的数据结构，元素在队列尾追加，而从队列头删除。在优先队列中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出 （first in, largest out）的行为特征。通常采用堆数据结构来实现。

##### 性质和应用

- 最大或最小值在堆顶，可以用于排序数组
- 可以用队列表示，第$i$个元素大于或者小于第$i * 2$和第$i * 2 + 1$个元素
- 最基本的应用，查找[数组最小（大）k个值](/blogs/2019-10-21-programQuestion/#minKNumber)
- [C++标准库有接口可以直接应用](/blogs/2018-07-06-CppStudy/#bigHeap)

#### 4) 红黑树

参考 [红黑树(R-B tree)原理图文详解](https://zhuanlan.zhihu.com/p/78152265)

##### (1) 红黑树的特性和应用

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

## 3. bitmap 位图

### 3.1. 原理

- 一个int表示32位，可以表示32个数字，即

| 0   | 1   | 2   | 3   | ... | 31  |
| --- | --- | --- | --- | --- | --- |
| 0   | 0   | 1   | 0   | ... | 1   |

- 代表2和31都在集合中
- 按照在二进制中的位置表示对应的数字
- 使用数组可以将一个很大的内存区域集合在一起，根据对应的位置说明对应的数字

### 3.2. 举例

- 对一大批不同的11位qq号进行排序

#### 1) 实现

- 11位qq号最大99999999999，也就是需要 $1 \times 10^{12}$ 个bit保存
- 计算成int数组 $1 \times 10^{12} / 32 = 31250000000$

```cpp
#define MAX_QQ_NUMBER 99999999ll
const long BITS_PER_LONG = 8 * sizeof(long);
long qq_bitmap[MAX_QQ_NUMBER / BITS_PER_LONG + 1] = {0};

void set_qq_bitmap(long long qq) {
    if (qq > MAX_QQ_NUMBER) return;
    // 假设32位，long为32位
    // 先将输入的qq按照32一组找到对应的位置，然后将在其在32个一组中的位置置1
    // 61 => qq_bitmap[1] |= (1 << 29)
    qq_bitmap[qq / BITS_PER_LONG] |= (1 << (qq % BITS_PER_LONG));
}

long check_qq_bitmap(long long qq) {
    if (qq > MAX_QQ_NUMBER) return 0;
    // 同理，找到qq按照32个一组的位置，检查对应的32个数字中的位置是否为1
    return qq_bitmap[qq / BITS_PER_LONG] & (1 << (qq % BITS_PER_LONG));
}
```

## 4. ST 稀疏表

### 4.1. 原理

- 本身是解决区间问题，也就是给一个序列，找到`a-b`之间的最大值（或者其他值）为x的有几组
- 这种题目暴力解决，需要先计算所有两个点之间的最大值存到一个 $n \times n$ 的表里面，然后遍历表进行统计，空间复杂度和时间复杂度太高
- 使用稀疏表可以使用比较少的空间并节省大量的计算，用到的矩阵大小是 $n \times log_2 n$
- 首次对数据进行预处理形成一个稀疏表后，可以直接进行 $O(1)$ 的查询
- 整理成的矩阵的每个元素代表（拿取最大值表示）

$$
st[j][i] = \max \limits_{j \le x \le j + 2^{i}-1} arr[x]
$$

- 想要取a到b之间的结果，那么取 $s = log_2(b-a+1)$

$$
f(a, b) = \max(st[a][s], st[b-2^{s}+1][s])
$$

- 其中 $a + 2^{s}-1 = a + (b - a + 1) - 1 = b \ge a = b - (b - a + 1) + 1 = b - 2^{s} + 1$

## 5. 并查集

### 5.1. 原理

- 找关系，给一系列点，给出一些点和点的关系。然后找出某两个点是否有关系
- 单一一个可以直接使用bfs解就好了，如果空间不足，使用并查集可以做到 $O(n)$ 的空间复杂度
- 首先将所有点初始化关系只有自己和自己
- 然后遍历一边关系列表，将点和点连接起来，找到公共的父级
- 查找时就是看是否两个点存在公共父级

### 5.2. 实现

- 主要实现两个方法，查找和插入
- 由于查找要用递归，想要减少递归深度，每次查找，整条链上的所有节点的父级都要指向最终点，这样下次只有一级查找

```go
type unionFind struct {
	parent []int
}

func initUnionFind(n int) unionFind {
	u := unionFind{}
	u.parent = make([]int, n)
	for i := range u.parent {
		u.parent[i] = i
	}
	return u
}

func (u unionFind) find(a int) int {
	ap := u.parent[a]
	// 找到最终节点
	for ap != u.parent[ap] {
		ap = u.parent[ap]
	}
	// 沿途都赋值最终节点
	for a != ap {
		u.parent[a], a = ap, u.parent[a]
	}
	return ap
}

func (u unionFind) merge(a, b int) {
	// b的父节点等于a的父节点，就是将两个点合并
	u.parent[u.find(b)] = u.find(a)
}
```

# 四、算法

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

- $N$个互异数的数组的平均逆序数是 $N(N-1)/4$ 。
- 通过交换相邻元素进行排序的任何算法平均需要 $\Omega(N^2)$ 时间。

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

- 使用希尔增量时希尔排序的最坏情形的运行时间为 $\Theta(N^2)$ 。
- 使用Hibbard增量的希尔排序的最坏情形运行时间为 $\Theta(N^{\frac{3}{2}})$

### 1.3. 堆排序

<img src = "2018_09_26_04.gif" width = "80%" />

- 大根堆或小根堆，保证父节点一定大于（小于）子节点，头节点为最大（最小）的

### 1.4. 快速排序（分治思想）

<img src = "2021_03_26_01.gif" width = "80%" />

- 选定一个中轴数，将数组分为大于和小于的部分，中轴放中间
- 对大于部分和小于部分分别做同样的事

## 2. 最短路径算法

### Dijkstra(迪杰斯特拉)算法

Dijkstra(迪杰斯特拉)算法是典型的单源最短路径算法，用于计算一个节点到某个节点的最短路径。主要特点是以起始点为中心向外层层扩展，直到扩展到终点为止。

#### 算法描述

<img src = "2019_02_23_08.png" width = "40%">

<img src = "2019_02_23_09.png" width = "80%">

```go
type pointS struct {
	ch   byte
	step int
}

type littleQueue []pointS

func (q *littleQueue) Push(v interface{}) {
	*q = append(*q, v.(pointS))
}
func (q *littleQueue) Pop() interface{} {
	x := (*q)[len(*q)-1]
	*q = (*q)[:len(*q)-1]
	return x
}
func (q *littleQueue) Len() int           { return len(*q) }
func (q *littleQueue) Less(i, j int) bool { return (*q)[i].step < (*q)[j].step }
func (q *littleQueue) Swap(i, j int)      { (*q)[i], (*q)[j] = (*q)[j], (*q)[i] }

func dijkstra(src byte, dst byte, distMaps map[byte]map[byte]int) int {
	// 定义小根堆，主要为了每次取最小的一个距离进行扩展
	pq := make(littleQueue, 1)
	// 把起点插入
	pq[0] = pointS{src, 0}
	heap.Init(&pq)
	// 记录一下从起点到某一个点的最小距离
	finalDistMap := make(map[byte]int)

	for pq.Len() > 0 {
		// 取头部当前已走的最小距离的点进行拓展
		t := heap.Pop(&pq).(pointS)
		// 如果当前是目的地址，那么步数就是最小的
		// 反证一下，假设存在一个更小的，那么肯定还没到目的点，如果到了，前面会插入到小根堆中，这次就不会取到大的
		//          如果没到，还得再走，距离会是那个点继续加，那么就不可能比当前更小
		if t.ch == dst {
			return t.step
		}
		// 从此点向外走，获取此点能到的最近的点的距离
		distMap := distMaps[t.ch]
		for i, v := range distMap {
			// 从起点到t再走到i的距离
			step := t.step + v
			// 如果记录的到i点距离更小，这个点就不走了，因为之前记录那一次走过了
			if d, ok := finalDistMap[i]; ok && d <= step {
				continue
			}
			finalDistMap[i] = step
			heap.Push(&pq, pointS{i, step})
		}
	}
	return -1
}

func main() {
	// 定义好距离矩阵
	distMaps := make(map[byte]map[byte]int)
	distMaps['A'] = make(map[byte]int)
	distMaps['A']['B'] = 6
	distMaps['A']['C'] = 3
	distMaps['B'] = make(map[byte]int)
	distMaps['B']['A'] = 6
	distMaps['B']['C'] = 2
	distMaps['B']['D'] = 5
	distMaps['C'] = make(map[byte]int)
	distMaps['C']['A'] = 3
	distMaps['C']['B'] = 2
	distMaps['C']['D'] = 3
	distMaps['C']['E'] = 4
	distMaps['D'] = make(map[byte]int)
	distMaps['D']['B'] = 5
	distMaps['D']['C'] = 3
	distMaps['D']['E'] = 2
	distMaps['D']['F'] = 3
	distMaps['E'] = make(map[byte]int)
	distMaps['E']['C'] = 4
	distMaps['E']['D'] = 2
	distMaps['E']['F'] = 5
	distMaps['F'] = make(map[byte]int)
	distMaps['F']['D'] = 3
	distMaps['F']['E'] = 5
	// A => F 最短路径为 A => C => D => F = 9
	fmt.Println(dijkstra('A', 'F', distMaps))
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

## 3. bfs 广度优先遍历

#### 1) 二叉树的广度优先遍历

```go
type TreeNode struct {
	value int
	left  *TreeNode
	right *TreeNode
}

func bfs(root *TreeNode) {
	var queue list.List
	queue.PushBack(&root)
	for queue.Len() > 0 {
		node := queue.Front().Value.(*TreeNode)
		queue.Remove(queue.Front())
		if node.left != nil {
			queue.PushBack(node.left)
		}
		if node.right != nil {
			queue.PushBack(node.right)
		}
		fmt.Println(node.value)	// 这里取值
	}
}
```

#### 2) 方格中找两点最短路径

- 两格之间步数为1，`#`作为墙不可走

```go
type pointT struct {
	x int
	y int
}

var (
	// 按照上下左右的相对位置设定，用于后面方便找四周的点
	kRoundPoints = [][]int{
		{0, -1},
		{0, 1},
		{-1, 0},
		{1, 0},
	}
)

// 返回从src到dst的最短路径长度，带层间隔版本
func bfsFloor(src pointT, dst pointT, grid []string) int {
	// 减小计算量，走过的路不再走，记录一下哪里走过了
	seen := make([][]bool, len(grid))
	for i := range seen {
		seen[i] = make([]bool, len(grid[0]))
	}
	// 源地址记录走过了，注意x是第二维的坐标
	seen[src.y][src.x] = true

	// 使用层数作为步数
	curDepth := 0
	var queue list.List
	// 插入源地址，作为第一层，使用nil作为层间隔
	queue.PushBack(src)
	queue.PushBack(nil)
	// 队列一定含有一个层间隔，不在头就在尾，如果只剩一个层间隔，说明没路可走
	for queue.Len() > 1 {
		tmp := queue.Front().Value
		queue.Remove(queue.Front())
		if tmp == nil {
			// 找到层间隔，说明当前层遍历完了，步数加一准备下一层
			curDepth++
			// 当前层遍历完，队列剩余的都是下一层，加入一个层间隔
			queue.PushBack(nil)
			continue
		}

		// 判断当前点是不是目标点，如果是，说明走到了，返回步数
		tx, ty := tmp.(pointT).x, tmp.(pointT).y
		if tx == dst.x && ty == dst.y {
			return curDepth
		}
		// 不是目标点，从此点出发，向四周走一下
		for i := range kRoundPoints {
			px, py := tx+kRoundPoints[i][0], ty+kRoundPoints[i][1]
			// 如果超出边界或者已经走过了或者碰到墙，就继续
			if py < 0 || py >= len(grid) || px < 0 || px >= len(grid[0]) || seen[py][px] || grid[py][px] == '#' {
				continue
			}
			// 这个点可以走，走上去，记录到队列中，作为下一层的起点
			seen[py][px] = true
			queue.PushBack(pointT{px, py})
		}
	}
	return -1
}

type pointST struct {
	x    int
	y    int
	step int
}

// 返回从src到dst的最短路径长度
func bfs(src pointST, dst pointST, grid []string) int {
	// 减小计算量，走过的路不再走，记录一下哪里走过了
	seen := make([][]bool, len(grid))
	for i := range seen {
		seen[i] = make([]bool, len(grid[0]))
	}
	// 源地址记录走过了，注意x是第二维的坐标
	seen[src.y][src.x] = true

	var queue list.List
	// 插入源地址
	queue.PushBack(src)
	for queue.Len() > 0 {
		tmp := queue.Front().Value.(pointST)
		queue.Remove(queue.Front())

		// 判断当前点是不是目标点，如果是，说明走到了，返回步数
		if tmp.x == dst.x && tmp.y == dst.y {
			return tmp.step
		}
		// 不是目标点，从此点出发，向四周走一下
		for i := range kRoundPoints {
			px, py := tmp.x+kRoundPoints[i][0], tmp.y+kRoundPoints[i][1]
			// 如果超出边界或者已经走过了或者碰到墙，就继续
			if py < 0 || py >= len(grid) || px < 0 || px >= len(grid[0]) || seen[py][px] || grid[py][px] == '#' {
				continue
			}
			// 这个点可以走，走上去，记录到队列中，作为下一层的起点
			seen[py][px] = true
			queue.PushBack(pointST{px, py, tmp.step+1})
		}
	}
	return -1
}

func main() {
	/*
		@ # . . *
		. . . # .
		# . . . .
	*/
	grid := []string{"@#..*", "...#.", "#...."}
	// @ 到 * 的最短距离为6
	fmt.Println(bfs(pointST{0, 0, 0}, pointST{4, 0, 0}, grid))
	fmt.Println(bfsFloor(pointT{0, 0}, pointT{4, 0}, grid))
}
```

## 4. 全排列算法（回溯法）

<img src = "2020_04_29_01.png">

- 原理是遍历替换首字母和后面的字符，替换到最后输出

```go
// 全排列
func permutation(str []byte, index int, f func(str []byte)) {
	if len(str) == index {
        // 这里输出结果
		f(str)
		return
	}

	// 不交换的场景
	permutation(str, index+1, f)
	// index对应位置向后交换
	for i := index + 1; i < len(str); i++ {
		str[i], str[index] = str[index], str[i]
		permutation(str, index+1, f)
		str[i], str[index] = str[index], str[i]
	}
}
```

## 5. 动态规划

- 动态规划一般是推导真实场景的某一状态和下一状态之间的关系
- 根据两个状态的关系列出方程，然后从第一个状态不断向状态n靠近

### 5.1. 背包算法

#### 1) 01背包

- 主要解决选元素的方案数的问题
- 思想是考虑选择出来的元素要装到背包中，每个元素都有装或者不装两个选择
- 背包本身的价值可以使用map进行枚举，每个元素选择是否装入背包，装入则背包价值增加
- 每个价值的方案数因为一个元素的加入可以列出状态转移方程

$$
count[价值] = count[不包含这个元素时已有的方案数] + f[加上这个元素可以达到价值对应的方案数]
$$

- 典型题目: [无平方子集](https://leetcode.cn/problems/count-the-number-of-square-free-subsets/)，对应[讲解](/bookPages/docs/leetcode/)

## 6. gcd 最大公约数算法

- 利用欧几里得算法，即辗转相除法

### 证明

- a可以表示成 $a = kb + r$（a，b，k，r皆为正整数，且 $r<b$ ），则 $r = a\ mod\ b$
- 假设d是a,b的一个公约数，记作 $d|a, d|b$，即a和b都可以被d整除。
- 而 $r = a - kb$，两边同时除以d， $r/d = a/d - kb/d = m$，由等式右边可知m为整数，因此 $d|r$
- 因此d也是 $b, a\ mod\ b$ 的公约数
- 假设d是 $b, a\ mod\ b$ 的公约数, 则 $d|b,d|(a - k \times b)$，k是一个整数
- 因此 $(a, b)$ 和 $(b,a\ mod\ b)$ 的公约数是一样的，其最大公约数也必然相等，得证

### 代码

```go
func gcd(a int, b int) int {
    // 当a为最大公约数时，计算后a = 0，b = a
	for a != 0 {
		a, b = b%a, a
	}
	return b
}
```

## 7. 向上向下取整算法（不使用除法和余数运算）

### 7.1. 2的幂次取整

#### 向下取整

- 直接二进制上与要取整的数减一取反就好了，如要对8也就是`0b1000`取整，就是与上`0b11111000`

#### 向上取整

- 由于需要有多余位就加一个，防止判断可以使用加上取整的数减一然后再对其取反相与
- 如要对8向上取整就是`(i + 0b00000111) & 0b11111000`

## 8. 选取中位数

- 可以使用两个堆，一个大根堆一个小根堆，中间的就是中位数

## 9. 按位从低到高找第一个1

- 取反加1，再和原来数字与一下

```go
func getFirst(i int) int {
	v := (^i) + 1
	return i & v
}
```

- 举个例子，`0b00110010`，取反`0b11001101`，加一`0b11001110`，与一下原来的数`0b00000010`
