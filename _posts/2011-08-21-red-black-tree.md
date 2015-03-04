---
title: 红黑树(Red Black Tree)
author: Wei Li
excerpt: 红黑树的每个结点至少包含五个域：color，key，left，right 和 parent，如果某结点没有子结点或者父结点，则该结点相应的指针(p)域包含值NIL，我们将这些 NIL 当作叶子结点.(图a)。
layout: post
permalink: /2011/08/21/red-black-tree/
views:
  - 5274
categories:
  - 算法学习
tags:
  - 数据结构
  - 树
  - 笔记
  - 算法
  - 算法导论
---
红黑树本质是二叉查找树的一种，它的性能高于普通的二叉查找树，即使是在最坏的情况下也能保证时间复杂度为O(lgn)。红黑树在每个结点上增加一个存储位表示结点的颜色（或红或黑）。通过对任何一条从根到叶子的路径上各个结点着色方式的限制，红黑树可以保证没有一条路径会比其他路径长出两倍，因而是接近平衡的。

红黑树的每个结点至少包含五个域：color，key，left，right 和 parent，如果某结点没有子结点或者父结点，则该结点相应的指针(p)域包含值NIL，我们将这些 NIL 当作叶子结点.(图a)。

在实际处理过程中，往往将最底层的孩子结点和根结点的父亲都指向同一个 NIL 结点，以便于处理红黑树代码中的边界条件（这里的哨兵NIL是一个与树内普通结点有相同域的对象，它的color域为BLACK，而它的其他域——parent, left, right, key,可以设置为任意允许的值），而将其它结点当作内结点。 (图b)

通常表示红黑树的方法是图(c),即忽略叶子与根部的父结点。(图c），图中，黑结点用黑色表示，红结点用浅阴影表示。
![Image](/uploads/2011/01/red_black_tree.jpg)
红黑树在很多地方都有应用。在C++ STL中，很多部分(目前包括set, multiset, map, multimap)应用了红黑树的变体(SGI STL中的红黑树有一些变化，这些修改提供了更好的性能，以及对set操作的支持)。

一棵树满足下边的红黑树性质，则它就是一棵红黑树：

- 性质1. 结点是红色或黑色。

- 性质2. 根是黑色。

- 性质3 每个叶结点是黑色的。

- 性质4 每个红色结点的两个子结点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色结点)

- 性质5. 从任一结点到其每个叶子的所有路径都包含相同数目的黑色结点。

红黑树数据结构：

{% highlight c %}
const int RED = 0;
const int BLACK = 1;
typedef struct RBTNode RBT;
typedef struct RBTNode * Position;
typedef struct RBTNode * SearchTree;
//红黑树数据结构 
struct  RBTNode{
    int key;
    int color;
    Position parent, left, right;
};
 
static RBT sentinel = {0, 1, 0, 0, 0}; //哨兵 
#define NIL &sentinel
{% endhighlight %}

红黑树的查找，最小值，最大值，前驱，后继操作和二叉查找树基本一样：

{% highlight c %}
/*红黑树的查找，最小，最大，前驱，后继操作和二叉查找树
几乎是一模一样的，直接复制过来，改了下名字和NULL 
*/
  
//红黑树查找
Position RBTreeSearch(SearchTree tree, int k) {
    if (tree == NIL || k == tree->key) {
        return tree;
    }
    if (k < tree->key) {
        return RBTreeSearch(tree->left, k);
    } else {
        return RBTreeSearch(tree->right, k);
    }
}
 
//返回二叉树最左边值（即最小值）的指针， 
Position RBTreeMinimim(SearchTree tree) { 
    while (tree->left != NIL) { 
        tree = tree->left; 
    } 
    return tree; 
} 
   
//返回二叉树最右边值（即最大值）的指针 
Position RBTreeMaximum(SearchTree tree) { 
    while (tree->right != NIL) { 
        tree = tree->right; 
    } 
    return tree; 
}
 
//中序遍历下找它的后继 
Position RBTreeSuccessor(Position x) { 
    if (x->right != NIL) { 
        return RBTreeMinimim(x->right); 
    } 
    Position y = x->parent; 
    while (y != NIL && x == y-> right) { 
        x = y; 
        y = y->parent; 
    } 
    return y; 
}
{% endhighlight %}

**旋转（Rotate）**：红黑树的插入和删除操作可能会破坏上边列出的红黑树性质，所以需要通过旋转，来保持红黑树性质，看看这幅图，仔细观察下，左图到右图是LeftRotate，右图到左图是RightRotate，旋转的意思是：整个图（其实就是x,y之间的链)以从左到右水平方向为轴，旋转180°，x，y位置就对调了，然后跟换结果子结点的指针就OK了。
![Image](/uploads/2011/08/Red_Black_Tree_Rotate.jpg)

左旋伪代码：
![Image](/uploads/2011/08/red_black_tree_left_rotate.jpg)

旋转代码实现：
{% highlight c %}
//左旋，旋转只有指针被改变，其他域保持不变 
void LeftRotate(SearchTree &tree, Position x) {
    Position y = x->right;
    //set y's z左子树 into x's 右子树 
    x->right = y->left;
    if (y->left != NIL) {
        y->left->parent = x;
    }
     
    //下面代码修改x的父节点与y的关系
    y->parent = x->parent;
    if (x->parent == NIL) {//如果x是根节点 
        tree = y;
    } else if (x == x->parent->left) {//如果x不是根节点
        x->parent->left = y;
    } else {
        x->parent->right = y;
    }
    //修改x与y的关系 
    y->left = x;
    x->parent = y;
}
 
//右旋，只要把左旋里的程序对称就行了
void RightRotate(SearchTree &tree, Position x) {
    Position y = x->left;
    //set y's 右子树 into x's 左子树 
    x->left = y->right;
    if (y->right != NIL) {
        y->right->parent = x;
    }
     
    //下面代码修改x的父节点与y的关系
    y->parent = x->parent;
    if (x->parent == NIL) {//如果x是根节点 
        tree = y;
    } else if (x == x->parent->left) {//如果x不是根节点
        x->parent->left = y;
    } else {
        x->parent->right = y;
    }
    //修改x与y的关系 
    y->right = x;
    x->parent = y;
}
{% endhighlight %}

**插入（RBTreeInsert）**：红黑树的插入过程大致是和二叉查找树的插入过程差不多的，只是略微修改了下，因为插入新结点可能会破坏红黑树性质，所以要调用一个辅助函数RbInsertFixup()来对结点重新着色并旋转。因此红黑树插入操作的重点是辅助函数RbInsertFixup()

插入操作伪代码：
![Image](/uploads/2011/08/Red_Black_Tree_Insert_code.jpg)

RBTreeInsert和BSTreeInsert区别有四点：

1. RBTreeInsert内所有NULL(代表空)都用NIL(代表一个结点)代替;

2. 14,15行设置z.left和z.right为NIL，来保持正确的树结构。

3. 16行将z着色为红色

4. z设置为红色可能会违反某条红黑性质，所以在17行调用RBInsertFixup(T, x)来保持红黑性质

插入有三种情况，所以相应的RBInsertFixup(T, x)case1,2,3分别处理三种情况。
![Image](/uploads/2011/08/Red_Black_Tree_Insert_pic.jpg)

如图:

**Case1** : z的叔叔y是红色的，如图(a), (b)

A、B都为红色，破坏了性质4，因为要保证黑高度不变(性质5)，同时满足性质4，只能让A、D变为黑色，而B、C变为红色（567行)，这样就满足所有红黑性质了，然后将C设定为新的z，然后向上迭代。

**Case2**：z的叔叔y是黑色，而且z是右孩子

**Case3**：z的叔叔y是黑色，而且z是左孩子

从图(c)中可以看出，Case2和Case3仅仅是孩子位置不同，所以可以将Case2进行一个左旋转变为Case3，此时破坏了性质4，所以需要重新着色来保证性质4，将B着色黑色，C着色红色，此时性质4满足，但是性质5破坏了，所以再对C进行一次右旋，得到最终结果，此时满足所有性质。

PS：

1. 调用RBInsertFixup(T, x)时，只可能破坏红黑性质的第2,4（2->根结点为黑色，4->一个红结点不能有红子女)，其他的没影响。所以只需判断性质2,4即可。

2. 循环每次迭代都会有两种可能：指针z沿着树上移，或者执行某些旋转然后循环结束。

实例：
![Image](/uploads/2011/08/Red_Black_Tree_Insert_Sample.jpg)



插入代码实现：
{% highlight c %}
//插入的FixUp 
void RBTreeInsertFixUp(SearchTree &tree, Position z) {
    while (z->parent->color == RED) {
        if (z->parent = z->parent->parent->left) {//y在右边 
            Position y = z->parent->parent->right;//设定y 
            if (y->color == RED) {//Case1
                z->parent->color = BLACK;
                y->color = BLACK;
                z->parent->parent ->color = RED;
                z = z->parent->parent;
            } else {
                if (z == z->parent->right) {//Case2 
                    z = z->parent;//旋转会改变新结点的位置，所以要调整 
                    LeftRotate(tree, z); //case2->case3
                }
                z->parent->color = BLACK;//Case3 
                z->parent->parent->color = RED;
                RightRotate(tree, z->parent->parent); 
            }
        } else {//y在左边
            Position y = z->parent->parent->left;
            if (y->color == RED) {
                z->parent->color = BLACK;
                y->color = BLACK;
                z->parent->parent ->color = RED;
                z = z->parent->parent;
            } else {
                if (z == z->parent->left) {//Case2 
                    z = z->parent;//旋转会改变新结点的位置，所以要调整 
                    RightRotate(tree, z); //case2->case3
                }
                z->parent->color = BLACK;//Case3 
                z->parent->parent->color = RED;
                LeftRotate(tree, z->parent->parent);  
            }
        }//if-else
    }//while 
    tree->color = BLACK;
}//RBInsertFixUp
 
//插入操作 
void RBTreeInsert(SearchTree &tree, int k) { 
    Position y = NIL; 
    Position x = tree; 
    Position z = new RBT; 
    z->key = k; 
    z->parent = z->left = z->right = NIL;//初始化 
   
     //找到要插入的位置 
    while (x != NIL) { 
        y = x; 
        if (z->key < x->key) { 
            x = x->left; 
        } else { 
            x = x->right; 
        } 
    } 
    //和周边点增加关系 
    z->parent = y; 
    if (y == NIL) { 
        tree = z; 
    } else { 
        if (k < y->key) { 
            y->left = z; 
        } else { 
            y->right = z; 
        } 
    } 
    z->left = z->right = NIL;
    z->color = RED;
    RBTreeInsertFixUp(tree, z);
}   
{% endhighlight %}

红黑树插入，一个很酷的视频（配乐是The A La Menthe）

<div align="center">
<embed src="http://player.youku.com/player.php/sid/XMjI3NjM0MTgw/v.swf" allowFullScreen="true" quality="high" width="600" height="500" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash">
</div>

**删除（RBTreeDelete）**：删除操作和插入操作类似，也是在二叉查找树删除操作基础上略微修改。同样删除结点后可能会破坏红黑树性质，也要调用一个RBDeleteFixUp(T, x)函数。

删除操作伪代码：
![Image](/uploads/2011/08/RBTree_delete_code.jpg)

RBTreeDelete和BSTreeDelete有三点不同：

1. RBTreeDelete中所有的NULL替换为NIL(结点)。

2. BSTreeDelete第7行的判断，x是否为NIL去掉了，因为NIL本身就是结点，而NULL代表的是空

3. 16,17行，如果y是黑色的，则调用RBTreeDeleteFixUp(T, x)，如果y是红色，不进行操作，因为红黑性质没有破坏。（回想下二叉查找树三种删除情况，就可以发现，当y是红色时，红黑性质不会被破坏）


PS：当要删除的结点y为黑色时，会产生3个问题：

1. 如果y是根结点，而y的一个红色孩子成为了新的根结点，破坏了性质2

2. 当x和y->parent都是红色，破坏了性质4

3. 包含y的路径，黑高度降低1，破坏了性质5

解决方法：视x还有额外一层黑色，即为双重黑色或红黑，这里，一个结点的额外黑色反映在x指向它，而不是它的color属性。此时可以解决性质5，但是又破坏了性质1(根结点既不是黑，也不是红)，接下来过程就是，恢复性质1),2),4)。RBTreeDeleteFixUp(T, x)中while循环目标是：将额外的黑色沿树上移，知道x是根或者x是红色结点。

删除一个黑色结点，RBTreeDeleteFixUp需要处理四种情况：
![Image](/uploads/2011/08/RBTree_Delete_pic.jpg)

（图片说明：加黑的color属性为BLACK, 深阴影的结点color属性为RED, 浅阴影的结点color属性用c和c’表示，可为RED，也可为BLACK，即都有可能。)

对应伪代码中的Case1234；

**Case1**：x的兄弟w是红色的.

可以通过改变w和x->parent的颜色，再对x->parent做一次左旋，红黑性质保持，x的新兄弟new w是旋转之前w的某个孩子，为黑色，把情况1转换为情况2,3,或4。

**Case2**：x的兄弟w是黑色的，而且w的两个孩子都是黑色的。

w也为黑色，从x和w上去掉一重黑色，最后，w变为红色，为了补偿，x->parent新增一重黑色额外黑色。x->parent为new x，如果通过Case1进入Case2，则新结点x是红黑色的。

**Case3**：x的兄弟w是黑色的，且w的左孩子是红色，w的右孩子是黑色。

交换w和w->left的颜色，并对w进行右旋，x的新兄弟变为C（new w)，new w是有一个红色孩子的黑结点，这样，Case3转换为Case4。

**Case4**：x的兄弟w是黑色的，且w的右孩子是红色的。

做颜色修改，并对x->parent做一次旋转，可以去掉x的额外黑色，来把x变成单独的黑色，而不破坏红黑性质，将x置为根后，循环结束。

下边这张图是文章开始红黑树中，删除28的过程，可以看出是Case4
![Image](/uploads/2011/01/Red_Black_Tree_Delete_Case4.jpg)

删除实现代码：
{% highlight c %}
//删除FixUp
void RBTreeDeleteFixUp(SearchTree &tree, Position x) {
    while (x != tree && x->color == BLACK) {
        if (x == x->parent->left) {
            Position w = x->parent->right;
            if (w->color == RED) {//Case1 
                w->color = BLACK;
                x->parent->color = RED;
                LeftRotate(tree, x->parent);
                w = x->parent->right;
            }
            //____________Case2__________________ 
            if (w->left->color == BLACK && w->right->color == BLACK) {
                w->color = RED;
                x = x->parent;
            } else if (w->right->color == BLACK) {//Case3->Case4
                w->left->color = BLACK;
                w->color = RED;
                RightRotate(tree, w);
                w = x->parent->right;
            } else {//Case4
                w->color = x->parent->color;
                x->parent->color = BLACK;
                w->right->color = BLACK;
                LeftRotate(tree, x->parent);
                x = tree;
            }
        } else {//if-else完全对称 
            Position w = x->parent->left;
            if (w->color == RED) {//Case1 
                w->color = BLACK;
                x->parent->color = RED;
                RightRotate(tree, x->parent);
                w = x->parent->left;
            }
            //____________Case2__________________ 
            if (w->left->color == BLACK && w->right->color == BLACK) {
                w->color = RED;
                x = x->parent;
            } else if (w->left->color == BLACK) {//Case3->Case4
                w->left->color = BLACK; 
                w->color = RED;
                LeftRotate(tree, w);
                w = x->parent->left;
            } else {//Case4
                w->color = x->parent->color;
                x->parent->color = BLACK;
                w->left->color = BLACK;
                RightRotate(tree, x->parent);
                x = tree;
            }
        }
    }
    x->color = BLACK;
}
 
//删除操作 ,可以参考二叉查找树中的注释 
Position RBTreeDelete(SearchTree tree, Position z) { 
    Position x, y; 
     
    if (z->left == NIL || z->right == NIL) { 
        y = z;
    } else {
        y = RBTreeSuccessor(z); 
    } 
     
    if (y->left != NIL) { 
        x = y->left; 
    } else { 
        x = y->right; 
    } 
     
    //去掉了判断条件 
    x->parent = y->parent; 
     
    if (y->parent == NIL) { 
        tree = x; 
    } else if (y == y->parent->left) { 
        y->parent->left = x; 
    } else { 
        y->parent->right = x; 
    } 
     
    if (y != z) { 
        z->key = y->key; 
    } 
     
    if (y->color == BLACK) {
        RBTreeDeleteFixUp(tree, x);
    }
    return y;  
}
{% endhighlight %}

完整红黑树操作代码：(有点长)

{% highlight c %}
#include<iostream> 
#include<cstdio> 
using namespace std; 
   
const int RED = 0; 
const int BLACK = 1; 
typedef struct RBTNode RBT; 
typedef struct RBTNode * Position; 
typedef struct RBTNode * SearchTree; 
//红黑树数据结构 
struct  RBTNode{ 
    int key; 
    int color; 
    Position parent, left, right; 
}; 
   
static RBT sentinel = {0, 1, 0, 0, 0}; //哨兵 
#define NIL &sentinel 
   
/*红黑树的查找，最小，最大，前驱，后继操作和二叉查找树 
几乎是一模一样的，直接复制过来，改了下名字和NULL 
*/
   
//红黑树查找 
Position RBTreeSearch(SearchTree tree, int k) { 
    if (tree == NIL || k == tree->key) { 
        return tree; 
    } 
    if (k < tree->key) { 
        return RBTreeSearch(tree->left, k); 
    } else { 
        return RBTreeSearch(tree->right, k); 
    } 
} 
   
//返回二叉树最左边值（即最小值）的指针， 
Position RBTreeMinimim(SearchTree tree) { 
    while (tree->left != NIL) { 
        tree = tree->left; 
    } 
    return tree; 
}  
   
//返回二叉树最右边值（即最大值）的指针 
Position RBTreeMaximum(SearchTree tree) { 
    while (tree->right != NIL) { 
        tree = tree->right; 
    } 
    return tree; 
} 
   
//中序遍历下找它的后继 
Position RBTreeSuccessor(Position x) { 
    if (x->right != NIL) { 
        return RBTreeMinimim(x->right); 
    } 
    Position y = x->parent; 
    while (y != NIL && x == y-> right) { 
        x = y; 
        y = y->parent; 
    } 
    return y; 
}  
   
//左旋，旋转只有指针被改变，其他域保持不变 
void LeftRotate(SearchTree &tree, Position x) { 
    Position y = x->right; 
    //set y's z左子树 into x's 右子树 
    x->right = y->left; 
    if (y->left != NIL) { 
        y->left->parent = x; 
    } 
   
    //下面代码修改x的父节点与y的关系 
    y->parent = x->parent; 
    if (x->parent == NIL) {//如果x是根节点 
        tree = y; 
    } else if (x == x->parent->left) {//如果x不是根节点 
        x->parent->left = y; 
    } else { 
        x->parent->right = y; 
    } 
    //修改x与y的关系 
    y->left = x; 
    x->parent = y; 
} 
   
//右旋，只要把左旋里的程序对称就行了 
void RightRotate(SearchTree &tree, Position x) { 
    Position y = x->left; 
    //set y's 右子树 into x's 左子树 
    x->left = y->right; 
    if (y->right != NIL) { 
        y->right->parent = x; 
    } 
   
    //下面代码修改x的父节点与y的关系 
    y->parent = x->parent; 
    if (x->parent == NIL) {//如果x是根节点 
        tree = y; 
    } else if (x == x->parent->left) {//如果x不是根节点 
        x->parent->left = y; 
    } else { 
        x->parent->right = y; 
    } 
    //修改x与y的关系 
    y->right = x; 
    x->parent = y; 
}    
   
//插入的FixUp 
void RBTreeInsertFixUp(SearchTree &tree, Position z) { 
    while (z->parent->color == RED) { 
        if (z->parent = z->parent->parent->left) {//y在右边 
            Position y = z->parent->parent->right;//设定y 
            if (y->color == RED) {//Case1 
                z->parent->color = BLACK; 
                y->color = BLACK; 
                z->parent->parent ->color = RED; 
                z = z->parent->parent; 
            } else { 
                if (z == z->parent->right) {//Case2 
                    z = z->parent;//旋转会改变新结点的位置，所以要调整 
                    LeftRotate(tree, z); //case2->case3 
                } 
                z->parent->color = BLACK;//Case3 
                z->parent->parent->color = RED; 
                RightRotate(tree, z->parent->parent); 
            } 
        } else {//y在左边 
            Position y = z->parent->parent->left; 
            if (y->color == RED) { 
                z->parent->color = BLACK; 
                y->color = BLACK; 
                z->parent->parent ->color = RED; 
                z = z->parent->parent; 
            } else { 
                if (z == z->parent->left) {//Case2 
                    z = z->parent;//旋转会改变新结点的位置，所以要调整 
                    RightRotate(tree, z); //case2->case3 
                } 
                z->parent->color = BLACK;//Case3 
                z->parent->parent->color = RED; 
                LeftRotate(tree, z->parent->parent); 
            } 
        }//if-else 
    }//while 
    tree->color = BLACK; 
}//RBInsertFixUp 
   
//插入操作 
void RBTreeInsert(SearchTree &tree, int k) { 
    Position y = NIL; 
    Position x = tree; 
    Position z = new RBT; 
    z->key = k; 
    z->parent = z->left = z->right = NIL;//初始化  
   
     //找到要插入的位置 
    while (x != NIL) { 
        y = x; 
        if (z->key < x->key) { 
            x = x->left; 
        } else { 
            x = x->right; 
        } 
    } 
    //和周边点增加关系 
    z->parent = y; 
    if (y == NIL) { 
        tree = z; 
    } else { 
        if (k < y->key) { 
            y->left = z; 
        } else { 
            y->right = z; 
        } 
    } 
    z->left = z->right = NIL; 
    z->color = RED; 
    RBTreeInsertFixUp(tree, z); 
}    
   
//删除FixUp 
void RBTreeDeleteFixUp(SearchTree &tree, Position x) { 
    while (x != tree && x->color == BLACK) { 
        if (x == x->parent->left) { 
            Position w = x->parent->right; 
            if (w->color == RED) {//Case1 
                w->color = BLACK; 
                x->parent->color = RED; 
                LeftRotate(tree, x->parent); 
                w = x->parent->right; 
            } 
            //____________Case2__________________ 
            if (w->left->color == BLACK && w->right->color == BLACK) { 
                w->color = RED; 
                x = x->parent; 
            } else if (w->right->color == BLACK) {//Case3->Case4 
                w->left->color = BLACK; 
                w->color = RED; 
                RightRotate(tree, w); 
                w = x->parent->right; 
            } else {//Case4 
                w->color = x->parent->color; 
                x->parent->color = BLACK; 
                w->right->color = BLACK; 
                LeftRotate(tree, x->parent); 
                x = tree; 
            } 
        } else {//if-else完全对称 
            Position w = x->parent->left; 
            if (w->color == RED) {//Case1 
                w->color = BLACK; 
                x->parent->color = RED; 
                RightRotate(tree, x->parent); 
                w = x->parent->left; 
            } 
            //____________Case2__________________ 
            if (w->left->color == BLACK && w->right->color == BLACK) { 
                w->color = RED; 
                x = x->parent; 
            } else if (w->left->color == BLACK) {//Case3->Case4 
                w->left->color = BLACK; 
                w->color = RED; 
                LeftRotate(tree, w); 
                w = x->parent->left; 
            } else {//Case4 
                w->color = x->parent->color; 
                x->parent->color = BLACK; 
                w->left->color = BLACK; 
                RightRotate(tree, x->parent); 
                x = tree; 
            } 
        } 
    } 
    x->color = BLACK; 
} 
   
//删除操作 ,可以参考二叉查找树中的注释 
Position RBTreeDelete(SearchTree tree, Position z) { 
    Position x, y;  
   
    if (z->left == NIL || z->right == NIL) { 
        y = z; 
    } else { 
        y = RBTreeSuccessor(z); 
    }  
   
    if (y->left != NIL) { 
        x = y->left; 
    } else { 
        x = y->right; 
    }  
   
    //去掉了判断条件 
    x->parent = y->parent;  
   
    if (y->parent == NIL) { 
        tree = x; 
    } else if (y == y->parent->left) { 
        y->parent->left = x; 
    } else { 
        y->parent->right = x; 
    }  
   
    if (y != z) { 
        z->key = y->key; 
    }  
   
    if (y->color == BLACK) { 
        RBTreeDeleteFixUp(tree, x); 
    } 
    return y; 
} 
   
//中序遍历，输出颜色 
void InOrderTraverse(SearchTree tree) { 
    if (tree != NIL) { 
        InOrderTraverse(tree->left); 
        printf("\nkey = %2d ：color = ", tree->key); 
        if (tree->color == 0) { 
            printf("RED  " ); 
        } else { 
            printf("BLACK" ); 
        } 
   
        printf("  parent = %2d ", tree->parent->key); 
        if (tree->left != NIL) { 
            printf(" left  = %2d ", tree->left->key); 
        } 
        if (tree->right != NIL) { 
            printf(" right = %2d", tree->right->key); 
        } 
        InOrderTraverse(tree->right); 
    } 
}         
   
int main() { 
    SearchTree tree = NIL; 
    //构造文章最开头的红黑树 
    RBTreeInsert(tree, 26), RBTreeInsert(tree, 17); 
    RBTreeInsert(tree, 41), RBTreeInsert(tree, 14); 
    RBTreeInsert(tree, 21), RBTreeInsert(tree, 30); 
    RBTreeInsert(tree, 47), RBTreeInsert(tree, 10); 
    RBTreeInsert(tree, 16), RBTreeInsert(tree, 19); 
    RBTreeInsert(tree, 23), RBTreeInsert(tree, 28); 
    RBTreeInsert(tree, 38), RBTreeInsert(tree, 7); 
    RBTreeInsert(tree, 12), RBTreeInsert(tree, 15); 
    RBTreeInsert(tree, 20), RBTreeInsert(tree, 35); 
    RBTreeInsert(tree, 39), RBTreeInsert(tree, 3); 
   
    puts("中序遍历:"); 
    InOrderTraverse(tree);  
   
    Position locate28 = RBTreeSearch(tree, 28); 
    printf("\n\n28的后继是："); 
    printf("%d\n\n", RBTreeSuccessor(locate28) -> key); 
    puts("删除28后的中序遍历: "); 
    RBTreeDelete(tree, locate28); 
    InOrderTraverse(tree);  
   
    return 0; 
} 
/* 
运行结果： 
___________________________________________________________ 
中序遍历:
 
key =  3 ：color = RED    parent =  7
key =  7 ：color = BLACK  parent = 10  left  =  3
key = 10 ：color = RED    parent = 14  left  =  7  right = 12
key = 12 ：color = BLACK  parent = 10
key = 14 ：color = BLACK  parent = 17  left  = 10  right = 16
key = 15 ：color = RED    parent = 16
key = 16 ：color = BLACK  parent = 14  left  = 15
key = 17 ：color = RED    parent = 26  left  = 14  right = 21
key = 19 ：color = BLACK  parent = 21  right = 20
key = 20 ：color = RED    parent = 19
key = 21 ：color = BLACK  parent = 17  left  = 19  right = 23
key = 23 ：color = BLACK  parent = 21
key = 26 ：color = BLACK  parent =  0  left  = 17  right = 41
key = 28 ：color = BLACK  parent = 30
key = 30 ：color = RED    parent = 41  left  = 28  right = 38
key = 35 ：color = RED    parent = 28
key = 38 ：color = BLACK  parent = 30  left  = 35  right = 39
key = 39 ：color = RED    parent = 38
key = 41 ：color = BLACK  parent = 26  left  = 30  right = 47
key = 47 ：color = BLACK  parent = 41
 
28的后继是：30
 
删除28后的中序遍历:
 
key =  3 ：color = RED    parent =  7
key =  7 ：color = BLACK  parent = 10  left  =  3
key = 10 ：color = RED    parent = 14  left  =  7  right = 12
key = 12 ：color = BLACK  parent = 10
key = 14 ：color = BLACK  parent = 17  left  = 10  right = 16
key = 15 ：color = RED    parent = 16
key = 16 ：color = BLACK  parent = 14  left  = 15
key = 17 ：color = RED    parent = 26  left  = 14  right = 21
key = 19 ：color = BLACK  parent = 21  right = 20
key = 20 ：color = RED    parent = 19
key = 21 ：color = BLACK  parent = 17  left  = 19  right = 23
key = 23 ：color = BLACK  parent = 21
key = 26 ：color = BLACK  parent =  0  left  = 17  right = 41
key = 30 ：color = BLACK  parent = 38  right = 35
key = 35 ：color = RED    parent = 30
key = 38 ：color = RED    parent = 41  left  = 30  right = 39
key = 39 ：color = BLACK  parent = 38
key = 41 ：color = BLACK  parent = 26  left  = 38  right = 47
key = 47 ：color = BLACK  parent = 41 
PS：结合上边的图可知：程序运行结果正确 
*/
{% endhighlight %}
这里有一篇演示红黑树插入删除的神文： 《[红黑树从头至尾插入和删除结点的全程演示图](http://blog.csdn.net/v_JULY_v/article/details/6284050)》