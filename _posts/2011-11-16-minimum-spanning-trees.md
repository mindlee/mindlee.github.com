---
title: 最小生成树(Minimum Spanning Trees)
author: Wei Li
excerpt: 假设要在 n 个城市之间建立通讯联络网，则连通 n 个城市只需要修建 n-1条线路，如何在最节省经费的前提下建立这个通讯网？答案就是：最小生成树。术语描述就是：在 e条带权的边中选取 n-1 条边（不构成回路），使“权值之和”为最小。
meta_description: 假设要在 n 个城市之间建立通讯联络网，则连通 n 个城市只需要修建 n-1条线路，如何在最节省经费的前提下建立这个通讯网？答案就是：最小生成树。术语描述就是：在 e条带权的边中选取 n-1 条边（不构成回路），使“权值之和”为最小。
layout: post
permalink: /2011/11/16/minimum-spanning-trees/
views:
  - 6590
categories:
  - 算法学习
tags:
  - 图论
  - 数据结构
  - 算法
  - 算法导论
---
假设要在 n 个城市之间建立通讯联络网，则连通 n 个城市只需要修建 n-1条线路，如何在最节省经费的前提下建立这个通讯网？答案就是：最小生成树。术语描述是：在 e条带权的边中选取 n-1 条边（不构成回路），使“权值之和”为最小。
![Image][1]
如上图是一个无向连通图，图中显示各条边的权值，带阴影的边为最小生成树的边，树中各边的权值之和为37。最小生成树不是唯一的：如图，用边（a, h）替代边（b，c）得到的是另外一棵最小生成书，其中各边权值之和也是37。如果还是不够清楚，再上一张图：（如下，借用自lcy课件），将最小生成树单独分离画出来，明显多了。
![Image][2]
解决最小生成树问题有两种算法：Kruskal算法和Prim算法。在了解这两种算法之前，先得了解一下MST性质（最小生成树性质）:设G = (V，E)是一个连通网络，U是顶点集V的一个真子集。若(u，v)是G中一条“一个端点在U中(例如：u∈U)，另一个端点不在U中的边(例如：v∈V-U)，且(u，v)具有最小权值，则一定存在G的一棵最小生成树包括此边(u，v)。通俗的讲，就是最小权值的边必定在最小生成树上，前提是，边的一个顶点在生成树上，另一个点不在。

###一、Kruskal算法

考虑问题的出发点: 为使生成树上边的权值之和达到最小，则应使生成树中每一条边的权值尽可能地小。具体做法: 先构造一个只含 n 个顶点的子图 SG（Sub-Graph），然后从权值最小的边开始，若它的添加不使SG 中产生回路，则在 SG 上加上这条边，如此重复，直至加上 n-1 条边为止。看图示应该会更清楚：
![Image][3]
![Image][4]
简单的说，Kruskal算法过程就是，每次寻找最小权值的边，然后加入到最小生成树中，这个过程用到的操作，就是并查集，FIND-SET(u)返回最小包含u的集合中的一个代表元素（最小权值边的顶点），通过测试FIND-SET(u)是否等价于FIND-SET(v)判定顶点u和v是否属于同一棵树。通过过程UNION，实现树与树的合并。简单伪代码：

	MST-KRUSKAL(G, w)
	1  A ← Ø
	2  for each vertex v ∈ V[G]
	3       do MAKE-SET(v)
	4  sort the edges of E into nondecreasing order by weight w
	5  for each edge (u, v) ∈ E, taken in nondecreasing order by weight
	6       do if FIND-SET(u) ≠ FIND-SET(v)
	7             then A ← A ∪ {(u, v)}
	8                  UNION(u, v)
	9  return A

第1~3行将集合A初始化为空集，并建立\|V\|棵树，每棵树都包含了图的一个顶点。在第4行中，根据权值的非递减性顺序，对E中的边进行排序。在第5~8行的for循环中，首先检查每条边（u, v），其端点u和v是否属于同一棵树。如果是，把(u，v)加入到森林中就会形成一个回路，所以放弃边（u, v），否则说明两个顶点分属于不同的树，由第7行把边加入集合A中，第8行对两棵树中的顶点进行合并。

Kruskal的代码实现，可以从一道题目开始[HDU 1233 还是畅通工程](http://acm.hdu.edu.cn/showproblem.php?pid=1233)，这个题目的大意，和本文开始时描述的情形类似。这个题目可以用并查集 + Kruskal + Prim三种方法解决。下面是Kruskal方法解决代码（刚整理的Kruskal模板）

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int NV = 101;
const int NE = 5011;
 
struct Kruskal {
    int n, size;
    int root[NV];
    int mst;
 
    struct Edge {
        int u, v, w;
        Edge () {}
        Edge (int U, int V, int W = 0) : u(U), v(V), w(W) {}
        bool operator < (const Edge &rhs) const {//从小到大
            return w < rhs.w;
        }
    } E[NE];
 
    inline void init(int x) {
        n = x, size = 0;
        for (int i = 0; i < x; i++) {
            root[i] = i;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size++] = Edge(u, v, w);
    }
 
    int Getroot (int n) {
        int r = n;
        while (r != root[r]) r = root[r];
        int x = n, y;
        while (x != r) {
            y = root[x];
            root[x] = r;
            x = y;
        }
        return r;
    }
 
    bool Union(int x, int y) {
        int fx = Getroot(x), fy = Getroot(y);
        if (fx != fy) {
            root[fx] = fy;
            return true;
        }
        return false;
    } 
 
    int kruskal () {
        mst = 0;
        sort(E, E + size);
        int cnt = 0;
        for (int i = 0; i < size; i++) {
            if (Union(E[i].u, E[i].v)) {
                mst += E[i].w;
                if (++cnt == n - 1) break; // 优化
            }
        }
        return mst;
    } // kruskal O(E)
}G;
 
int main() {
    int n;
    while (~scanf("%d", &n), n) {
        G.init(n);
        int k = n * (n - 1) / 2;
        for (int i = 0; i < k; i++) {
            int a, b, c;
            scanf("%d%d%d", &a, &b, &c);
            G.insert(a - 1, b - 1, c);
        }
        printf("%d\n", G.kruskal());
    }
    return 0;
}
{% endhighlight %}

###二、Prim算法

取图中任意一个顶点 v 作为生成树的根，之后往生成树上添加新的顶点 u。在添加的顶点 u 和已经在生成树上的顶点v 之间必定存在一条边，并且该边的权值在所有连通顶点 v 和 u 之间的边中取值最小。之后继续往生成树上添加顶点，直至生成树上含有 n-1 个顶点为止。也就是，每次添加到树中的边，都是树的权尽可能小的边。图示：

![Image][5]

实现过程，即每次找到和它连接所有边中最小权值的边（寻找最小值，可以用最小堆，斐波那契堆等），然后加到最小生成树中即可，伪代码：

	MST-PRIM(G, w, r)
	1  for each u ∈ V [G]
	2       do key[u] ← ∞
	3          π[u] ← NIL
	4  key[r] ← 0
	5   Q ← V [G]
	6   while Q ≠ Ø
	7       do u ← EXTRACT-MIN(Q)
	8          for each v ∈ Adj[u]
	9              do if v ∈ Q and w(u, v) < key[v]
	10                    then π[v] ← u
	11                         key[v] ← w(u, v)

1~5行置每个顶点的域为inf（根顶点r除外，r的key域被置为0，这样它就会成为第一个被处理的顶点），6~11更新权值信息。同样是上边那道题目，Prim算法实现：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
using namespace std;
const int NV = 101;
const int inf = 0x7f7f7f7f;
 
int n, m, a, b, w, gra[NV][NV];
bool mark[NV];
//weight权值，pre记录边的前一个点
int weight[NV], pre[NV];
 
int prim (int src) {
    int  id;
    int ans = 0;
    memset(mark,false,sizeof(mark));
    for (int i = 1; i <= n; i++) {
        weight[i] = gra[src][i];
        pre[i] = src;
    }
    mark[src] = true;
    //n个节点至少需要n - 1条边构成最小生成树
    for (int i = 1; i < n; i++) {
        id = -1;
        for (int j = 1; j <= n; j++) {
            if (mark[j] ) continue;
            ///权值较小且不在生成树中
            if (id == -1 || weight[j] < weight[id]) { 
                id = j;
            }
        }
        if (gra[ pre[id] ][ id ] == inf) return -1;
        ans += gra[ pre[id] ][ id ]; 
        mark[id] = true;
        //更新当前节点到其他节点的权值， 更新权值信息
        for (int j = 1; j <= n; j++) {
            if (mark[j]) continue;
            if (weight[j] > gra[id][j]) {
                weight[j] = gra[id][j];
                pre[j] = id;
            }//if
        }
    }//for
    return ans;
}
 
int main() {
    while(~scanf("%d", &n), n) {
        for (int i = 1; i <= n; i++) {
            gra[i][i] = 0;
            for (int j = i + 1; j <= n; j++) {
                gra[i][j] = inf;
            }
        }//for
        m = n * (n - 1) / 2;
        while (m--) {
            scanf("%d%d%d", &a, &b, &w);
            gra[a][b] = gra[b][a] = w;
        }
        printf("%d\n",prim(1));
    }
    return 0;
}
{% endhighlight %}

###三、Kruskal和Prim对比

####1）过程简单对比

Kruskal算法：所有的顶点放那，每次从所有的边中找一条代价最小的；

Prim:算法：在U，（V – U）之间的边，每次找一条代价最小的

####2）效率对比

效率上，稠密图 Prim > Kruskal；稀疏图 Kruskal > Prim

Prim适合稠密图，因此通常使用邻接矩阵储存，而Kruskal多用邻接表。

####3）空间对比

解决最小生成树题目的时候，要根据给出数据的情况，来决定使用哪种算法，比如：1）当点少边多的时候，如1000个点500000条边，这样BT的数据，用prim做就要开一个1000 * 1000的二维数组，而用kruskal做只须开一个500000的数组，500000跟1000*1000比较相差一半。2）当点多边少的时候，如1000000个点2000条边，像这种数据就是为卡内存而存在的，如果用prim做，你想开一个1000000 * 1000000的二维数组，内存绝对爆掉，而用kruskal只须开一个2000的数组就可以了。

[1]: /uploads/2011/11/最小生成树图示.png
[2]: /uploads/2011/11/最小生成树示例.png
[3]: /uploads/2011/11/kruskal_pic1.png
[4]: /uploads/2011/11/kruskal_pic2.png
[5]: /uploads/2011/11/Prim图示.png