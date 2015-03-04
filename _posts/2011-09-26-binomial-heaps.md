---
title: 二项堆(Binomial Heaps)
author: 酷~行天下
excerpt: 二项堆（Binomial Heap）是一种类似于二叉堆(Binary Heap)的堆结构。与二叉堆相比，其优势是可以快速合并两个堆，因此它属于可合并堆（Mergeable Heap）。后边要学的斐波那契堆也是可合并堆。一个二项堆由一组二项树所构成，这里的二项树(Binomial Trees)不同于二叉树(Binary Trees)。二叉树是“左孩子，右孩子”的表示方法，而二项树是“左孩子，右兄弟”的表示方法，在了解二项堆之前，需要先了解它的前验知识：二叉树，多叉树，二项树等数据结构，以及它们的不同。
layout: post
permalink: /2011/09/26/binomial-heaps/
views:
  - 12168
categories:
  - 算法学习
tags:
  - 堆
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
二项堆（Binomial Heap）是一种类似于二叉堆(Binary Heap)的堆结构。与二叉堆相比，其优势是可以快速合并两个堆，因此它属于可合并堆（Mergeable Heap）。后边要学的斐波那契堆也是可合并堆。一个二项堆由一组二项树所构成，这里的二项树(Binomial Trees)不同于二叉树(Binary Trees)。二叉树是“左孩子，右孩子”的表示方法，而二项树是“左孩子，右兄弟”的表示方法，在了解二项堆之前，需要先了解它的前验知识：二叉树，多叉树，二项树等数据结构，以及它们的不同。

###一、二叉树   

二叉树的结构如右图，用域p、left和right来存放指向二叉树T中的父亲、左儿子和右儿子的指针。如果P[x]=NIL，则x为根。如果left[x]=NIL，x无左儿子，右儿子也类似。整个树T的根由属性root[T]指向，如果root[T]=NIL，则树为空。二叉树中每个结点都最多有两个子女，即“左孩子，右孩子”。二项树与二叉树最大的不同是，二项树是一棵分支数无限制的有根树(Rooted trees with unbounded branching， 下图图示)，而二叉树最多有两个分叉。
![Image](/uploads/2011/09/Binary_Tree.jpg)

###二、分支树无限制的有根数

对于多叉树，也许你会想，结点的分支增多，直接用child1, child2…, childk来取代left和right域不就得了。但是如果树中结点的子女数是无限制的，那么就不行了。因为我们不知道事先要开多大的域，通常情况下，为了防止最坏情况，会开一个很大的界，这种情况下如果多数结点只有少数子女，则会浪费大量内存空间。而左图这种“左孩子，有兄弟”的方法可以完美解决这个问题。它的每个结点包含域p[x](顶端)，left-child[x](指向结点x的最左孩子), right-child[x](指向x紧右边的兄弟)。也就是说，一个结点的所有孩子形成一个链表，这样中间就可以任意加入结点了，在加入结点到树中时，动态开辟空间，这样就不会浪费任何空间了。
![Image](/uploads/2011/09/ordered-tree.jpg)

###三、二项树

而二项树则是一种特殊的多分支有序树。结构如右图，(下图a)二项树Bk是一种递归定义的有序树。二项树B0只包含一个结点。二项树Bk由两个子树Bk-1连接而成：其中一棵树的根是另一棵树的根的最左孩子。**二项树**的性质有：

1. 共有2k个结点。

2. 树的高度为k。

3. 在深度i处恰有Cik个结点。

4. 根的度数（子女的个数）为k，它大于任何其他结点的度数；如果根的子女从左到右的编号设为k-1, k-2, …, 0，子女i是子树Bi的根。
![Image](/uploads/2011/09/Binomial-Trees.jpg)

如图中图(b)表示二项树B0至B4中示出了各结点的深度。图(c)是以另外一种方式来看二项树Bk。PS : 在一棵包含n个结点的二项树中，任意结点的最大度数为lgn.

###四、二项堆

前边已经说过一个二项堆由一组二项树所构成，这里的二项树需要满足下列条件：

1）H中的每个二项树遵循**最小堆**的性质：结点的关键字大于等于其父结点的关键字。

2）对于任意非负整数k，在H中至多有一棵二项树的根具有度数k。

对于性质2，任意高度最多有一棵二项树，这样就可以用二项树的集合唯一地表示任意大小的二项堆，比如13个结点的二项堆H，13的二进制表示为（1101），故H包含了最小堆有序二项树B3， B2和B0， 他们分别有8， 4， 2， 1个结点，即共有13个结点。如下图（另外：二项堆中各二项树的根被组织成一个链表，称之为根表）
![Image](/uploads/2011/09/BinHeapTree13.jpg)

####1）二项树ADT：

{% highlight c %}
typedef struct BinHeapNode BinNode;
typedef struct BinHeapNode * BinHeap;
typedef struct BinHeapNode * Position;
 
//结点ADT
struct BinHeapNode {
    int key;
    int degree;
    Position parent;
    Position leftChild;
    Position sibling;
};
{% endhighlight %}
####2）创建一个新二项堆

MAKE-BINOMIAL-HEAP分配并返回一个对象H，且head[H]=NIL，运行时间为O（1）

{% highlight c %}
BinHeap MakeBinHeap() {  
    BinHeap heap = NULL;   
    heap = (BinHeap) malloc(sizeof(BinNode));
    if (heap == NULL) {
        puts("Out of the Space");
        exit(1);
    }
    memset(newHeap, 0, sizeof(BinNode));  
    return heap;  
}  
{% endhighlight %}
####3）寻找最小关键字

遍历根表，找出根表中关键字最小的结点。伪代码：

{% highlight c %}
1  y ← NIL
2  x ← head[H]
3  min ← ∞
4  while x ≠ NIL
5     do if key[x] < min
6           then min ← key[x]
7                y ← x
8         x ← sibling[x]
9  return y
{% endhighlight %}
代码实现：
{% highlight c %}
//返回最小根节点的指针
BinHeap BinHeapMin(BinHeap heap) {
    Position y = NULL, x = heap;
    int min = INT_MAX;
     
    while (x != NULL) {
        if (x->key < min) {
            min = x->key;
            y = x;
        }
        x = x->sibling;
    }
    return y;
}     
{% endhighlight %}    
####4）合并两个二项堆

合并包含三个函数：

BINOMIAL-LINK，连接操作，即将两棵根节点度数相同的二项树Bk-1连接成一棵Bk。伪代码：
{% highlight c %}
BINOMIAL-LINK(y, z)
1  p[y] ← z
2  sibling[y] ← child[z]
3  child[z] ← y
4  degree[z] ← degree[z] + 1
{% endhighlight %}
BINOMIAL-HEAP-MERGE ，将H1和H2的根表合并成一个按度数的单调递增次序排列的链表。

BINOMIAL-HEAP-UNION，反复连接根节点的度数相同的各二项树。伪代码：
{% highlight c %}
1  H ← MAKE-BINOMIAL-HEAP()
2  head[H] ← BINOMIAL-HEAP-MERGE(H1, H2)
3  free the objects H1 and H2 but not the lists they point to
4  if head[H] = NIL
5     then return H
6  prev-x ← NIL
7  x ← head[H]
8  next-x ← sibling[x]
9  while next-x ≠ NIL
10      do if (degree[x] ≠ degree[next-x]) or
                (sibling[next-x] ≠ NIL and degree[sibling[next-x]] = degree[x])
11            then prev-x ← x                                ▹ Cases 1 and 2
12                 x ← next-x                                ▹ Cases 1 and 2
13            else if key[x] ≤ key[next-x]
14                    then sibling[x] ← sibling[next-x]          ▹ Case 3
15                         BINOMIAL-LINK(next-x, x)               ▹ Case 3
16                    else if prev-x = NIL                        ▹ Case 4
17                            then head[H] ← next-x              ▹ Case 4
18                            else sibling[prev-x] ← next-x       ▹ Case 4
19                         BINOMIAL-LINK(x, next-x)               ▹ Case 4
20                         x ← next-x                            ▹ Case 4
21         next-x ← sibling[x]
22  return H
{% endhighlight %}
合并操作分为两个阶段：

第一阶段：执行BINOMIAL-HEAP-MERGE，将两个堆H1和H2的根表合并成一个链表H，它按度数排序成单调递增次序。MERGE的时间复杂度O(logn)。n为H1和H2的结点总数。（对于每一个度数值，可能有两个根与其对应，所以第二阶段要把这些相同的根连起来）。


第二阶段：将相等度数的根连接起来，直到每个度数至多有一个根时为止。执行过程中，合并的堆H的根表中至多出现三个根具有相同的度数。（MERGE后H中至多出现两个根具有相同的度数，但是将两个相同度数的根的二项树连接后，可能与后面的至多两棵二项树出现相同的度数的根，因此至多出现三个根具有相同的度数）

第二阶段根据当前遍历到的根表中的结点x，分四种情况考虑:

**Case1**：degree[x] != degree[sibling[x]]。此时，不需要做任何变化，将指针向根表后移动即可。(下图示a)

**Case2**：degree[x] == degree[sibling[x]] == degree[sibling[sibling[x]]]。此时，仍不做变化，将指针后移。(下图示b)

Case3 & Case4：degree[x] = degree[sibling[x]] != degree[sibling[sibling[x]]] (下图示c和d)

**Case3**：key[x] <= key[sibling[x]]。此时，将sibling[x]连接到x上。

**Case4**：key[x] > key[sibling[x]]。此时，将x连接到sibling[x]上。

复杂度：O(logn)， 四个过程变化情况：
![Image](/uploads/2011/09/BinHeapUnionExplainPic.jpg)
完整的一个合并Sample：
![Image](/uploads/2011/09/BinHeapUnionSample1.jpg)
![Image](/uploads/2011/09/BinHeapUnionSample2.jpg)
整个合并操作代码实现：

{% highlight c %}
//两个堆合并
BinHeap BinHeapUnion(BinHeap &H1, BinHeap &H2) {
    Position heap = NULL, pre_x = NULL, x = NULL, next_x = NULL;
    heap = BinHeapMerge(H1, H2);
    if (heap == NULL) {
        return heap;
    }
     
    pre_x = NULL;
    x = heap;
    next_x = x->sibling;
 
    while (next_x != NULL) {
        if ((x->degree != next_x->degree) ||//Cases 1 and 2
            ((next_x->sibling != NULL) && (next_x->degree == next_x->sibling->degree))) {
                pre_x = x;
                x = next_x;
        } else if (x->key <= next_x->key) {//Cases 3
            x->sibling = next_x->sibling;
            BinLink(next_x, x);
        } else {//Cases 4
            if (pre_x == NULL) {
                heap = next_x;
            } else {
                pre_x->sibling = next_x;
            }
            BinLink(x, next_x);
            x = next_x;
        }
        next_x = x->sibling;
    }
    return heap;
}
 
//将H1, H2的根表合并成一个按度数的单调递增次序排列的链表
BinHeap BinHeapMerge(BinHeap &H1, BinHeap &H2) {
    //heap->堆的首地址，H3为指向新堆根结点 
    BinHeap heap = NULL, firstHeap = NULL, secondHeap = NULL,
        pre_H3 = NULL, H3 = NULL;
 
    if (H1 != NULL && H2 != NULL){
        firstHeap = H1;
        secondHeap = H2;
        //整个while，firstHeap, secondHeap, pre_H3, H3都在往后顺移
        while (firstHeap != NULL && secondHeap != NULL) {
            if (firstHeap->degree <= secondHeap->degree) {
                H3 = firstHeap;
                firstHeap = firstHeap->sibling;
            } else {
                H3 = secondHeap;
                secondHeap = secondHeap->sibling;
            }
 
            if (pre_H3 == NULL) {
                pre_H3 = H3;
                heap = H3;
            } else {
                pre_H3->sibling = H3;
                pre_H3 = H3;
            }
            if (firstHeap != NULL) {
                H3->sibling = firstHeap;
            } else {
                H3->sibling = secondHeap;
            }
        }//while
    } else if (H1 != NULL) {
        heap = H1;
    } else {
        heap = H2;
    }
    H1 = H2 = NULL;
    return heap;
}
 
//使H2成为H1的父节点
void BinLink(BinHeap &H1, BinHeap &H2) {
    H1->parent = H2;
    H1->sibling = H2->leftChild;
    H2->leftChild = H1;
    H2->degree++;
}
{% endhighlight %}
####5）插入一个结点

先构造一个只包含一个结点的二项堆，再将此二项堆与原二项堆合并, 伪代码：

{% highlight c %}
BINOMIAL-HEAP-INSERT(H, x)
1  H′ ← MAKE-BINOMIAL-HEAP()
2  p[x] ← NIL
3  child[x] ← NIL
4  sibling[x] ← NIL
5  degree[x] ← 0
6  head[H′] ← x
7  H ← BINOMIAL-HEAP-UNION(H, H′)
{% endhighlight %}
代码实现(for循环内即插入过程)：

{% highlight c %}
//用数组内的值建堆
BinHeap MakeBinHeapWithArray(int keys[], int n) {
    BinHeap heap = NULL, newHeap = NULL;
    for (int i = 0; i < n; i++) {
        newHeap = (BinHeap) malloc(sizeof(BinNode));
        if (newHeap == NULL) {
            puts("Out of the Space");
            exit(1);
        }
        memset(newHeap, 0, sizeof(BinNode));
        newHeap->key = keys[i];
        if (NULL == heap) {
            heap = newHeap;
        } else {
            heap = BinHeapUnion(heap, newHeap);
            newHeap = NULL;
        }
    }
    return heap;
}
{% endhighlight %}
####6）抽取有最小关键字的结点

从根表中找到最小关键字的结点，将以该结点为根的整棵二项树从堆取出，删除取出的二项树的根，将其剩下的子女倒序排列，组成了一个新的二项堆，再与之前的二项堆合并。

伪代码：
{% highlight c %}
BINOMIAL-HEAP-EXTRACT-MIN(H)
1  find the root x with the minimum key in the root list of H,
            and remove x from the root list of H
2  H′ ← MAKE-BINOMIAL-HEAP()
3  reverse the order of the linked list of x’s children,
            and set head[H′] to point to the head of the resulting list
4  H ← BINOMIAL-HEAP-UNION(H, H′)
5  return x
{% endhighlight %}
代码实现：

{% highlight c %}
//抽取有最小关键字的结点
BinHeap BinHeapExtractMin(BinHeap &heap) {
    BinHeap pre_y = NULL, y = NULL, x = heap;
    int min = INT_MAX;
    while (x != NULL) {
        if (x->key < min) {
            min = x->key;
            pre_y = y;
            y = x;
        }
        x = x->sibling;
    }
 
    if (y == NULL) {
        return NULL;
    }
 
    if (pre_y == NULL) {
        heap = heap->sibling;
    } else {
        pre_y->sibling = y->sibling;
    }
 
    //将y的子结点指针reverse
    BinHeap H2 = NULL, p = NULL;
    x = y->leftChild;
    while (x != NULL) {
        p = x;
        x = x->sibling;
        p->sibling = H2;
        H2 = p;
        p->parent = NULL;
    }
 
    heap = BinHeapUnion(heap, H2);
    return y;
}
{% endhighlight %}
####7）减小关键字的值

伪代码：
{% highlight c %}
BINOMIAL-HEAP-DECREASE-KEY(H, x, k)
1 if k > key[x]
2    then error "new key is greater than current key"
3 key[x] ← k
4 y ← x
5 z ← p[y]
6 while z ≠ NIL and key[y] < key[z]
7     do exchange key[y] ↔ key[z]
8        ▸ If y and z have satellite fields, exchange them, too.
9        y ← z
10        z ← p[y]
{% endhighlight %}
减小关键字的过程类似维护最小堆结构，key[y]与y的父结点z的关键字作比较。如果y为根或者key[y] >= key[z],则该二项树已是最小堆有序。否则结点研究违反了最小堆有序，故将其关键字与其父节点z的关键字相交换，同时还要交换其他卫星数据。第6行的while循环这个过程。

看看图示：
![Image](/uploads/2011/09/BinHeapExtract.jpg)

代码实现：
{% highlight c %}      
//减少关键字的值
void BinHeapDecreaseKey(BinHeap heap, BinHeap x, int key) {
    if(key > x->key) {
        puts("new key is greaer than current key");
        exit(1); //不为降键
    }
    x->key = key;
 
    BinHeap z = NULL, y = NULL;
    y = x;
    z = x->parent;
    while(z != NULL && z->key > y->key) {
        swap(z->key, y->key);
        y = z;
        z = y->parent;
    }
}
{% endhighlight %}
####8）删除一个关键字

伪代码：
{% highlight c %}
BINOMIAL-HEAP-DELETE(H, x)
1  BINOMIAL-HEAP-DECREASE-KEY(H, x, -∞)
2  BINOMIAL-HEAP-EXTRACT-MIN(H)
{% endhighlight %}
可以看出，删除的原理非常简单，把关键字减小，让它到达根节点，然后剔除最小值即可。

可以看看图示：
![Image](/uploads/2011/09/BinHeapDelete.jpg)
代码实现：
{% highlight c %}
//删除一个关键字
BinHeap BinHeapDelete(BinHeap &heap, int key) {
    BinHeap x = NULL;
    x = BinHeapFind(heap, key);
    if (x != NULL) {
        BinHeapDecreaseKey(heap, x, INT_MIN);
        return BinHeapExtractMin(heap);
    }
    return x;
}
 
//找出一个关键字
BinHeap BinHeapFind(BinHeap &heap, int key) {
    Position p = NULL, x = NULL;
    p = heap;
    while (p != NULL) {
        if (p->key == key) {
            return p;
        } else {
            if((x =BinHeapFind(p->leftChild, key)) != NULL) {
                return x;
            }
            p = p->sibling;
        }
    }
    return NULL;
}
{% endhighlight %}
完整代码实现：

{% highlight c %}
/*
第一个二叉堆H1:
 (41)  (28 (33) )  (7 (15 (25) )  (12) )
 
第二个二叉堆H2:
 (55)  (24 (50) )  (2 (3 (8 (10 (44) )  (29) )  (6 (37) )  (18) )  (17 (32 (45)
)  (31) )  (23 (30) )  (48) )
 
合并H1,H2后，得到H3:
 (41 (55) )  (7 (24 (28 (33) )  (50) )  (15 (25) )  (12) )  (2 (3 (8 (10 (44) )
 (29) )  (6 (37) )  (18) )  (17 (32 (45) )  (31) )  (23 (30) )  (48) )
 
用于测试提取和删除的二叉堆H4:
 (27 (42) )  (11 (17 (38) )  (18) )  (1 (8 (14 (23 (26) )  (29) )  (12 (16) )  (
25) )  (10 (37 (41) )  (28) )  (13 (77) )  (6) )
 
抽取最小的值1后：
 (6)  (13 (27 (42) )  (77) )  (8 (10 (11 (17 (38) )  (18) )  (37 (41) )  (28) )
 (14 (23 (26) )  (29) )  (12 (16) )  (25) )
 
抽取最小的值6后：
 (13 (27 (42) )  (77) )  (8 (10 (11 (17 (38) )  (18) )  (37 (41) )  (28) )  (14
(23 (26) )  (29) )  (12 (16) )  (25) )
 
抽取最小的值8后：
 (25)  (12 (16) )  (10 (13 (14 (23 (26) )  (29) )  (27 (42) )  (77) )  (11 (17 (
38) )  (18) )  (37 (41) )  (28) )
 
删除12后：
 (16 (25) )  (10 (13 (14 (23 (26) )  (29) )  (27 (42) )  (77) )  (11 (17 (38) )
 (18) )  (37 (41) )  (28) ) 
 */
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cstdlib>
#include<climits>
using namespace std;
 
typedef struct BinHeapNode BinNode;
typedef struct BinHeapNode * BinHeap;
typedef struct BinHeapNode * Position;
 
//结点ADT
struct BinHeapNode {
    int key;
    int degree;
    Position parent;
    Position leftChild;
    Position sibling;
};
 
//用数组内的值建堆
BinHeap MakeBinHeapWithArray(int keys[], int n);
 
//两个堆合并
BinHeap BinHeapUnion(BinHeap &H1, BinHeap &H2);
 
//将H1, H2的根表合并成一个按度数的单调递增次序排列的链表
BinHeap BinHeapMerge(BinHeap &H1, BinHeap &H2);
 
//使H2成为H1的父节点
void BinLink(BinHeap &H1, BinHeap &H2);
 
//返回最小根节点的指针
BinHeap BinHeapMin(BinHeap heap);
 
//减少关键字的值
void BinHeapDecreaseKey(BinHeap heap, BinHeap x, int key);
 
//删除一个关键字
BinHeap BinHeapDelete(BinHeap &heap, int key);
 
//找出一个关键字
BinHeap BinHeapFind(BinHeap &heap, int key);
 
//打印输出堆结构
void PrintBinHeap(BinHeap heap);
 
//销毁堆
void DestroyBinHeap(BinHeap &heap);
 
 
//用数组内的值建堆
BinHeap MakeBinHeapWithArray(int keys[], int n) {
    BinHeap heap = NULL, newHeap = NULL;
    for (int i = 0; i < n; i++) {
        newHeap = (BinHeap) malloc(sizeof(BinNode));
        if (newHeap == NULL) {
            puts("Out of the Space");
            exit(1);
        }
        memset(newHeap, 0, sizeof(BinNode));
        newHeap->key = keys[i];
        if (NULL == heap) {
            heap = newHeap;
        } else {
            heap = BinHeapUnion(heap, newHeap);
            newHeap = NULL;
        }
    }
    return heap;
}
 
//两个堆合并
BinHeap BinHeapUnion(BinHeap &H1, BinHeap &H2) {
    Position heap = NULL, pre_x = NULL, x = NULL, next_x = NULL;
    heap = BinHeapMerge(H1, H2);
    if (heap == NULL) {
        return heap;
    }
     
    pre_x = NULL;
    x = heap;
    next_x = x->sibling;
 
    while (next_x != NULL) {
        if ((x->degree != next_x->degree) ||//Cases 1 and 2
            ((next_x->sibling != NULL) && (next_x->degree == next_x->sibling->degree))) {
                pre_x = x;
                x = next_x;
        } else if (x->key <= next_x->key) {//Cases 3
            x->sibling = next_x->sibling;
            BinLink(next_x, x);
        } else {//Cases 4
            if (pre_x == NULL) {
                heap = next_x;
            } else {
                pre_x->sibling = next_x;
            }
            BinLink(x, next_x);
            x = next_x;
        }
        next_x = x->sibling;
    }
    return heap;
}
 
//将H1, H2的根表合并成一个按度数的单调递增次序排列的链表
BinHeap BinHeapMerge(BinHeap &H1, BinHeap &H2) {
    //heap->堆的首地址，H3为指向新堆根结点 
    BinHeap heap = NULL, firstHeap = NULL, secondHeap = NULL,
        pre_H3 = NULL, H3 = NULL;
 
    if (H1 != NULL && H2 != NULL){
        firstHeap = H1;
        secondHeap = H2;
        //整个while，firstHeap, secondHeap, pre_H3, H3都在往后顺移
        while (firstHeap != NULL && secondHeap != NULL) {
            if (firstHeap->degree <= secondHeap->degree) {
                H3 = firstHeap;
                firstHeap = firstHeap->sibling;
            } else {
                H3 = secondHeap;
                secondHeap = secondHeap->sibling;
            }
 
            if (pre_H3 == NULL) {
                pre_H3 = H3;
                heap = H3;
            } else {
                pre_H3->sibling = H3;
                pre_H3 = H3;
            }
            if (firstHeap != NULL) {
                H3->sibling = firstHeap;
            } else {
                H3->sibling = secondHeap;
            }
        }//while
    } else if (H1 != NULL) {
        heap = H1;
    } else {
        heap = H2;
    }
    H1 = H2 = NULL;
    return heap;
}
 
//使H2成为H1的父节点
void BinLink(BinHeap &H1, BinHeap &H2) {
    H1->parent = H2;
    H1->sibling = H2->leftChild;
    H2->leftChild = H1;
    H2->degree++;
}
 
//返回最小根节点的指针
BinHeap BinHeapMin(BinHeap heap) {
    Position y = NULL, x = heap;
    int min = INT_MAX;
     
    while (x != NULL) {
        if (x->key < min) {
            min = x->key;
            y = x;
        }
        x = x->sibling;
    }
    return y;
}
 
//抽取有最小关键字的结点
BinHeap BinHeapExtractMin(BinHeap &heap) {
    BinHeap pre_y = NULL, y = NULL, x = heap;
    int min = INT_MAX;
    while (x != NULL) {
        if (x->key < min) {
            min = x->key;
            pre_y = y;
            y = x;
        }
        x = x->sibling;
    }
 
    if (y == NULL) {
        return NULL;
    }
 
    if (pre_y == NULL) {
        heap = heap->sibling;
    } else {
        pre_y->sibling = y->sibling;
    }
 
    //将y的子结点指针reverse
    BinHeap H2 = NULL, p = NULL;
    x = y->leftChild;
    while (x != NULL) {
        p = x;
        x = x->sibling;
        p->sibling = H2;
        H2 = p;
        p->parent = NULL;
    }
 
    heap = BinHeapUnion(heap, H2);
    return y;
}
 
//减少关键字的值
void BinHeapDecreaseKey(BinHeap heap, BinHeap x, int key) {
    if(key > x->key) {
        puts("new key is greaer than current key");
        exit(1); //不为降键
    }
    x->key = key;
 
    BinHeap z = NULL, y = NULL;
    y = x;
    z = x->parent;
    while(z != NULL && z->key > y->key) {
        swap(z->key, y->key);
        y = z;
        z = y->parent;
    }
}
 
//删除一个关键字
BinHeap BinHeapDelete(BinHeap &heap, int key) {
    BinHeap x = NULL;
    x = BinHeapFind(heap, key);
    if (x != NULL) {
        BinHeapDecreaseKey(heap, x, INT_MIN);
        return BinHeapExtractMin(heap);
    }
    return x;
}
 
//找出一个关键字
BinHeap BinHeapFind(BinHeap &heap, int key) {
    Position p = NULL, x = NULL;
    p = heap;
    while (p != NULL) {
        if (p->key == key) {
            return p;
        } else {
            if((x =BinHeapFind(p->leftChild, key)) != NULL) {
                return x;
            }
            p = p->sibling;
        }
    }
    return NULL;
}
 
//打印输出堆结构
void PrintBinHeap(BinHeap heap) {
    if (NULL == heap) {
        return ;
    }
    Position p = heap;
 
    while (p != NULL) {
        printf(" (");
        printf("%d", p->key);
        //显示其孩子
        if(NULL != p->leftChild) {
            PrintBinHeap(p->leftChild);
        }
        printf(") ");
         
        p = p->sibling;
    }
}       
 
int kp1[8] = {12, 
               7, 25, 
              15, 28, 33, 41};
 
int kp2[20] = {18,
                3, 37,
                6, 8, 29, 10, 44, 30, 23, 2, 48, 31, 17, 45, 32, 24, 50, 55};
 
int kp4[23] = {37, 41,
               10, 28, 13, 77,
               1, 6, 16, 12, 25, 8, 14, 29, 26, 23, 18, 11, 17, 38, 42, 27};
int main() {
    BinHeap H1 = NULL;
    H1 = MakeBinHeapWithArray(kp1, 7);
    puts("第一个二叉堆H1:");
    PrintBinHeap(H1);
 
    BinHeap H2 = NULL;
    H2 = MakeBinHeapWithArray(kp2, 19);
    puts("\n\n第二个二叉堆H2:");
    PrintBinHeap(H2);
 
    BinHeap H3 = NULL;
    H3 = BinHeapUnion(H1, H2);
    puts("\n\n合并H1,H2后，得到H3:");
    PrintBinHeap(H3);
 
    BinHeap H4 = NULL;
    H4 = MakeBinHeapWithArray(kp4, 22);
    puts("\n\n用于测试提取和删除的二叉堆H4:");
    PrintBinHeap(H4);
 
    BinHeap extractNode = BinHeapExtractMin(H4);
    if (extractNode != NULL) {
        printf("\n\n抽取最小的值%d后：\n", extractNode->key);
        PrintBinHeap(H4);
    }
 
    extractNode = BinHeapExtractMin(H4);
    if (extractNode != NULL) {
        printf("\n\n抽取最小的值%d后：\n", extractNode->key);
        PrintBinHeap(H4);
    }
 
    extractNode = BinHeapExtractMin(H4);
    if (extractNode != NULL) {
        printf("\n\n抽取最小的值%d后：\n", extractNode->key);
        PrintBinHeap(H4);
    }
 
    BinHeapDelete(H4, 12);
    puts("\n\n删除12后：");
    PrintBinHeap(H4);
    return 0;
}
{% endhighlight %}
wiki里提供了两个二项堆实现的代码，不妨去看看：

[C语言实现版](http://www.cs.unc.edu/~bbb/#binomial_heaps)， [Python语言实现版](http://code.activestate.com/recipes/511508/)