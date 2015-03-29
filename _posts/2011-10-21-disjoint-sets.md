---
title: 并查集(Disjoint Sets)
author: Wei Li
excerpt: 是在FIND-SET操作中，把查找路径上的每个结点都直接指向根结点。路径压缩并不改变结点的秩。关于路径压缩，看图理解，之间为FIND-SET操作前集合，之后为FIND-SET操作后集合。此时，查找路径上的每一个结点都直接指向根。
layout: post
permalink: /2011/10/21/disjoint-sets/
views:
  - 7287
categories:
  - 算法学习
tags:
  - 数据结构
  - 树
  - 笔记
  - 算法
  - 算法导论
---
《算法导论》这一章讲的有点麻烦。首先，这一章标题是《用于不相交集合的数据结构》，内容看了半天才反应过来，这不就并查集吗，搞这么长个翻译。对于并查集的理解，非常好的一个例子：（借用自师兄PPT）
![Image][1]

这幅图中，每一个单独的小组合就是一个独立的集合，集合与集合之间不相交，比如虚竹所在逍遥派是一个集合，（天龙八部背景大概在1090-1094年，离北宋灭亡还有33年），小龙女和杨过所在古墓派是一个集合（神雕侠侣背景大概在1233-1259年，离南宋灭亡还有20年），从时间上和门派上这两个集合之间没有任何联系，只有“乱世出英雄”是相同的。把这些集合放一起就是并查集了。

###一、并查集基础

在一些应用中，要将n个不同的元素分成一组不相交的集合。并查集的两个操作：找出给定元素所属的集合和合并两个集合，即查询和合并功能，是非常实用的两个功能。并查集经常在使用中以森林来表示。并查集还有另外一种翻译是：Union Find Sets，从英文基本可以得出并查集能进行的操作：

1. MAKE-SET(x): 建立一个新的集合，其唯一成员为x。

2. UNION(x,y): 将包含x和y的动态集合合并为一个新的集合。

3. FIND-SET(x): 返回一个指针，指向包含x的集合的代表。

前两个操作伪代码：

	MAKE-SET(x)
	1  p[x] ← x
	2  rank[x] ← 0
 	

	UNION(x, y)
	1  LINK(FIND-SET(x), FIND-SET(y))
 

	LINK(x, y)
	1  if rank[x] > rank[y]
	2     then p[y] ← x
	3     else p[x] ← y
	4          if rank[x] = rank[y]
	5             then rank[y] ← rank[y] + 1

其中，LINK是UNION调用的一个子过程，具体代码实现可以参考下边第例子。

###二、并查集应用

关于并查集的应用，一个很好的简单例子可以从（[HDU 1232 畅通工程](http://acm.hdu.edu.cn/showproblem.php?pid=1232)）开始。大致题意是：给出单独点之间的连接关系，问有多少个不相交的集合？典型的并查集题目。

下边是AC代码：
{% highlight c %}
#include<iostream>
#include<cstdio>
using namespace std;
 
int parent[1002];
 
//查找集合 
int Find(int x){
    while (parent[x] != x) {
        x = parent[x];
    }
    return x;
}
 
//合并操作 
void Union(int x, int y) {
    x = Find(x);
    y = Find(y);
    if(x != y) {
        parent[x] = y; 
    }
}
 
int main() {
    int n, m;
    while(~scanf("%d", &n), n) {
        //建立集合 
        for(int i = 1; i <= n; i++) {
            parent[i] = i;
        }
         
        scanf("%d", &m);
        while (m--) {
            int x, y;
            scanf("%d %d", &x, &y);
            Union(x, y);
        }
         
        //计算不相交集合数 
        int cnt = 0;
        for(int i = 1; i <= n; i++) {
            if(parent[i] == i) {
                cnt ++;
            }
        }
        printf("%d\n", cnt - 1);
    }
}
{% endhighlight %}

在这之后，有两种优化方法，第一种是按秩合并，第二种是路径压缩。

####1）按秩合并。

按秩合并的基本思想是使包含较少结点的树德根指向包含较多结点的树的根，而这个树的大小可以抽象为树的高度，即高度小的树合并到高度大的树，这样资源利用更加合理。

为了实现一个按秩合并的不想交集合森林，要记录下秩的变化。对于每个结点x，有一个整数rank[x]，它是x的高度（从x到其某一个后代叶结点的最长路径上边的数目）的一个上界。（即树高）。当由MAKE-SET创建了一个单元集时，对应的树中结点的初始秩为0，每个FIND-SET操作不改变任何秩。当对两棵树应用UNION时，有两种情况，具体取决于根是否有相等的秩。当两个秩不相等时，我们使具有高秩的根成为具有较低秩的根的父结点，但秩本身保持不变。当两个秩相同时，任选一个根作为父结点，并增加其秩的值路径压缩。

简单代码解释：
{% highlight c %}
void Union(int a, int b) {
    if(village[a].weight == village[b].weight) {//树高一样
        village[b].parent = a;
        village[a].weight += 1;
    } else if (village[a].weight > village[b].weight) {//矮树并入高树
        village[b].parent = a;//并入a
    } else {
        village[a].parent = b;//并入b
    }
}
{% endhighlight %}
####2）路径压缩

是在FIND-SET操作中，把查找路径上的每个结点都直接指向根结点。路径压缩并不改变结点的秩。关于路径压缩，看图理解，之间为FIND-SET操作前集合，之后为FIND-SET操作后集合。此时，查找路径上的每一个结点都直接指向根。
![Image][2]
路径压缩代码实现方式有两种：递归式和非递归式。

1）递归方式

伪代码：

	FIND-SET(x)
	1  if x ≠ p[x]
	2     then p[x] ← FIND-SET(p[x])
	3  return p[x]

这个过程FIND-SET是一种两趟方法（two-pass method）：一趟是沿查找路径上升，直至找到根；第二趟是沿查找路径下降，以便更新每个结点，使之直接指向根。对FIND-SET（x）的每一次调用，都会在第3行返回p[x]。如果x为根，则不执行第2行，返回p[x] = x。这种情况下递归结束。否则，执行第2行，且参数为p[x]的递归调用返回一个指向根的指针。第2行更新结点x，使之直接指向根，并在第3行返回这个指针。

{% highlight c %}
int Find(int x) {
    if(root[x] == -1)  {
        return x;
    }
    return root[x] = Find(root[x]);
}
{% endhighlight %}

2）非递归方式

{% highlight c %}
//非递归版路径压缩
int Find (int n) {//r->root
    int r = n;
    while (r != root[r]) {//寻找根结点
        r = root[r];
    }
    int x = n, y;
    while (x != r) {//压缩路径，全部赋值为根结点的值  
        y = root[x];
        root[x] = r;
        x = y;
    }
    return r;
}
{% endhighlight %}

下边是刚刚的题目利用两种改进方法优化后的代码，此题数据规模太小，OJ提交后，结果上没体现出效率改进，数据量大时，应该会非常明显。

{% highlight c %}
#include<iostream>
using namespace std;
 
const int N = 1001;
 
//结点ADT 
struct Node {
    int parent;
    int weight;
}village[N];
 
//路径压缩，查找
int Find(int n) {//r->root
    int r = n;
    while (r != village[r].parent) {//寻找根节点
        r = village[r].parent;
    }
    int x = n, y;
    while (x != r) {//压缩路径，全部赋值为根节点的值
        y = village[x].parent;
        village[x].parent = r;
        x = y;
    }
    return r;
}
  
//按秩合并
void Union(int a, int b) {
    int a1 = Find(a);
    int b1 = Find(b);
    if (a1 == b1) {//如果是同一个集合就不用合并
        return ;
    }
     //按秩合并
    if (village[a1].weight == village[b1].weight) {
        village[b1].parent = a1;
        village[a1].weight += 1;
    } else if (village[a1].weight < village[b1].weight) {
        village[a1].parent = b1;
    } else {
        village[b1].parent = a1;
    }
}
 
int main() {
    int n, m;
    while (~scanf("%d", &n), n) {
        //构造集合
        for (int i = 1; i <= n; i++) {
            village[i].parent = i;
            village[i].weight = 1;
        }
        scanf("%d", &m);
        while (m--) {
            int x, y;
            scanf("%d %d", &x, &y);
            Union(x, y);
        }
        //计算集合数
        int cnt = 0;
        for (int i = 1; i <= n; i++) {
            if (village[i].parent == i) {
                cnt ++;
            }
        }
        printf("%d\n", cnt - 1);
    }
    return 0;
}
{% endhighlight %}
###三、最近公共祖先LCA Tarjan

在一棵有根数T中，两个结点u和v的最近公共祖先（Least Common Ancestors）是指这样一个结点w， 它是u和v的祖先，并且在树T中具有最大深度。换种说法就是，对于有根树T的两个结点u、v，最近公共祖先 LCA(T, u, v)：询问一个距离根最远的结点x，使得x同时为结点u、v的祖先。只有两种情况，上图：
![Image][3]
引用其他地方对Tarjan的描述：

>“利用并查集优越的时空复杂度，我们可以实现LCA问题的O(n + Q)算法，这里Q表示询问的次数。Tarjan算法基于深度优先搜索的框架，对于新搜索到 的一个结点，首先创建由这个结点构成的集合，再对当前结点的每一个子树进行搜索，每搜索完一棵子树，则可确定子树内的LCA询问都已解决。其他的LCA询 问的结果必然在这个子树之外，这时把子树所形成的集合与当前结点的集合合并，并将当前结点设为这个集合的祖先。之后继续搜索下一棵子树，直到当前结点的所 有子树搜索完。这时把当前结点也设为已被检查过的，同时可以处理有关当前结点的LCA询问，如果有一个从当前结点到结点v的询问，且v已被检查过，则由于 进行的是深度优先搜索，当前结点与v的最近公共祖先一定还没有被检查，而这个最近公共祖先的包涵v的子树一定已经搜索过了，那么这个最近公共祖先一定是v 所在集合的祖先。”

>为了解决最近公共祖先问题，通过对LCA（root[T]）初始调用，来执行对T的树遍历。在遍历之前，假定每个结点都着色为WHITE。（同flag，标记false，true同理）.下边是离线Tarjan算法的伪代码，说离线是因为，这个算法必须将所有的询问先记录下来，再一次性的求出每个点对的最近公共祖先。

	LCA(u)
	1  MAKE-SET(u)
	2  ancestor[FIND-SET(u)] ← u
	3  for each child v of u in T
	4       do LCA(v)
	5          UNION(u, v)
	6          ancestor[FIND-SET(u)] ← u
	7  color[u] ← BLACK
	8  for each node v such that {u, v} ∈ P
	9       do if color[v] = BLACK
	10            then print "The least common ancestor of"
                          u "and" v "is" ancestor[FIND-SET(v)]

对于这个算法的应用，很好的一个例子是（[HDU 2586 How far away ？](http://acm.hdu.edu.cn/showproblem.php?pid=2586)）。题意是，求在一颗无向树中，任意两点间的距离。利用的简单的公式：distance(a,b) = dis[a] + dis[b]  – 2 * dis[LCA(a,b)]即可求出。

下边是HDU2586AC代码：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std; 
 
const int N = 40001; 
 
struct Edge {
    int v, w, next; 
}edge[2 * N]; 
 
int n, m, size, head[N]; 
int x[N], y[N], z[N], root[N], dis[N];
bool mark[N]; 
 
//插入边
void Insert(int u, int v, int w) {
    edge[size].v = v; edge[size].w = w; 
    edge[size].next = head[u]; head[u] = size++ ; 
    edge[size].v = u; edge[size].w = w; 
    edge[size].next = head[v]; head[v] = size++ ; 
}
 
//查找操作
int Find(int x){
    if(root[x] != x) {
        return root[x] = Find(root[x]); 
    }
    return root[x]; 
}
 
void LCA_Tarjan(int k) {
    mark[k] = true; 
    root[k] = k; 
    //m次询问, z[i]保存的是点 x[i] 和 y[i] 最近公共祖先
    for(int i = 1; i <= m; i++ ) {
        if(x[i] == k && mark[y[i]]) z[i] = Find(y[i]); 
        if(y[i] == k && mark[x[i]]) z[i] = Find(x[i]); 
    }
    for(int i = head[k]; i != -1; i = edge[i].next) {
        if(!mark[edge[i].v]) {
            dis[edge[i].v] = dis[k] + edge[i].w; 
            LCA_Tarjan(edge[i].v); 
            root[edge[i].v] = k; 
        }
    }
}
 
int main() {
    int cas, u, v, w, i; 
    scanf("%d", &cas); 
    while(cas--) {
        scanf("%d %d", &n, &m); 
        size = 0; 
        memset(head, -1, sizeof(head)); 
        for(i = 1; i < n; i++ ) {
            scanf("%d %d %d", &u, &v, &w); 
            Insert(u, v, w); 
        }
 
        for(i = 1; i <= n; i++ ) {
            x[i] = y[i] = z[i] = 0; 
        }
 
        for(i = 1; i <= m; i++ ) {
            scanf("%d %d", &u, &v); 
            x[i] = u; y[i] = v; 
        }
 
        memset(mark, false, sizeof(mark)); 
        dis[1] = 0; 
        LCA_Tarjan(1); 
 
        for(i = 1; i <= m; i++ ) {
            printf("%d\n", dis[x[i]] + dis[y[i]] - 2 * dis[z[i]]); 
        }
    }
    return 0; 
}
{% endhighlight %}

<hr/>
###一些评论

kds说道：2012/09/08 16:36 
: 酷行天下, 在第二部分，你改进后的代码，weight更新会出错的。更新之后，root的weight一定会大于等于实际weight。

酷~行天下说道：2012/09/09 00:00 
: weight ? 这个值的含义是根据需要自己定义的。

{% highlight c %}
void Union(int a, int b) {
    if(village[a].weight == village[b].weight) {//树高一样
        village[b].parent = a;
        village[a].weight += 1;
    } else if (village[a].weight > village[b].weight) {//矮树并入高树
        village[b].parent = a;//并入a
    } else {
        village[a].parent = b;//并入b
    }
}
{% endhighlight %}
这里的weight代表树的高度

{% highlight c %}
//按秩合并
void Union(int a, int b) {
    a = Find(a);
    b = Find(b);
    if (a == b) {//如果是同一个集合就不用合并
        return ;
    }
    //按秩合并
    if (village[a].weight >= village[b].weight) {
        village[b].parent = a;
        village[a].weight += village[b].weight;
    } else {
        village[a].parent = b;
        village[b].weight += village[a].weight;
    }
}//Union
这里的weight是代表树中结点的个数。
{% endhighlight %}

你说的"root的weight一定会大于等于实际weight。" 那道题中，root的weight代表所有结点的个数，每个结点只加了一遍，怎么会大于实际weight呢？

kds说道：2012/09/09 01:19 
: 我看错了。晕。不好意思。。。

酷~行天下说道：2012/09/09 07:23 
: ^_^

 [1]: /uploads/2011/10/disjoint-sets_pic.png
 [2]: /uploads/2012/10/路径压缩.png
 [3]: /uploads/2011/10/LCA.png