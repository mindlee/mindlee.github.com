---
title: 二叉查找树
author: Wei Li
excerpt: 二叉查找树也是一种用于查找的数据结构，二叉查找树性质： 设x为二叉查找树中的一个结点。如果y是x的左子树中的一个结点，则key[y]<=key[x];如果y是x的右子树的一个结点，则key[y]>=key[x]。即结点的左孩子 < 结点 < 结点的右孩子
layout: post
permalink: /2011/08/20/binary-search-tree/
views:
  - 4776
categories:
  - 算法学习
tags:
  - 数据结构
  - 树
  - 笔记
  - 算法
  - 算法导论
---
二叉查找树也是一种用于查找的数据结构，二叉查找树性质：

设x为二叉查找树中的一个结点。如果y是x的左子树中的一个结点，则key[y]<=key[x];如果y是x的右子树的一个结点，则key[y]>=key[x]。即结点的左孩子 < 结点 < 结点的右孩子。如图：
![Image](/uploads/2011/08/BSTree.jpg)

{% highlight c %}
typedef struct BSTNode BST;//BST代表结点 
//BSTNode * == Position; Position代表指针 
typedef struct BSTNode * Position;
//BSTNode * == SearchTree; SearchTree代表二叉树的地址 
typedef struct BSTNode * SearchTree;
 
//二叉树数据结构 
struct BSTNode {
    int key;
    Position parent, left, right;
}; 
{% endhighlight %}
       
支持的操作：Search(查找操作), Minimum(最小值), Maximum(最大值), Predecessor(前趋), Successor(后继), Insert(插入), Delete(删除);

二叉查找树执行的基本操作的时间与树的高度成正比，含n个结点的完全二叉树，最坏情况运行时间时：O(lgn)，平均时间为θ(lgn);

二叉树的遍历操作有三种，先序遍历，中序遍历，后序遍历（可以去看看严奶奶的书，讲的很详细）


先序遍历：根结点在左右孩子之前输出;

上边图的先序遍历顺序：15->6->3->2->4->7->13->9->18->17->20

操作：若二叉树不空，则：

1. 访问根结点

2. 先序遍历左子树

3. 先序遍历右子树

{% highlight c %}
//先序遍历(递归版) 
void PreOrderTraverse(SearchTree tree) {
    if (tree != NULL) {
        printf("%d ", tree->key);
        PreOrderTraverse(tree->left);
        PreOrderTraverse(tree->right);
    }
}
{% endhighlight %}

中序遍历：按关键字从小到大顺序输出，即左孩子->根结点->右孩子

上边图的中序遍历顺序: 2->3->4->6->7->9->13->15->17->18->20

操作：若二叉树不空，则：

1. 中序遍历左子树

2. 访问根结点

3. 中序遍历右子树

{% highlight c %}
//中序遍历(递归版)
void InOrderTraverse(SearchTree tree) {
    if (tree != NULL) {
        InOrderTraverse(tree->left);
        printf("%d ", tree->key);
        InOrderTraverse(tree->right);
    }
}
{% endhighlight %}

后序遍历：根结点在左右孩子之后输出

上边图的后序遍历顺序：2->4->3->9->13->7->6->17->20->18->15

操作：若二叉树不空，则：

1. 后序遍历左子树

2. 后序遍历右子树

3. 访问根结点

{% highlight c %}
//后序遍历(递归版) 
void PostOrderTraverse(SearchTree tree) {
    if (tree != NULL) {
        if (tree->left)
        PostOrderTraverse(tree->left);
        PostOrderTraverse(tree->right);
        printf("%d ", tree->key);
    }
}
{% endhighlight %}

Search(查找操作)，从根结点开始进行，下降查找即可：

{% highlight c %}
//Search,查找操作(递归版本),递归版本简洁，易于理解，但效率不如非递归 
Position BSTreeSearch(SearchTree tree, int k) {
    if (tree == NULL || k == tree->key) {
        return tree;
    }
    if (k < tree->key) {
        return BSTreeSearch(tree->left, k);
    } else {
        return BSTreeSearch(tree->right, k);
    }
}
 
// Search操作，非递归版本 
Position BSTreeSearch1(SearchTree tree, int k) {
    while (tree != NULL && k != tree->key) {
        if (k < tree->key) {
            tree = tree->left;
        } else {
            tree = tree->right;
        }
    }
    return tree;
}
{% endhighlight %}

Minimum(最小值), Maximum(最大值)，最小就是最左边的值，最大就是最右边的值：

{% highlight c %}
//返回二叉树最左边值（即最小值）的指针， 
Position BSTreeMinimim(SearchTree tree) {
    while (tree->left != NULL) {
        tree = tree->left;
    }
    return tree;
}
 
//返回二叉树最右边值（即最大值）的指针 
Position BSTreeMaximum(SearchTree tree) {
    while (tree->right != NULL) {
        tree = tree->right;
    }
    return tree;
}
{% endhighlight %}

Predecessor(前趋), Successor(后继)，中序遍历下找它的前驱或后继，有两种情况（以后继为例） :

1. 右子树非空，则x的后继就是右子树的最左结点，文章开始图中15的后继结点是17

2. 右子树为空，且x有一个后继y，则y是x的最低祖父结点，且y的左孩子也是x的祖先，图中13的后继为15

为找到y,可以从x开始向上查找，直到找到某个是其父结点的左儿子的结点时为止，例如查找13的后继，找到x = 6, y = 15时查找终止，返回15，即，13的后继。

{% highlight c %}
//中序遍历下找它的后继
Position BSTreeSuccessor(Position x) {
    if (x->right != NULL) {
        return BSTreeMinimim(x->right);
    }
    Position y = x->parent;
    while (y != NULL && x == y-> right) {
        x = y;
        y = y->parent;
    }
    return y;
}
 
//找前驱是一样的道理，把right改为left就可以了
 Position BSTreePredecessor(Position x) {
    if (x->left != NULL) {
        return BSTreeMaximum(x->left);
    }
    Position y = x->parent;
    while (y != NULL && x == y-> left) {
        x = y;
        y = y->parent;
    }
    return y;
}
{% endhighlight %}

Insert(插入)：伪代码如下：
![Image](/uploads/2011/01/BSTree_insert_code.jpg)
Tree_Insert从根节点开始，并沿树下降，指针x跟踪了这条路径，而y始终指向x的父节点。初始化后，3~7行的中的while循环使这两个指针沿树下降，8~13行重置z的有关指针。以关键字13插入一棵二叉树为例，浅阴影节点表示从树根结点开始向下的路径（3~7行), 虚线表示为插入的该数据项而加入链接(8~13行)，如图： 
![Image](/uploads/2011/01/BSTree_insert_pic.jpg)
{% highlight c %}
//插入操作 
void BSTreeInsert(SearchTree &tree, int k) {
    Position y = NULL;
    Position x = tree;
    Position z = new BST;
    z->key = k;
    z->parent = z->left = z->right = NULL;//初始化
      
     //找到要插入的位置 
    while (x != NULL) {
        y = x;
        if (z->key < x->key) {
            x = x->left;
        } else {
            x = x->right;
        }
    }
    //和周边点增加关系 
    z->parent = y;
    if (y == NULL) {
        tree = z;
    } else {
        if (k < y->key) {
            y->left = z;
        } else {
            y->right = z;
        }
    }
}
{% endhighlight %}
Delete(删除):

删除时有三种可能：

1. 如果z没有子女，则修改其父节点p[z]， 使NIL为其子女，如图(a)

2. 如果结点只有一个子女，则可以通过在其子结点与父节点间建立一条链来删除z，如图(b)

3. 如果结点z有两个子女，先删除z的后继y，再用y的内容来替代z的内容，如图(c)
![Image](/uploads/2011/01/BSTree_delete_arc.jpg)

代码：
{% highlight c %}
// 删除操作
Position  BSTreeDelete(SearchTree tree, Position z) {
    Position x, y;
    //下边if-else确定要删除的节点y 
    if (z->left == NULL || z->right == NULL) {
        y = z;//如果z不是第三种情况，则要删除的结点就是z 
    } else {//如果是第三种情况，则先删除z的后继 
        y = BSTreeSuccessor(z);
    }
    //x为y的子女或NULL 
    if (y->left != NULL) {
        x = y->left;
    } else {
        x = y->right;
    }
    //通过修改y周边的关系，将y删除 
     
    //改变x的parent 是y的parent
    if (x != NULL) {
        x->parent = y->parent;
    }
    //上边的if第一种情况处理完毕
    //下边的if-elseif-else第二种情况处理完毕 
    //y的parent的儿子是x，父子关系都修改，y即删除 
    if (y->parent == NULL) {
        tree = x;
    } else if (y == y->parent->left) {
        y->parent->left = x;
    } else {
        y->parent->right = x;
    }
    //处理第三种情况，把y中的内容复制到z中 
    if (y != z) {
        z->key = y->key;
    }
    return y;
}
{% endhighlight %}

完整测试代码：

{% highlight c %}
#include<iostream>
#include<cstdio>
using namespace std;
 
typedef struct BSTNode BST;//BST代表结点 
//BSTNode * == Position; Position代表指针 
typedef struct BSTNode * Position;
//BSTNode * == SearchTree; SearchTree代表二叉树的地址 
typedef struct BSTNode * SearchTree;
 
//二叉树数据结构 
struct BSTNode {
    int key;
    Position parent, left, right;
}; 
 
//先序遍历(递归版) 
void PreOrderTraverse(SearchTree tree) {
    if (tree != NULL) {
        printf("%d ", tree->key);
        PreOrderTraverse(tree->left);
        PreOrderTraverse(tree->right);
    }
}
 
//中序遍历(递归版)
void InOrderTraverse(SearchTree tree) {
    if (tree != NULL) {
        InOrderTraverse(tree->left);
        printf("%d ", tree->key);
        InOrderTraverse(tree->right);
    }
}
 
//后序遍历(递归版) 
void PostOrderTraverse(SearchTree tree) {
    if (tree != NULL) {
        if (tree->left)
        PostOrderTraverse(tree->left);
        PostOrderTraverse(tree->right);
        printf("%d ", tree->key);
    }
}  
         
//Search,查找操作(递归版本),递归版本简洁，易于理解，但效率不如非递归 
Position BSTreeSearch(SearchTree tree, int k) {
    if (tree == NULL || k == tree->key) {
        return tree;
    }
    if (k < tree->key) {
        return BSTreeSearch(tree->left, k);
    } else {
        return BSTreeSearch(tree->right, k);
    }
}
 
// Search操作，非递归版本 
Position BSTreeSearch1(SearchTree tree, int k) {
    while (tree != NULL && k != tree->key) {
        if (k < tree->key) {
            tree = tree->left;
        } else {
            tree = tree->right;
        }
    }
    return tree;
}
      
//返回二叉树最左边值（即最小值）的指针， 
Position BSTreeMinimim(SearchTree tree) {
    while (tree->left != NULL) {
        tree = tree->left;
    }
    return tree;
}
 
//返回二叉树最右边值（即最大值）的指针 
Position BSTreeMaximum(SearchTree tree) {
    while (tree->right != NULL) {
        tree = tree->right;
    }
    return tree;
}
 
 
//中序遍历下找它的后继
Position BSTreeSuccessor(Position x) {
    if (x->right != NULL) {
        return BSTreeMinimim(x->right);
    }
    Position y = x->parent;
    while (y != NULL && x == y-> right) {
        x = y;
        y = y->parent;
    }
    return y;
}
 
//找前驱是一样的道理，把right改为left就可以了
 Position BSTreePredecessor(Position x) {
    if (x->left != NULL) {
        return BSTreeMaximum(x->left);
    }
    Position y = x->parent;
    while (y != NULL && x == y-> left) {
        x = y;
        y = y->parent;
    }
    return y;
}
  
//插入操作 
void BSTreeInsert(SearchTree &tree, int k) {
    Position y = NULL;
    Position x = tree;
    Position z = new BST;
    z->key = k;
    z->parent = z->left = z->right = NULL;//初始化
      
     //找到要插入的位置 
    while (x != NULL) {
        y = x;
        if (z->key < x->key) {
            x = x->left;
        } else {
            x = x->right;
        }
    }
    //和周边点增加关系 
    z->parent = y;
    if (y == NULL) {
        tree = z;
    } else {
        if (k < y->key) {
            y->left = z;
        } else {
            y->right = z;
        }
    }
}
 
// 删除操作
Position  BSTreeDelete(SearchTree tree, Position z) {
    Position x, y;
    //下边if-else确定要删除的节点y 
    if (z->left == NULL || z->right == NULL) {
        y = z;//如果z不是第三种情况，则要删除的结点就是z 
    } else {//如果是第三种情况，则先删除z的后继 
        y = BSTreeSuccessor(z);
    }
    //x为y的子女或NULL 
    if (y->left != NULL) {
        x = y->left;
    } else {
        x = y->right;
    }
    //通过修改y周边的关系，将y删除 
     
    //改变x的parent 是y的parent
    if (x != NULL) {
        x->parent = y->parent;
    }
    //上边的if第一种情况处理完毕
    //下边的if-elseif-else第二种情况处理完毕 
    //y的parent的儿子是x，父子关系都修改，y即删除 
    if (y->parent == NULL) {
        tree = x;
    } else if (y == y->parent->left) {
        y->parent->left = x;
    } else {
        y->parent->right = x;
    }
    //处理第三种情况，把y中的内容复制到z中 
    if (y != z) {
        z->key = y->key;
    }
    return y;
}
 
int main() {
    SearchTree tree = NULL;
     
    //构造文章开头图中的二叉树
    BSTreeInsert(tree,15); BSTreeInsert(tree, 6);
    BSTreeInsert(tree,18); BSTreeInsert(tree, 3);
    BSTreeInsert(tree, 7); BSTreeInsert(tree, 17); 
    BSTreeInsert(tree, 20); BSTreeInsert(tree, 2); 
    BSTreeInsert(tree, 4);  BSTreeInsert(tree, 13); 
    BSTreeInsert(tree, 9);

    puts("先序遍历:"); 
    PreOrderTraverse(tree);
     
    puts("\n\n中序遍历:"); 
    InOrderTraverse(tree);
     
    puts("\n\n后序遍历:"); 
    PostOrderTraverse(tree);
     
    printf("\n\n最小值是：%d\n", BSTreeMinimim(tree)->key);
    printf("\n最大值是；%d\n\n", BSTreeMaximum(tree)->key);  
     
    Position locate13 = BSTreeSearch(tree, 13);
    printf("13的前驱是：");
    printf("%d\n\n", BSTreePredecessor(locate13) -> key);  
    printf("13的后继是：");
    printf("%d\n\n", BSTreeSuccessor(locate13) -> key); 
     
    puts("删除13后的中序遍历: ");
    BSTreeDelete(tree, locate13);
    InOrderTraverse(tree);
     
    return 0;
}
/*
运行结果：(运行环境C-FREE5专业版) 
______________________________ 
先序遍历:
15 6 3 2 4 7 13 9 18 17 20
 
中序遍历:
2 3 4 6 7 9 13 15 17 18 20
 
后序遍历:
2 4 3 9 13 7 6 17 20 18 15
 
最小值是：2
 
最大值是；20
 
13的前驱是：9
 
13的后继是：15
 
删除13后的中序遍历:
2 3 4 6 7 9 15 17 18 20 
*/
{% endhighlight %}