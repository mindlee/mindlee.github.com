---
title: 斐波那契堆(Fibonacci heaps)
author: Wei Li
excerpt: 对于斐波那契堆上的各种可合并操作，关键思想是尽可能久地将工作推后。例如，当向斐波那契堆中插入新结点或合并两个斐波那契堆时，并不去合并树，而是将这个工作留给EXTRACT-MIN操作。
layout: post
permalink: /2011/09/29/fibonacci-heaps/
views:
  - 13741
categories:
  - 算法学习
tags:
  - 堆
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
斐波那契堆同二项堆一样,也是一种可合并堆。斐波那契堆的优势是：不涉及删除元素的操作仅需要O（1）的平摊运行时间（关于平摊分析的知识建议看《算法导论》第17章）。和二项堆一样，斐波那契堆由一组树构成。这种堆松散地基于二项堆，说松散是因为：如果不对斐波那契堆做任何DECREASE-KEY 或 DELETE 操作，则堆中每棵树就和二项树一样；但是如果执行这两种操作，在一些状态下必须要破坏二项树的特征，比如DECREASE-KEY或DELETE 后，有的树高为k，但是结点个数却少于2k。这种情况下，堆中的树不是二项树。

与二项堆相比，斐波那契堆同样是由一组最小堆有序树构成，但是斐波那契堆中的树都是有根而无序的，也就是说，单独的树满足最小堆特性，但是堆内树与树之间是无序的，如下图。

对于斐波那契堆上的各种可合并操作，关键思想是尽可能久地**将工作推后**。例如，当向斐波那契堆中插入新结点或合并两个斐波那契堆时，并不去合并树，而是将这个工作留给EXTRACT-MIN操作。
![Image][1]

###一、每个结点x的域：

1. 父节点p[x]
2. 指向任一子女的指针child[x]——结点x的子女被链接成一个环形双链表，称为x的子女表
3. 左兄弟left[x]
4. 右兄弟right[x]——当left[x] = right[x] = x时，说明x是独子。
5. 子女的个数degree[x]
6. 布尔值域mark[x]——标记是否失去了一个孩子
####结点ADT:

{% highlight c %}
//斐波那契结点ADT
struct FibonacciHeapNode {
    int key;       //结点
    int degree;    //度
    FibonacciHeapNode * left;  //左兄弟
    FibonacciHeapNode * right; //右兄弟
    FibonacciHeapNode * parent; //父结点
    FibonacciHeapNode * child;  //第一个孩子结点
    bool marked;           //是否被删除第1个孩子
};
typedef FibonacciHeapNode FibNode;
{% endhighlight %}
###二、堆结构ADT：
对于一个给定的斐波那契堆H，可以通过指向包含最小关键字的树根的指针min[H]来访问，这个结点被称为斐波那契堆中的最小结点。如果一个斐波那契堆H是空的，则min[H] = NIL. 在一个斐波那契堆中，所有树的根都通过left和right指针链接成一个环形的双链表，称为堆的根表。于是，指针min[H]就指向根表中具有最小关键字的结点。

####堆结构ADT：
{% highlight c %}
//斐波那契堆ADT
struct FibonacciHeap {
    int keyNum;   //堆中结点个数
    FibonacciHeapNode * min;//最小堆，根结点
    int maxNumOfDegree;   //最大度
    FibonacciHeapNode * * cons;//指向最大度的内存区域
};
 
typedef FibonacciHeap FibHeap;
{% endhighlight %}
###三、创建一个新的斐波那契堆

创建一个空的斐波那契堆，过程MAKE-FIB-HEAP 分配并返回一个斐波那契堆对象H；

{% highlight c %}
//初始化一个空的Fibonacci Heap
FibHeap * FibHeapMake() {
    FibHeap * heap = NULL;
    heap = (FibHeap *) malloc(sizeof(FibHeap));
    if (NULL == heap) {
        puts("Out of Space!!");
        exit(1);
    }
    memset(heap, 0, sizeof(FibHeap));
    return heap;
}
 
//初始化结点x
FibNode * FibHeapNodeMake() {
    FibNode * x = NULL;
    x = (FibNode *) malloc(sizeof(FibNode));
    if (NULL == x) {
        puts("Out of Space!!");
        exit(1);
    }
    memset(x, 0, sizeof(FibNode));
    x->left = x->right = x;
    return x;
}
{% endhighlight %}
###四、插入一个结点

简单说就是生成一个结点x，对结点的各域初始化，赋值，然后构造自身的环形双向链表后，将x加入H的根表中。 也就是说，结点x 成为一棵单结点的最小堆有序树，同时就是斐波那契堆中一棵无序二项树。                伪代码：
{% highlight c %}
FIB-HEAP-INSERT(H, x)
1  degree[x] ← 0
2  p[x] ← NIL
3  child[x] ← NIL
4  left[x] ← x
5  right[x] ← x
6  mark[x] ← FALSE
7  concatenate the root list containing x with root list H
8  if min[H] = NIL or key[x] < key[min[H]]
9     then min[H] ← x
10  n[H] ← n[H] + 1
{% endhighlight %}
如图是将关键字为21的结点插入斐波那契堆。该结点自成一棵最小堆有序树，从而被加入到根表中，成为根的左兄弟。
![Image][2]


{% highlight c %}
//堆结点x插入fibonacci heap中
void FibHeapInsert(FibHeap * heap, FibNode * x) {
    if (0 == heap->keyNum) {
        heap->min = x;
    } else {
        FibNodeAdd(x, heap->min);
        x->parent = NULL;
        if (x->key < heap->min->key) {
            heap->min = x;
        }
    }
    heap->keyNum++;
}
 
//将数组内的值插入Fibonacci Heap
void FibHeapInsertKeys(FibHeap * heap, int keys[], int keyNum) {
    for (int i = 0; i < keyNum; i++) {
        FibHeapInsertKey(heap, keys[i]);
    }
}
 
//将值插入Fibonacci Heap
static void FibHeapInsertKey(FibHeap * heap, int key) {
    FibNode * x = NULL;
    x = FibHeapNodeMake();
    x->key = key;
    FibHeapInsert(heap, x);
}
{% endhighlight %}
###五、合并两个斐波那契堆

不同于二项堆，这个操作在斐波那契堆里非常简单。仅仅简单地将H1和H2的两根表并置，然后确定一个新的最小结点。

伪代码：
{% highlight c %}
FIB-HEAP-UNION(H1, H2)
1  H ← MAKE-FIB-HEAP()
2  min[H] ← min[H1]
3  concatenate the root list of H2 with the root list of H
4  if (min[H1] = NIL) or (min[H2] ≠ NIL and min[H2] < min[H1])
5    then min[H] ← min[H2]
6  n[H] ← n[H1] + n[H2]
7  free the objects H1 and H2
8  return H
{% endhighlight %}
###六、抽取最小结点

前边说过，对根表中的树合并是推后到EXTRACT-MIN中的，所以抽取最小这个操作比较麻烦。该过程还用到一个辅助过程CONSOLIDATE。

伪代码：
{% highlight c %}
FIB-HEAP-EXTRACT-MIN(H)
1  z ← min[H]
2  if z ≠ NIL
3     then for each child x of z
4              do add x to the root list of H
5                 p[x] ← NIL
6          remove z from the root list of H
7          if z = right[z]
8             then min[H] ← NIL
9             else min[H] ← right[z]
10                  CONSOLIDATE(H)
11          n[H] ← n[H] – 1
12  return z
{% endhighlight %}
这个过程先使最小结点的每个子女都成为一个根，并将最小结点从根表中取出。然后，通过将度数相同的根链接起来，直至对应每个度数至多只有一个根来调整根表。FIB-HEAP-EXTRACT-MIN中，3~5行中使z的所有子女成为根（将他们放入根表）来从H中删除结点z，并在第6行中将z从根表中去掉。如果z为根节点中唯一的结点且没有子女，则第8行返回空即可；否则，让指针min[H]指向根表中的一个非z的结点（伪代码中为right[z])。这个min[H]只是临时值，并不是真正的最小结点。第9行之前程序执行过程如图a)~b)（一图胜千言）。

![Image][3]
![Image][4]

CONSOLIDATE过程要做的工作是：使每个度数的二项树唯一，也就是使每个根都有一个不同的degree值为止。对根表的合并过程是反复执行下面的步骤：

1. 在根表中找出两个具有相同度数的根x和y，且key[x] <= key[y].

2. 将y链接到x：将y从根表中移出，成为x的一个孩子。这个过程由FIB-HEAP-LINK完成。

伪代码：
{% highlight c %}
CONSOLIDATE(H)
1 for i ← 0 to D(n[H])
2      do A[i] ← NIL
3 for each node w in the root list of H
4      do x ← w
5         d ← degree[x]
6         while A[d] ≠ NIL
7             do y ← A[d]      ▹ Another node with the same degree as x.
8                if key[x] > key[y]
9                   then exchange x ↔ y
10                FIB-HEAP-LINK(H, y, x)
11                A[d] ← NIL
12                d ← d + 1
13         A[d] ← x
14 min[H] ← NIL
15 for i ← 0 to D(n[H])
16      do if A[i] ≠ NIL
17            then add A[i] to the root list of H
18                 if min[H] = NIL or key[A[i]] < key[min[H]]
19                    then min[H] ← A[i]
FIB-HEAP-LINK(H, y, x)
1  remove y from the root list of H
2  make y a child of x, incrementing degree[x]
3  mark[y] ← FALSE
{% endhighlight %}

这个过程中用到的数组是哈希辅助数组A[]，如果度数为i的树不存在，则A[i]为空。这个伪代码3~13行的工作就是使每个度数的二项树唯一，里边的while循环反复地将包含结点w的数的根x链接到与其相同度数的其他树根上，直到没有其他度数相同的根为止。14行清空旧的根表，第15~19行根据数组A重新构造根表，最终结果如图m)。

{% highlight c %}
//抽取最小结点
FibNode * FibHeapExtractMin(FibHeap * heap) {
    FibNode * x = NULL, * z = heap->min;
    if (z != NULL) {
 
        //删除z的每一个孩子
        while (NULL != z->child) {
            x = z->child;
            FibNodeRemove(x);
            if (x->right == x) {
                z->child = NULL;
            } else {
                z->child = x->right;
            }
            FibNodeAdd(x, z);//add x to the root list heap
            x->parent = NULL;
        }
 
        FibNodeRemove(z);
        if (z->right == z) {
            heap->min = NULL;
        } else {
            heap->min = z->right;
            FibHeapConsolidate(heap);
        }
        heap->keyNum--;
    }
    return z;
}
 
//合并左右相同度数的二项树
void FibHeapConsolidate(FibHeap * heap) {
    int D, d;
    FibNode * w = heap->min, * x = NULL, * y = NULL;
    FibHeapConsMake(heap);//开辟哈希所用空间
    D = heap->maxNumOfDegree + 1;
    for (int i = 0; i < D; i++) {
        *(heap->cons + i) = NULL;
    }
 
    //合并相同度的根节点，使每个度数的二项树唯一
    while (NULL != heap->min) {
        x = FibHeapMinRemove(heap);
        d = x->degree;
        while (NULL != *(heap->cons + d)) {
            y = *(heap->cons + d);
            if (x->key > y->key) {//根结点key最小
                swap(x, y);
            }
            FibHeapLink(heap, y, x);
            *(heap->cons + d) = NULL;
            d++;
        }
        *(heap->cons + d) = x;
    }
    heap->min = NULL;//原有根表清除
 
    //将heap->cons中结点都重新加到根表中，且找出最小根
    for (int i = 0; i < D; i++) {
        if (*(heap->cons + i) != NULL) {
            if (NULL == heap->min) {
                heap->min = *(heap->cons + i);
            } else {
                FibNodeAdd(*(heap->cons + i), heap->min);
                if ((*(heap->cons + i))->key < heap->min->key) {
                    heap->min = *(heap->cons + i);
                }//if(<)
            }//if-else(==)
        }//if(!=)
    }//for(i)
}
 
 
//将x根结点链接到y根结点
void FibHeapLink(FibHeap * heap, FibNode * x, FibNode *y) {
    FibNodeRemove(x);
    if (NULL == y->child) {
        y->child = x;
    } else {
        FibNodeAdd(x, y->child);
    }
    x->parent = y;
    y->degree++;
    x->marked = false;
}
 
//开辟FibHeapConsolidate函数哈希所用空间
static void FibHeapConsMake(FibHeap * heap) {
    int old = heap->maxNumOfDegree;
    heap->maxNumOfDegree = int(log(heap->keyNum * 1.0) / log(2.0)) + 1;
    if (old < heap->maxNumOfDegree) {
        //因为度为heap->maxNumOfDegree可能被合并,所以要maxNumOfDegree + 1
        heap->cons = (FibNode **) realloc(heap->cons, 
            sizeof(FibHeap *) * (heap->maxNumOfDegree + 1));
        if (NULL == heap->cons) {
            puts("Out of Space!");
            exit(1);
        }
    }
}
 
//将堆的最小结点移出，并指向其有兄弟
static FibNode *FibHeapMinRemove(FibHeap * heap) {
    FibNode *min = heap->min;
    if (heap->min == min->right) {
        heap->min = NULL;
    } else {
        FibNodeRemove(min);
        heap->min = min->right;
    }
    min->left = min->right = min;
    return min;
}
{% endhighlight %}

###七、减小一个关键字

减小关键字操作最大的难点是，如果减小后的结点破坏了最小堆的性质，如何维护斐波那契堆的性质。这里用到一个操作：级联剪枝（Cascading Cut）。减小关键字的代码流程基本就是：如果减小后的结点破坏了最小堆性质，则把它切下来(cut)，即从所在双向链表中删除，并将其插入到由最小树根节点形成的双向链表中，然后再从parent[x]到所在树根节点递归执行级联剪枝。

关于级联剪枝，《[数据结构](http://book.douban.com/subject/1886174/)》中的解释：

>由于增加了删除和关键字减值操作，所以，F堆中的最小树就不一定必须是二项树了。事实上，可能存在度为k却只有k + 1（原书是k + 1，应该是k – 1吧）个结点的最小树。为了保证每个度为k的最小树至少包含ck个结点（c > 1)， 每次执行删除操作和关键字减值操作后，还必须进行级联剪枝操作。为此，为每个结点增加一个布尔类型的child_cut域（即本文里的marked)。child_cut域的值仅对那些不是最小树树根的结点有意义。对于不是最小树树根的结点x， x的child_cut域为TRUE，当且仅当在最近一次x成为其当前父结点的儿子之后，x的一个儿子被删除。这就意味着，在执行删除最小元素中，每次连接两棵最小树时，关键字值较大的根结点的child_cut域应该赋值为FALSE。更进一步地说，一旦删除操作或关键字减值操作将最小树的非根结点q从其所在双向链表中删除时，则调用级联剪枝操作。**在执行级联剪枝操作过程中，检查从被删除结点q的父节点p开始，到被删节点的最近的child_cut域为FALSE的祖先结点的路径。对在该路径上所有child_cut域为TRUE的非根结点，将其从所在的双向链表中删除，并将其加入到F堆的最小树的根节点组成的双向链表中。**如果该路径上存在child_cut域为FALSE的结点 ,则将其该域的值修改为TRUE。

伪代码：
{% highlight c %}
FIB-HEAP-DECREASE-KEY(H, x, k)
1  if k > key[x]
2     then error "new key is greater than current key"
3  key[x] ← k
4  y ← p[x]
5  if y ≠ NIL and key[x] < key[y]
6     then CUT(H, x, y)
7          CASCADING-CUT(H, y)
8  if key[x] < key[min[H]]
9      then min[H] ← x
CUT(H, x, y)
1 remove x from the child list of y, decrementing degree[y]
2 add x to the root list of H
3 p[x] ← NIL
4 mark[x] ← FALSE
CASCADING-CUT(H, y)
1 z ← p[y]
2 if z ≠ NIL
3    then if mark[y] = FALSE
4            then mark[y] ← TRUE
5            else CUT(H, y, z)
6                 CASCADING-CUT(H, z)
{% endhighlight %}
图中：a),b)46减小为5；  c),d),e)35减小为5
![Image][5]

级联剪切的过程很明了，我当时看的时候最烦的问题是:

**为什么要进行级联剪切，级联剪切要干什么?**

	如果仅仅要切除父结点y的一个结点x，则仅仅需要把结点x加入到根结点所在双向链表中，再检测y是否marked == true即可，这是因为斐波那契中的树并不一定是二项树，近似二项树也可以。当删除y的第二个结点时，对在该路径上所有marked域为TRUE的非根结点，将其从所在的双向链表中删除，并将其加入到F堆的最小树的根节点组成的双向链表中，即只有在删除同一个结点偶数个孩子时，才要进行级联剪枝，来维护二项树性质，奇数个时（即一个），对树影响不大，莫管它，只标记一下即可。

**为什么偶数个的时候要递归往上删除？**

    二项树中在深度为i处恰有Cik个结点(I = 0, 1, 2, ……, k)。试着如果不进行级联剪枝，就可以发现，稍微删得结点超过两三个，最后的树就会不成样子，毫无章法。但是如果进行了级联剪枝，在偶数个结点时进行级联剪切时，原来是C30 = 1, C31 = 3, C32 = 3, C33 = 1, 减少两个结点关键字后，变为：C20 = 0，C21 = 2， C22 = 1;二项式是对称的，所以，偶数个结点时进行级联剪枝可以保证类似上边的正好使二项式减少一个数量级。

{% highlight c %}
//减小一个关键字
void FibHeapDecrease(FibHeap * heap, FibNode * x, int key) {
    FibNode * y = x->parent;
    if (x->key < key) {
        puts("new key is greater than current key!");
        exit(1);
    }
    x->key = key;
 
    if (NULL != y && x->key < y->key) {
        //破坏了最小堆性质，需要进行级联剪切操作
        FibHeapCut(heap, x, y);
        FibHeapCascadingCut(heap, y);
    }
    if (x->key < heap->min->key) {
        heap->min = x;
    }
}
 
//切断x与父节点y之间的链接，使x成为一个根
static void FibHeapCut(FibHeap * heap, FibNode * x, FibNode * y) {
    FibNodeRemove(x);
    renewDegree(y, x->degree);
    if (x == x->right) {
        y->child = NULL;
    } else {
        y->child = x->right;
    }
    x->parent = NULL;
    x->left = x->right = x;
    x->marked = false;
    FibNodeAdd(x, heap->min);
}
 
//级联剪切
static void FibHeapCascadingCut(FibHeap * heap, FibNode * y) {
    FibNode * z = y->parent;
    if (NULL != z) {
        if (y->marked == false) {
            y->marked = true;
        } else {
            FibHeapCut(heap, y, z);
            FibHeapCascadingCut(heap, z);
        }
    }
}
 
//修改度数
void renewDegree(FibNode * parent, int degree) {
    parent->degree -= degree;
    if (parent-> parent != NULL) {
        renewDegree(parent->parent, degree);
    }
}
{% endhighlight %}

###八、删除一个结点

伪代码：
{% highlight c %}
FIB-HEAP-DELETE(H, x)
1 FIB-HEAP-DECREASE-KEY(H, x, -∞)
2 FIB-HEAP-EXTRACT-MIN(H)
{% endhighlight %}
过程很简单，先减小直到min[H], 然后直接剔除最小值即可


{% highlight c %}
//删除结点
void FibHeapDelete(FibHeap * heap, FibNode * x) {
    FibHeapDecrease(heap, x, INT_MIN);
    FibHeapExtractMin(heap);
}

{% endhighlight %}


完整代码：（参考[这里](http://blog.csdn.net/golden_shadow/article/details/6216921)，[这里](http://download.csdn.net/detail/Ture010Love/3567665)，[这里](http://blog.csdn.net/ture010love/article/details/6738394) ）

{% highlight c %}
/*
The keyNum = 10
 (1)  (11)  (10)  (9)  (7)  (6)  (5)  (4)  (3)  (2)
 
抽取最小值1之后：
The keyNum = 9
 (2 (3)  (6 (7)  (9 (10) ) )  (4 (5) ) )  (11)
 
查找11成功,减小到8后：
The keyNum = 9
 (2 (3)  (6 (7)  (9 (10) ) )  (4 (5) ) )  (8)
 
删除7成功:
The keyNum = 8
 (2 (3)  (6 (9 (10) ) )  (4 (5) ) )  (8)
 
*/
//说明：
//代码中Fibonacci Heap 用变量heap表示
//结点通常用x，y等表示
#include<iostream>
#include<cstdio>
#include<cstdlib>
#include<cmath>
#include<climits>
using namespace std;
 
//斐波那契结点ADT
struct FibonacciHeapNode {
    int key;       //结点
    int degree;    //度
    FibonacciHeapNode * left;  //左兄弟
    FibonacciHeapNode * right; //右兄弟
    FibonacciHeapNode * parent; //父结点
    FibonacciHeapNode * child;  //第一个孩子结点
    bool marked;           //是否被删除第1个孩子
};
 
typedef FibonacciHeapNode FibNode;
 
//斐波那契堆ADT
struct FibonacciHeap {
    int keyNum;   //堆中结点个数
    FibonacciHeapNode * min;//最小堆，根结点
    int maxNumOfDegree;   //最大度
    FibonacciHeapNode * * cons;//指向最大度的内存区域
};
 
typedef FibonacciHeap FibHeap;
 
 
/*****************函数申明*************************/
//将x从双链表移除
inline void FibNodeRemove(FibNode * x);
 
//将x堆结点加入y结点之前(循环链表中)
void FibNodeAdd(FibNode * x, FibNode * y);
 
//初始化一个空的Fibonacci Heap
FibHeap * FibHeapMake() ;
 
//初始化结点x
FibNode * FibHeapNodeMake();
 
//堆结点x插入fibonacci heap中
void FibHeapInsert(FibHeap * heap, FibNode * x);
 
//将数组内的值插入Fibonacci Heap
void FibHeapInsertKeys(FibHeap * heap, int keys[], int keyNum);
 
//将值插入Fibonacci Heap
static void FibHeapInsertKey(FibHeap * heap, int key);
 
//抽取最小结点
FibNode * FibHeapExtractMin(FibHeap * heap);
 
//合并左右相同度数的二项树
void FibHeapConsolidate(FibHeap * heap);
 
//将x根结点链接到y根结点
void FibHeapLink(FibHeap * heap, FibNode * x, FibNode *y);
 
//开辟FibHeapConsolidate函数哈希所用空间
static void FibHeapConsMake(FibHeap * heap);
 
//将堆的最小结点移出，并指向其有兄弟
static FibNode *FibHeapMinRemove(FibHeap * heap);
 
//减小一个关键字
void FibHeapDecrease(FibHeap * heap, FibNode * x, int key);
 
//切断x与父节点y之间的链接，使x成为一个根
static void FibHeapCut(FibHeap * heap, FibNode * x, FibNode * y);
 
//级联剪切
static void FibHeapCascadingCut(FibHeap * heap, FibNode * y);
 
//修改度数
void renewDegree(FibNode * parent, int degree);
 
//删除结点
void FibHeapDelete(FibHeap * heap, FibNode * x);
 
//堆内搜索关键字
FibNode * FibHeapSearch(FibHeap * heap, int key);
 
//被FibHeapSearch调用
static FibNode * FibNodeSearch(FibNode * x, int key);
 
//销毁堆
void FibHeapDestory(FibHeap * heap);
 
//被FibHeapDestory调用
static void FibNodeDestory(FibNode * x);
 
//输出打印堆
static void FibHeapPrint(FibHeap * heap);
 
//被FibHeapPrint调用
static void FibNodePrint(FibNode * x);
/************************************************/
 
 
//将x从双链表移除
inline void FibNodeRemove(FibNode * x) {
    x->left->right = x->right;
    x->right->left = x->left;
}
 
/*   
将x堆结点加入y结点之前(循环链表中)
    a …… y
    a …… x …… y
*/
inline void FibNodeAdd(FibNode * x, FibNode * y) {
    x->left = y->left;
    y->left->right = x;
    x->right = y;
    y->left = x;
}
 
//初始化一个空的Fibonacci Heap
FibHeap * FibHeapMake() {
    FibHeap * heap = NULL;
    heap = (FibHeap *) malloc(sizeof(FibHeap));
    if (NULL == heap) {
        puts("Out of Space!!");
        exit(1);
    }
    memset(heap, 0, sizeof(FibHeap));
    return heap;
}
 
//初始化结点x
FibNode * FibHeapNodeMake() {
    FibNode * x = NULL;
    x = (FibNode *) malloc(sizeof(FibNode));
    if (NULL == x) {
        puts("Out of Space!!");
        exit(1);
    }
    memset(x, 0, sizeof(FibNode));
    x->left = x->right = x;
    return x;
}
 
//堆结点x插入fibonacci heap中
void FibHeapInsert(FibHeap * heap, FibNode * x) {
    if (0 == heap->keyNum) {
        heap->min = x;
    } else {
        FibNodeAdd(x, heap->min);
        x->parent = NULL;
        if (x->key < heap->min->key) {
            heap->min = x;
        }
    }
    heap->keyNum++;
}
 
//将数组内的值插入Fibonacci Heap
void FibHeapInsertKeys(FibHeap * heap, int keys[], int keyNum) {
    for (int i = 0; i < keyNum; i++) {
        FibHeapInsertKey(heap, keys[i]);
    }
}
 
//将值插入Fibonacci Heap
static void FibHeapInsertKey(FibHeap * heap, int key) {
    FibNode * x = NULL;
    x = FibHeapNodeMake();
    x->key = key;
    FibHeapInsert(heap, x);
}
 
//抽取最小结点
FibNode * FibHeapExtractMin(FibHeap * heap) {
    FibNode * x = NULL, * z = heap->min;
    if (z != NULL) {
 
        //删除z的每一个孩子
        while (NULL != z->child) {
            x = z->child;
            FibNodeRemove(x);
            if (x->right == x) {
                z->child = NULL;
            } else {
                z->child = x->right;
            }
            FibNodeAdd(x, z);//add x to the root list heap
            x->parent = NULL;
        }
 
        FibNodeRemove(z);
        if (z->right == z) {
            heap->min = NULL;
        } else {
            heap->min = z->right;
            FibHeapConsolidate(heap);
        }
        heap->keyNum--;
    }
    return z;
}
 
//合并左右相同度数的二项树
void FibHeapConsolidate(FibHeap * heap) {
    int D, d;
    FibNode * w = heap->min, * x = NULL, * y = NULL;
    FibHeapConsMake(heap);//开辟哈希所用空间
    D = heap->maxNumOfDegree + 1;
    for (int i = 0; i < D; i++) {
        *(heap->cons + i) = NULL;
    }
 
    //合并相同度的根节点，使每个度数的二项树唯一
    while (NULL != heap->min) {
        x = FibHeapMinRemove(heap);
        d = x->degree;
        while (NULL != *(heap->cons + d)) {
            y = *(heap->cons + d);
            if (x->key > y->key) {//根结点key最小
                swap(x, y);
            }
            FibHeapLink(heap, y, x);
            *(heap->cons + d) = NULL;
            d++;
        }
        *(heap->cons + d) = x;
    }
    heap->min = NULL;//原有根表清除
 
    //将heap->cons中结点都重新加到根表中，且找出最小根
    for (int i = 0; i < D; i++) {
        if (*(heap->cons + i) != NULL) {
            if (NULL == heap->min) {
                heap->min = *(heap->cons + i);
            } else {
                FibNodeAdd(*(heap->cons + i), heap->min);
                if ((*(heap->cons + i))->key < heap->min->key) {
                    heap->min = *(heap->cons + i);
                }//if(<)
            }//if-else(==)
        }//if(!=)
    }//for(i)
}
 
 
//将x根结点链接到y根结点
void FibHeapLink(FibHeap * heap, FibNode * x, FibNode *y) {
    FibNodeRemove(x);
    if (NULL == y->child) {
        y->child = x;
    } else {
        FibNodeAdd(x, y->child);
    }
    x->parent = y;
    y->degree++;
    x->marked = false;
}
 
//开辟FibHeapConsolidate函数哈希所用空间
static void FibHeapConsMake(FibHeap * heap) {
    int old = heap->maxNumOfDegree;
    heap->maxNumOfDegree = int(log(heap->keyNum * 1.0) / log(2.0)) + 1;
    if (old < heap->maxNumOfDegree) {
        //因为度为heap->maxNumOfDegree可能被合并,所以要maxNumOfDegree + 1
        heap->cons = (FibNode **) realloc(heap->cons, 
            sizeof(FibHeap *) * (heap->maxNumOfDegree + 1));
        if (NULL == heap->cons) {
            puts("Out of Space!");
            exit(1);
        }
    }
}
 
//将堆的最小结点移出，并指向其有兄弟
static FibNode *FibHeapMinRemove(FibHeap * heap) {
    FibNode *min = heap->min;
    if (heap->min == min->right) {
        heap->min = NULL;
    } else {
        FibNodeRemove(min);
        heap->min = min->right;
    }
    min->left = min->right = min;
    return min;
}
 
//减小一个关键字
void FibHeapDecrease(FibHeap * heap, FibNode * x, int key) {
    FibNode * y = x->parent;
    if (x->key < key) {
        puts("new key is greater than current key!");
        exit(1);
    }
    x->key = key;
 
    if (NULL != y && x->key < y->key) {
        //破坏了最小堆性质，需要进行级联剪切操作
        FibHeapCut(heap, x, y);
        FibHeapCascadingCut(heap, y);
    }
    if (x->key < heap->min->key) {
        heap->min = x;
    }
}
 
//切断x与父节点y之间的链接，使x成为一个根
static void FibHeapCut(FibHeap * heap, FibNode * x, FibNode * y) {
    FibNodeRemove(x);
    renewDegree(y, x->degree);
    if (x == x->right) {
        y->child = NULL;
    } else {
        y->child = x->right;
    }
    x->parent = NULL;
    x->left = x->right = x;
    x->marked = false;
    FibNodeAdd(x, heap->min);
}
 
//级联剪切
static void FibHeapCascadingCut(FibHeap * heap, FibNode * y) {
    FibNode * z = y->parent;
    if (NULL != z) {
        if (y->marked == false) {
            y->marked = true;
        } else {
            FibHeapCut(heap, y, z);
            FibHeapCascadingCut(heap, z);
        }
    }
}
 
//修改度数
void renewDegree(FibNode * parent, int degree) {
    parent->degree -= degree;
    if (parent-> parent != NULL) {
        renewDegree(parent->parent, degree);
    }
}
 
//删除结点
void FibHeapDelete(FibHeap * heap, FibNode * x) {
    FibHeapDecrease(heap, x, INT_MIN);
    FibHeapExtractMin(heap);
}
 
//堆内搜索关键字
FibNode * FibHeapSearch(FibHeap * heap, int key) {
    return FibNodeSearch(heap->min, key);
}
 
//被FibHeapSearch调用
static FibNode * FibNodeSearch(FibNode * x, int key) {
    FibNode * w = x, * y = NULL;
    if (x != NULL) {
        do { 
            if (w->key == key) {
                y = w;
                break;
            } else if (NULL != (y = FibNodeSearch(w->child, key))) {
                break;
            }
            w = w->right;
        } while (w != x);
    }
    return y;
}
 
//销毁堆
void FibHeapDestory(FibHeap * heap) {
    FibNodeDestory(heap->min);
    free(heap);
    heap = NULL;
}
 
//被FibHeapDestory调用
static void FibNodeDestory(FibNode * x) {
    FibNode * p = x, *q = NULL;
    while (p != NULL) {
        FibNodeDestory(p->child);
        q = p;
        if (p -> left == x) {
            p = NULL;
        } else {
            p = p->left;
        }
        free(q->right);
    }
}
 
//输出打印堆
static void FibHeapPrint(FibHeap * heap) {
    printf("The keyNum = %d\n", heap->keyNum);
    FibNodePrint(heap->min);
    puts("\n");
};
 
//被FibHeapPrint调用
static void FibNodePrint(FibNode * x) {
    FibNode * p = NULL;
    if (NULL == x) {
        return ;
    }
    p = x;
    do {
        printf(" (");
        printf("%d", p->key);
        if (p->child != NULL) {
            FibNodePrint(p->child);
        }
        printf(") ");
        p = p->left;
    }while (x != p);
}
 
int keys[10] = {1, 2, 3, 4, 5, 6, 7, 9, 10, 11};
 
int main() {
    FibHeap * heap = NULL;
    FibNode * x = NULL;
    heap = FibHeapMake();
    FibHeapInsertKeys(heap, keys, 10);
    FibHeapPrint(heap);
 
    x = FibHeapExtractMin(heap);
    printf("抽取最小值%d之后：\n", x->key);
    FibHeapPrint(heap);
 
    x = FibHeapSearch(heap, 11);
    if (NULL != x) {
        printf("查找%d成功,", x->key);
        FibHeapDecrease(heap, x, 8);
        printf("减小到%d后：\n", x->key);
        FibHeapPrint(heap);
    }
 
 
    x = FibHeapSearch(heap, 7);
    if (NULL != x) {
        printf("删除%d成功:\n", x->key);
        FibHeapDelete(heap, x);
        FibHeapPrint(heap);
    }
 
    FibHeapDestory(heap);
    return 0;
}
{% endhighlight %}

<hr/>
###一些评论问题

cucool说道：2012/04/24 22:03
: 请教搂主，书里没有提到如何在fibonacci heap上执行增加结点的值的操作，请问这该如何实现？和减少关键字的值操作时一样的吗？还是实现方式和过程不一样？
另外，如何分析增加关键字的值的运行时间？会是O(logn)吗？

谢谢呀，本人看的云里雾里，特来请教。

Wei Li说道：2012/04/25 10:08 
: 好久没碰算法，快生了，翻了一下书：
{% highlight c %}
FIB-HEAP-DECREASE-KEY(H, x, k)
1 if k > key[x]
2     then error "new key is greater than current key"
3 key[x] ← k
4 y ← p[x]
5 if y ≠ NIL and key[x] < key[y]
6     then CUT(H, x, y)
7         CASCADING-CUT(H, y)
8 if key[x] < key[min[H]]
9     then min[H] ← x
{% endhighlight %}

上边是减小值 x 的伪代码，y 是 x 的父结点；如果 x 是根结点，也就是 y == NIL， 则直接执行8、9行，否则执行5、6、7、8、9行。

增大值和减小值面临的问题应该是一样的。就是，破坏了最小堆的性质怎么办，破坏了性质，当然级联剪切了，我上边写过为什么级联剪切可以帮助维护最小堆了，增大值的操作只是判断过程稍微有点变化；

{% highlight c %}
FIB-HEAP-INCREASE-KEY(H, x, k)
1 if k < key[x]
2     then error "new key is lesser than current key"
3 key[x] ← k
4 y ← child[x]
5 if y ≠ NIL and key[x] > key[y]
6     then CUT(H, y, x)
7         CASCADING-CUT(H, x)
{% endhighlight %}
增大一个值 x，y 是 x 的子结点，如果 x 没有子女结点，也就是y == NIL，则万事大吉，什么也不用干，否则执行5、6、7行，当然，相比减少值操作，这里也不用判断 y 会不会是新的min[H]， 因为它够大了，比上边省了两行（8、9行）。

增大值应该和减小值的时间分析是一样的，减小结点的值是 O(1) 的平摊时间，增大应该也是。

FIB-HEAP-EXTRACT-MIN(H) 和 FIB-HEAP-DELETE(H, x) 的平摊运行时间才应该是O(lgn)，也就是抽取最小值和删除一个结点的操作。

关于分析运行时间，《算法导论》17章里讲到平摊分析，这里就是用的平摊分析的知识，具体我有些不大熟了，还得再去看看。。

其实，增大结点值和减小结点值的操作可以合并为一个函数 → 改变结点值，里边把上边两个合并了就行了。

cucool说道：2012/04/25 20:39  

: 谢谢楼主。不过，第6行后，是否还要执行 CUT(H, x, p(x))这一步？因为如果x之前已被marked的话。
至于运行时间，因为任意结点n,其最大度数是logn，是不是说明，增值操作的平摊代价有可能是logn?这是和减值不一样的地方。

另外，我能否用如下操作，实现增值？
先把X增值后，然后和其孩子比较大小，如果大，就和孩子交换，重复这个操作，直到不违反最小堆性质，操作就算完成。如果是这个实现方式，那从结点到叶子，最多有logn的高度，所以平摊代价会是logn?
另，不知道以上两种操作，是否会改变原始堆的其他操作的平摊代价？

Wei Li说道：2012/04/26 20:31 

: “第6行后，是否还要执行 CUT(H, x, p(x))这一步？因为如果x之前已被marked的话。至于运行时间，因为任意结点n, 其最大度数是logn，是不是说明，增值操作的平摊代价有可能是logn? 这是和减值不一样的地方。”

x是否已被marked……好像没有关系吧，不管 x 如何，CUT的时候，都会mark[x] = false，因为它要去当根结点了，后续操作都是对p(x)进行的呀？和减值没区别。。这里的运行时间，不能简单的那么算，因为它的一个操作会造成其他影响，比如切掉一个点，单纯的代价是1，但是会引起结点mark值改变，mark的改变又会决定是否还会继续切下去。。在比较确定的排序之类里可以那么算，因为不会有其他影响，这里的一个操作影响是不确定的，所以要平摊分析。。

具体分析，《算法导论》300页最后两段 + 301页有简要的分析，《数据结构与算法分析——C语言描述》338页专门证明这个的。。下边是书中的摘录：（后一本，图片识别软件抓取的）

> “对于一次DecreaseKey操作所需要的实际时间是1加上在该操作期间所执行的级联切除的次数。由于级联切除的次数可能会比O(1)多很多，为此我们需要用位势的损失来作为补偿，树的棵数实际上是随着每次级联切除而增加，因此我们必须增强位势函数，使它包含某种在级联切除期间能够递减的成分。注意，我们不能从位势函数中抛开树的棵数，因为这样就不能够证明Merge操作的时间界了。级联切除引起被标记的节点的个数的减少，因为每个被级联切除分出的节点都变成了未标记的根。由于级联切除花费1个单元的实际时间并将树的位势增加1，因此我们将每个标记的节点算作2个位势单位。利用这种方法，我们就获得了一种消除级联切除机会的机会。”

>位势是斐波那契堆的集合中树的棵树加上两倍的标记节点数。像通常一样，初始的位势为0并且总是非负的。于是，经过一系列操作之后，总的的摊还时间则是总的实际时间的一个上界。

>最后考虑DecreaseKey操作。令C为级联切除的次数。DecreaseKey的实际花费为C+1，它是所执行的切除的总数。第一次（非级联）切除创建一棵新树从而使位势增1。每次级联切除都建立一棵新树，但却把一个标记节点转变成未标记的（根）节点，合计每次级联切除有一个单位的净损失。最后一次切除也可能把一个未标记节点，转变成标记节点，这就使得位势增加2。因此，位势总的变化最多是3-C，把实际时间和位势变化加起来则得到总和为4，即O（1）。“

“另外，我能否用如下操作，实现增值？
先把X增值后，然后和其孩子比较大小，如果大，就和孩子交换，重复这个操作，直到不违反最小堆性质，操作就算完成。如果是这个实现方式，那从结点到叶子，最多有logn的高度，所以平摊代价会是logn?

另，不知道以上两种操作，是否会改变原始堆的其他操作的平摊代价？”

这个应该可以的，不过，如果这样操作，斐波那契堆就没有任何优势了。

eagle说道：2012/06/01 16:32 
{% highlight c %}
static void FibNodeDestory(FibNode * x) {
    FibNode * p = x, *q = NULL;
    while (p != NULL) {
        FibNodeDestory(p->child);
        q = p;
        if (p == x) {
            p = NULL;
        } else {
            p = p->left;
        }
        free(q->right);
    }
}

q = p;
if (p == x) {
{% endhighlight %}
这应该有问题吧，第一步就相等了，双向链表就没法删除了。

酷~行天下说道：2012/06/23 13:13  
: 我看了下，这里确实有问题。
应该改成这样：

{% highlight c %}
//被FibHeapDestory调用 
static void FibNodeDestory(FibNode * x) { 
    FibNode * p = x, *q = NULL;
    while (p != NULL) { 
        FibNodeDestory(p->child); 
        q = p; 
        if (p ->left == x) { //检测是否是独立叶子节点
            p = NULL; 
        } else { 
            p = p->left; 
        } 
        free(q->right); 
    } 
} 
{% endhighlight %}
if语句用来判断是否是独立的叶子结点，如果是，p = NULL，递归这里结束，然后回溯到它的父结点；如果不是独立的叶子节点，则接着用函数处理它的左兄弟，自己销毁掉。

[1]: /uploads/2011/09/FibHeapPic.jpg
[2]: /uploads/2011/09/FibHeapInsertPic.jpg
[3]: /uploads/2011/09/FibExtractPic1.jpg
[4]: /uploads/2011/09/FibHeapExtractPic2.jpg
[5]: /uploads/2011/09/FibHeapDecrease.jpg



