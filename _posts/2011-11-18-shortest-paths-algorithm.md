---
title: 最短路算法(Shortest Paths Algorithm)
author: Wei Li
excerpt: SPFA算法是西南交通大学段凡丁于1994年发表的。它是Bellman-Ford的队列优化，时效性相对好，时间复杂度O（kE），也是单源最短路算法，同时可以处理负权边。从名字即可看出，此算法速度非同一般。
layout: post
permalink: /2011/11/18/shortest-paths-algorithm/
views:
  - 15794
categories:
  - 算法学习
tags:
  - 图论
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
假如你有一张地图，地图上给出了每一对相邻城市的距离，从一个地点到另外一个地点，如何找到一条最短的路？ 最短路算法要解决的就是这类问题。定义：给定一个有(无)向图，每一条边有一个权值 w，给定一个起始点 S 和终止点 T ，求从 S 出发走到 T 的权值最小路径，即为最短路径。最短路算法依赖一种性质：一条两顶点间的最短路径包含路径上其他最短路径。简单的说就是：最短路径的子路径是最短路径。这个用反证法很好证明。

###一、松弛技术（Relaxation）

了解最短路算法前，必须先了解松弛技术, 为什么叫松弛，有特定原因，有兴趣可以去查查相关资料，如果简化理解松弛技术，它本质上就是一个贪心操作。松弛操作：对每个顶点v∈V，都设置一个属性d[v]，用来描述从源点 s 到 v 的最短路径上权值的上界，成为最短路径估计(Shortest-path Estimate)，同时π[v]代表前趋。初始化伪代码：

	INITIALIZE-SINGLE-SOURCE(G, s)
	1  for each vertex v ∈ V[G]
	2       do d[v] ← ∞
	3          π[v] ← NIL
	4  d[s] ← 0
    
![Image][1]

初始化之后，对所有 v∈V，π[v] = NIL，对v∈V – {s}，有 d[s] = 0 以及 d[v] = ∞。松弛一条边（u, v），如果这条边可以对最短路径改进，则更新 d[v] 和 π[v] 。一次松弛操作可以减小最短路径估计的值 d[v] ，并更新 v 的前趋域 π[v]。下面的伪代码对边（u，v）进行了一步松弛操作：

	RELAX(u, v, w)
	1  if d[v] > d[u] + w(u, v)
	2     then d[v] ← d[u] + w(u, v)
	3          π[v] ← u

上边的图示中，左边例子，最短路径估计值减小，右边例子，最短路径估计值不变。当发现 v 到 u 有更近的路径时，更新 d[v] 和 π[v] 。

###二、Dijkstra算法

解决最短路问题，最经典的算法是 Dijkstra算法，它是一种单源最短路算法，其核心思想是贪心算法(Greedy Algorithm)，Dijkstra算法由荷兰计算机科学家Dijkstra发现，这个算法至今差不多已有50年历史，但是因为它的稳定性和通俗性，到现在依然强健。另外，Dijkstra算法要求所有边的权值非负。

Dijkstra算法思想为：设 G = (V, E) 是一个带权有向图，把图中顶点集合 V 分成两组，第一组为已求出最短路径的顶点集合（用 S 表示，初始时 S 中只有一个源点，以后每求得一条最短路径 , 就将其加入到集合 S 中，直到全部顶点都加入到 S 中，算法就结束了），第二组为其余未确定最短路径的顶点集合（用 U 表示），按最短路径长度的递增次序依次把第二组的顶点加入 S 中。在加入的过程中，总保持从源点 v 到 S 中各顶点的最短路径长度不大于从源点 v 到 U 中任何顶点的最短路径长度。此外，每个顶点对应一个距离，S 中的顶点的距离就是从 v 到此顶点的最短路径长度，U 中的顶点的距离，是从 v 到此顶点只包括 S 中的顶点为中间顶点的当前最短路径长度。伪代码：

	DIJKSTRA(G, w, s)
	1  INITIALIZE-SINGLE-SOURCE(G, s)
	2  S ← Ø
	3  Q ← V[G]
	4  while Q ≠ Ø
	5      do u ← EXTRACT-MIN(Q)
	6         S ← S ∪{u}
	7         for each vertex v ∈ Adj[u]
	8             do RELAX(u, v, w)

第 1 行将 d 和 π 初始化，第 2 行初始化集合 S 为空集，4 ~ 8 行每次迭代，都从 U 中选取一个点加入到 S 中，然后所有的边进行松弛操作，即每次迭代，整个图的 d 和 π 都更新一遍。过程本身很简单，下边是图示：

![Image][2]

源点 s 是最左端顶点。最短路径估计被标记在顶点内，阴影覆盖的边指出了前趋的值。黑色顶点在集合 S中，而白色顶点在最小优先队列 Q = V – S 中。a) 第 4 ~ 8 行 while 循环第一次迭代前的情形。阴影覆盖的顶点具有最小的 d 值，而且在第 5 行被选为顶点 u 。b) ~ f) while 循环在第一次连续迭代后的情形。每个图中阴影覆盖的顶点被选作下一次迭代第 5 行的顶点 u。f) 图中的 d 和 π 值是最终结果。

Dijkstra算法时间主要消耗在寻找最小权值的边，和松弛所有剩余边，所以 EXTRACT-MIN(Q) 这一步，更好的方法是使用优先队列，优先队列可以用二叉堆，斐波那契堆等来实现，下面的代码，我用库自带的优先队列，经这样改造后，效率还是很可观的。

理解最短路算法，最基础，最简单，最经典的要数这个题目：[HDU 2544 最短路](http://acm.hdu.edu.cn/showproblem.php?pid=2544)，纯粹的算法练习题，用Dijkstra，我写了三个代码来实现。

1）邻接矩阵 + Dijkstra，最简单的方式，当然也是最好理解的方式：

{% highlight c %}
/*邻接矩阵 +  Dijkstra求最短路*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<climits>
using namespace std;
 
const int NV = 102;
const int inf = INT_MAX >> 1;
 
int map[NV][NV];
bool mark[NV];
int dis[NV];
int n, m;
 
void Dijkstra(int src) {
    for (int i = 0; i < n; i++) {
        dis[i] = map[src][i];
        mark[i] = false;
    }
    dis[src] = 0;
    mark[src] = true;
    for (int i = 1; i < n; i++) {
        int minn = inf;
        int k = src;
        for (int j = 0; j < n; j++) {
            if (!mark[j] && dis[j] < minn) {
                k = j;
                minn = dis[j];
            }
        }
        mark[k] = true;
        for (int j = 0; j < n; j++) {
            int tmp = map[k][j] + dis[k];
            if (!mark[j] && tmp < dis[j]) {
                dis[j] = tmp;
            }
        }
    }
}
 
int main() {
    while (~scanf("%d%d", &n, &m), n || m) {
        for (int i = 0; i < n; i++) {
            map[i][i] = inf;
            for (int j = i + 1; j < n; j++) {
                map[i][j] = inf;
                map[j][i] = inf;
            }
        }
 
        while (m--) {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            if (map[u - 1][v - 1] > w) {
                map[u - 1][v - 1] = w;
                map[v - 1][u - 1] = w;
            }
        }
        Dijkstra(0);
        printf("%d\n", dis[n - 1]);
    }
    return 0;
}
{% endhighlight %}
2）邻接表 + Dijkstra，更通用，最常见的方式：

{% highlight c %}*邻接表 +  Dijkstra求最短路*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
#include<queue>
#include<climits>
using namespace std;
 
const int NV = 102;
const int NE = 20002;
int n, m;
 
struct Dijkstra {
    int n, size;
    int dis[NV], head[NV];
    int mark[NV];
 
    struct node {
        int v, dis;
        node () {}
        node (int V, int DIS) : v(V), dis(DIS) {}
        friend bool operator < (const node a, const node b) {
            return a.dis > b.dis;
        }
    };
 
    struct edge {
        int v, w, next;
        edge () {}
        edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) {}
    }E[NE];
 
    inline void init(int x) {
        n = x, size = 0;
        memset(head, -1, sizeof(int) * (x + 1));
    }
 
    inline void insert(int u, int v, int w) {
        E[size] = edge(v, head[u], w);
        head[u] = size++;
    }
 
    void print() {
        for (int i = 0; i < n; i++) {
            printf("%d: ", i);
            for (int j = head[i]; j != -1; j = E[j].next) {
                printf(" %d", E[j].v);
            }puts("");
        }
    }
 
    int dijkstra(int src, int des) {
        node first, next;
        priority_queue <node> Q;
        for (int i = 0; i <= n; i++) {
            dis[i] = INT_MAX;
            mark[i] = false;
        }
 
        dis[src] = 0;
        Q.push(node(src, 0));
 
        while (!Q.empty()) {
            first = Q.top();
            Q.pop();
            mark[first.v] = true;
 
            for (int i = head[first.v]; i != -1; i = E[i].next) {
                if (mark[E[i].v]) continue;
                next = node(E[i].v, first.dis + E[i].w);
                if (next.dis < dis[next.v]) {
                    dis[next.v] = next.dis;
                    Q.push(next);
                }
            }
        }//while
        return dis[des];
    }//Dij
}G;
 
int main() {
    while (~scanf("%d%d", &n, &m), n || m) {
        G.init(n);
        while (m--) {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            G.insert(u - 1, v - 1, w);
            G.insert(v - 1, u - 1, w);
        }
        //G.print();
        printf("%d\n", G.dijkstra(0, n - 1));
    }
    return 0;
}
{% endhighlight %}

3）邻接表 + 优先队列优化 + Dijkstra，效率更高，更实用的方式：

{% highlight c %}
/*邻接表 +  优先队列 + Dijkstra求最短路*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
#include<climits>
#include<queue>
using namespace std;
 
const int NV = 102;
const int NE = 20002;
int n, m;
 
struct Dijkstra {
    int n, size;
    int dis[NV], head[NV];
    int mark[NV];
 
    struct node {
        int v, dis;
        node () {}
        node (int V, int DIS) : v(V), dis(DIS) {}
        friend bool operator < (const node a, const node b) {
            return a.dis > b.dis;
        }
    };
 
    struct edge {
        int v, w, next;
        edge () {}
        edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) {}
    }E[NE];
 
    inline void init(int vx) {
        n = vx, size = 0;
        memset(head, -1, sizeof(int) * (vx + 1));
    }
 
    inline void insert(int u, int v, int w) {
        E[size] = edge(v, head[u], w);
        head[u] = size++;
    }
 
    void print() {
        for (int i = 0; i < n; i++) {
            printf("%d: ", i);
            for (int j = head[i]; j != -1; j = E[j].next) {
                printf(" %d", E[j].v);
            }puts("");
        }
    }
 
    int dijkstra(int src, int des) {
        node first, next;
        priority_queue <node> Q;
        for (int i = 0; i <= n; i++) {
            dis[i] = INT_MAX;
            mark[i] = false;
        }
 
        dis[src] = 0;
        Q.push(node(src, 0));
 
        while (!Q.empty()) {
            first = Q.top();
            Q.pop();
            mark[first.v] = true;
 
            for (int i = head[first.v]; i != -1; i = E[i].next) {
                if (mark[E[i].v]) continue;
                next = node(E[i].v, first.dis + E[i].w);
                if (next.dis < dis[next.v]) {
                    dis[next.v] = next.dis;
                    Q.push(next);
                }
            }
        }//while
        return dis[des];
    }//Dij
}G;
 
int main() {
    while (~scanf("%d%d", &n, &m), n || m) {
        G.init(n);
        while (m--) {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            G.insert(u - 1, v - 1, w);
            G.insert(v - 1, u - 1, w);
        }
        //G.print();
        printf("%d\n", G.dijkstra(0, n - 1));
    }
    return 0;
}
{% endhighlight %}

如果对Dijkstra算法核心思想不是很理解，可能会问：**Dijkstra算法为什么不能处理负权边？**

Dijkstra由于是贪心的，每次都找一个距源点最近的点（dmin），然后将该距离定为这个点到源点的最短路径（d[i] ← dmin）；但如果存在负权边，那就有可能先通过并不是距源点最近的一个次优点（dmin’），再通过这个负权边 L (L < 0)，使得路径之和更小（dmin’ + L < dmin），则 dmin’ + L 成为最短路径，并不是dmin，这样Dijkstra就被囧掉了。比如n = 3，邻接矩阵：

0， 3， 4

3， 0，-2

4，-2， 0

用Dijkstra求得 d[1，2] = 3，事实上 d[1，2] = 2，就是通过了 1-3-2 使得路径减小。Dijkstra的贪心是建立在边都是正边的基础上，这样，每次往前推进，路径长度都是变大的，如果出现负边，那么先前找到的最短路就不是真正的最短路，比如上边的例子，这个算法也就算废了。

另外，Dijkstra算法时间复杂度为O($V^2$ + E)。源点可达的话，O(V * lgV + E * lgV) => O(E * lgV)。当是稀疏图的情况时，此时 E = V2/ lgV，所以算法的时间复杂度可为 O（V2） 。若是斐波那契堆作优先队列的话，算法时间复杂度为O（V * lgV + E）。

###三、Bellman-Ford算法

Bellman-Ford算法能在一般情况下（存在负权边的情况）下，解决单源最短路径问题。对于给定的带权有向图 G = (V, E)，其源点为 s，加权函数为 w：E → R，，对该图运行 Bellman-Ford 算法后可以返回一个布尔值，表明图中是否存在着一个从源点可达的权为负的回路。若存在这样的回路，问题无解；否则，算法产生最短路径及其权值。

Bellman-Ford算法运用松弛技术，对每个顶点 v，逐步减小从源 s 到 v 的最短路径的权的估计值 d[v] 直至其可达到实际最短路径的权 δ(s, v) 。算法返回布尔值True，当且仅当图中不包含从源点可达的负权回路。伪代码：

	BELLMAN-FORD(G, w, s)
	1  INITIALIZE-SINGLE-SOURCE(G, s)
	2  for i ← 1 to |V[G]| – 1
	3       do for each edge (u, v) ∈ E[G]
	4              do RELAX(u, v, w)
	5  for each edge (u, v) ∈ E[G]
	6       do if d[v] > d[u] + w(u, v)
	7             then return FALSE
	8  return TRUE

 1 行初始化每个顶点的 d 和 π 值后，算法对图中的边进行了 \|V\| – 1 遍操作。每一遍都是第 2 ~ 4 行for循环的一次迭代，有点类似于预处理。下边是的图示是算法对边进行四遍操作，每一遍过后的状态。在 \|V – 1\| 遍操作过后，第 5 ~ 8 行对负权回路进行检查，并返回适当的布尔值。图示：
![Image][3]

源点是顶点 s 。d 值被标记在顶点内，阴影覆盖的边指示了前趋值：如果边（u, v）被阴影覆盖，则 π[v] = u。在这个特定例子中，每一趟按照如下的顺序对边进行松弛：(t，x)，(t，y)，(t，z)，(x，t)，(y，x)，(y，z)，(z，x)，(z，s)，(s，t)，(s，y)。a) 示出了对边进行第一趟操作前的情况。b) ~ e) 示出了每一趟连续对边操作后的情况。e) 中 d 和 π 值是最终结果。这个例子中，返回值是True。

还是上边那道题目，用Bellon-Ford算法实现：（必然有最短路，所以不必判断布尔值）

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int NV = 101;
 
struct Edge {
    int u, v, w;
}gra[NV * NV];
 
int dis[NV];
 
void BellmanFord(int n, int m) {
    memset(dis, 0x7f, sizeof(dis));
    dis[1] = 0;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (dis[gra[j].u] + gra[j].w < dis[gra[j].v]) {
                dis[gra[j].v] = dis[gra[j].u] + gra[j].w;
            }
            if (dis[gra[j].v] + gra[j].w < dis[gra[j].u]) {
                dis[gra[j].u] = dis[gra[j].v] + gra[j].w;
            }
        }
    }
}
 
int main() {
    int n, m;
    while (~scanf("%d %d", &n, &m), n | m) {
        for (int i = 0; i < m; i++) {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            gra[i].u = u, gra[i].v = v, gra[i].w = w;
        }
        BellmanFord(n, m);
        printf("%d\n", dis[n]);
    }
    return 0;
}
{% endhighlight %}

Bellman-Ford虽然很简单，但是复杂度太高，达到了O(VE)，从上边图示中可以看出：(a) t，x，y，z 边的松弛是无用操作；(b) s，x，z 边的松弛是无用操作；(c) s，t，y边的松弛是无用操作；(d) s，x，y，z边的松弛是无用操作。也就是说，只有更新过的点所做的松弛才是有效操作，所以出现了更高效的算法，即SPFA：

###三、SPFA(Shortest Path Faster Algorithm)

SPFA算法是西南交通大学段凡丁于1994年发表的。它是Bellman-Ford的队列优化，时效性相对好，时间复杂度O（kE），也是单源最短路算法，同时可以处理负权边。从名字即可看出，此算法速度非同一般。

与Bellman-ford算法类似，SPFA算法采用一系列的松弛操作以得到从某一个节点出发到达图中其它所有节点的最短路径。所不同的是，SPFA算法通过维护一个队列，使得一个节点的当前最短路径被更新之后没有必要立刻去更新其他的节点，从而大大减少了重复的操作次数。伪代码：

	SPFA(G,w,s)
	1. INITIALIZE-SINGLE-SOURCE(G,s)
	2. INITIALIZE-QUEUE(Q)
	3. ENQUEUE(Q,s)
	4. While Not EMPTY(Q)
	5.      Do u<-DLQUEUE(Q)
	6. For 每条边(u,v) in E[G]
	7.      Do tmp<-d[v]
	8. Relax(u,v,w)
	9. If (d[v] < tmp) and (v不在Q中)
	10.     ENQUEUE(Q,v)
     
Bellon-Ford算法，每次都松弛所有的边，所以造成效率低下，而SPFA的高效之处在于，它每次只松弛更新过的点连接的边，简单的过程叙述就是：

	1. 初始 Dis[s] = 0，其他赋值为Inf；
	2. 将起点s放入空队列 Q；
	3. step 1. 从 Q 中选取元素u，并删除该元素；
	4. step 2. 对所有和 u 相连的点 v 进行松弛，如果 v 被更新且 v 不在队列中，把 v 加进队列；
	5. 一直循环 step 1 和 step 2，直到队列为空。结束；

HH师兄，当年讲SPFA算法时，提到过三个问题，不妨思考一下，可以帮助更好的理解SPFA算法：

	1. 想想怎么判断负环的情况？
	2. 为什么要判断v是否在队列？
	3. 怎么样有效的判断v在不在队列之中？

答案如下：

	1. 如果一个节点更新了 n 次，那么存在负环；
	2. 如果有多个 v 在队列,第一个 v 已经把松弛做完了，剩下的 v 属于无效操作；
	3. 用一个数组Inqueue[]，在元素进队的时候表示成 True，出队的时候标记成 False，判断的时候只要看看Inqueue[v] 是否为 True 就行了；

另外，还需要了解的是，SPFA 的算法时间效率是不稳定的，即它对于不同的图所需要的时间有很大的差别。在最好情形下，每一个节点都只入队一次，则算法实际上变为广度优先遍历，其时间复杂度仅为O(E)。另一方面，存在这样的例子，使得每一个节点都被入队(V – 1)次，此时算法退化为 Bellman-ford算法，其时间复杂度为O(VE)。

SPFA在负边权图上可以完全取代 Bellman-ford 算法，另外在稀疏图中也表现良好。但是在非负边权图中，为了避免最坏情况的出现，通常使用效率更加稳定的 Dijkstra 算法，以及它的使用堆优化的版本。通常的SPFA算法在一类网格图中的表现不尽如人意。

还是上边那道题，用SPFA算法实现：

1）邻接矩阵实现，便于理解算法过程：

{% highlight c %}
#include<iostream>
#include<cstring>
#include<cmath>
#include<algorithm>
#include<queue>
#include<climits>
using namespace std;
 
const int NV = 101;
const int inf = INT_MAX >> 1;
 
int m, n;
int map[NV][NV];
int dis[NV];
bool mark[NV];
 
int Spfa (int src, int des) {
    for (int i = 0; i <= n; i++) {
        mark[i] = false;
        dis[i] = inf;
    }       
    queue<int> Q;
    mark[src] = true;
    dis[src] = 0;
 
    Q.push(src);
    while (!Q.empty()) {
        int first = Q.front();
        Q.pop();
        mark[first] = false;
        for (int i = 1; i <= n; i++) {
            if (dis[first] + map[first][i] < dis[i]) {
                if (!mark[i]) {
                    Q.push(i);
                }
                dis[i] = dis[first] + map[first][i];
                mark[i] = true;
            }
        }//for
    }//while
    return  dis[des];
}
 
int main() {
    while (~scanf("%d%d", &n, &m), n | m) {
        int a, b, c;
        for (int i = 1; i <= n; i++) {
            map[i][i] = inf;
            for (int j = i + 1; j <= n; j++) {
                map[i][j] = map[j][i] = inf;
            }
        }
        while (m--) {
            scanf("%d%d%d", &a, &b, &c);
            map[a][b] = map[b][a] = c;
        }
        printf("%d\n", Spfa(1, n));
    }
    return 0;
}   
{% endhighlight %}

2）更通用的邻接表形式：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
#include<climits>
#include<queue>
#include<stack>
using namespace std;
 
const int NV = 101;
const int NE = 20001;
const int inf = INT_MAX >> 1;
 
struct SPFA {
    int n, size;
    int dis[NV], head[NV];
    bool in[NV];
 
    struct edge {
        int v, w, next;
        edge () {}
        edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) {}
    }E[NE];
 
    void init(int nn) {
        n = nn, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
            in[i] = 0;
            dis[i] = inf;
        }
    }
 
    inline void insert(int u, int v, int w) {
        E[size] = edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline bool relax(int u, int v, int w) {
        if (dis[v] == inf || dis[u] + w < dis[v]) {
            dis[v] = dis[u] + w;
            return true;
        }
        return false;
    }
 
    int spfa(int src, int des) {
        queue<int> que;
        dis[src] = 0;
        que.push(src);
        in[src] = true;
        while (!que.empty()) {
            int u = que.front();
            que.pop();
            in[u] = false;
            for (int i = head[u]; i != -1; i = E[i].next) {
                int v = E[i].v;
                if (relax(u, v, E[i].w) && !in[v] ) {
                    in[v] = true;
                    que.push(v);
                }//if
            }//for
        }//while
        return dis[des];
    }//SFPA
}G;
 
int main() {
    int n, m;
    while (~scanf("%d%d", &n, &m), n | m) {
        G.init(n);
        int a, b, c;
        for (int i = 0; i < m; i++) {
            scanf("%d%d%d", &a, &b, &c);
            G.insert(a, b, c);
            G.insert(b, a, c);
        }
        printf("%d\n", G.spfa(1, n));
    }
    return 0;
}
{% endhighlight %}

两大最强最短路算法，**SPFA和Dijkstra算法的比较：**

SPFA：执行松弛操作，用队列里有的点去刷新起始点到所有点的最短路，如果刷新成功且被刷新点不在队列中则把该点加入到队列最后，判断负环的话，需要记录一个节点的入队次数，超过\|V\|则存在负环。用SPFA的时侯，同一个节点可能会多次入队，然后多次去刷新其他节点，这样就会导致最短路条数出现重复计算（所以才能判断负环)，而Dijkstra使用优先队列，虽然同一个点可以多次入队，但是mark数组保证了一个点真正pop出来刷新其他点的时候只有一次，而且必定满足最短路！

SPFA 和 Dijkstra 同一个节点都有可能入队多次， 但是，由于Dijkstra算法不考虑负环的情况，使用优先队列，则每次pop()出来的都是最小的权值的点去刷新其他的点，因此可以保证满足最短路，同时用mark数组控制 可以保证每个节点出来刷新其他节点的机会只有一次 ，因而保证了计算最短路径的次数不会出现重复。

而SPFA考虑到负环的情况，每个节点可以多次刷新其他节点，以此才能得到最短路并且判断是否存在负环，使用时只需要确保相同节点不要同时出现在队列中即可，由于每个节点可以多次刷新其他节点， 因此，计算最短路径数时会有重复。

也就是，如果有负权边，则使用SPFA；如果都是正权边，Dijkstra + 优先队列效率更高，且更靠谱些。

###四、Floyd算法

前边的三种算法都是单源最短路算法，也就是用于求两点间的最短路，而Floyd是APSP(All Pairs Shortest Paths)，也就是所有顶点对之间的最短路径，理解这个算法，要用到一些矩阵乘法的知识，这个我在下下篇笔记中会写，Bloyd用矩阵记录图，是一种动态规划算法，稠密图效果最佳，边权可正可负。此算法简单有效，由于三重循环结构紧凑，对于稠密图，效率要高于执行\|V\|次Dijkstra算法。

Floyd算法，代码简单，可以算出任意另个结点间的距离，但是复杂度较高，达到O(n^3)，所以不适合有大量数据的运算。

Floyd算法的基本思想：从矩阵$A^{-1}$开始，依次生成矩阵$A^0， A^1，A^2$，……，A^{n – 1}。如果已经生成矩阵A$^{k – 1}$，那么就可以生成$A^k$，因为对于任意一对顶点 i 和 j ，一定满足下面两条规则中的一条：

1）如果 k 不是路径 p 的中间顶点，则 p 的所有中间顶点皆在{1，2，……，k – 1}中，其路径代价为$A^{k-1}$[i][j]。

2）如果 k 是路径 p 的中间顶点，那么该路径由从 i 到 k 的路径和从 k 到 j 的路径两部分构成，由于这两条子路径上的顶点序号都不大于k – 1，因此其路径代码分别为$A^{k – 1}$[i][k]和$A^{k – 1}$ [k][j] 。

基于上述两条规则，可以得到如下求解$A^k$[i][j]的公式：

$$
A^k[i][j] = min { A^k – 1[i][j]，A^{k – 1}[i][k] + A^k – 1[k][j] }，k >= 0 和 A^{-1}[i][j] = w_{ij}
$$

还是上边提到的题目，Floyd算法实现：

{% highlight c %}
#include<iostream>                                      
#include<cstdio>                                        
#include<cstring>                                                                             
#include<cmath>                                         
#include<algorithm>     
#include<climits>
 
const int N = 101;
int map[N][N];
 
void Floyd(int n) {
    for (int k = 1; k <= n; k++) {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                if((map[i][k] != INT_MAX) && (map[k][j] != INT_MAX)
                    &&(map[i][j] > map[i][k] + map[k][j] || map[i][j] == INT_MAX)) {
                        map[i][j] = map[i][k] + map[k][j];
                }
            }
        }
    }
}
 
int main() {
    int n, m;
    while (~scanf("%d%d", &n, &m), n | m) {
        for (int i = 1; i <= n; i++) {
            for (int j = i; j <= n; j++) {
                map[i][j] = map[j][i] = INT_MAX;
            }
        }
        while (m--) {
            int u, v, w;
            scanf("%d%d%d", &u, &v, &w);
            map[u][v] = map[v][u] = w;
        }
        Floyd(n);
        printf("%d\n",map[1][n]);
    }

    return 0;
}
{% endhighlight %}

<hr/>
###一些评论
wangyu说道：2012/06/07 15:40  
: sfpa，若w(A,B)=5,w(B,A)=-6,会出现死循环吧？还是我哪个部分没有理解到。请指教。谢谢你的博客，学习了很多。

Wei Li说道：2012/06/08 00:12  
: 你说的这种情况，确实会出现死循环。因为是有向图，所以这种情况下，不存在最短路径，所以SPFA算法失效。使用SPFA算法的前提是，图G中不存在负权回路。

实际应用中，应该会先检测是否存在负权值回路，如果不存在，才可以使用SPFA。

检测负环，可以给结点加一个值，用来计算该值更新了几次，如果一个节点更新了 n 次，那么存在负环。或者，也可以用拓扑排序来检测是否存在回路。

wangyu说道： 2012/06/08 09:20  
: 对于如何用拓扑排序判断负环回路，我不明白，能再详细说明下吗？谢谢。另外，我是在spfa里面，给顶点增加了一个更新计数，若是超过了n，就退出说明存在负环。

Wei Li说道：2012/06/08 10:23 
: 拓扑排序只能判断有向图有无回路，因为原理是DFS + 入度值，想了想，没有一种简单的办法可以加上权值计算，所以只能判断是否有环，负环不好判断。
拓扑排序判断是否DAG（Directed Acyclic Graph），这篇里涉及了些：《[图搜索算法(Graph Search Algorithm)](/2011/10/28/graph-search/)》

[1]: /uploads/2011/11/松弛操作1.png
[2]: /uploads/2011/11/Dijkstra图示.png
[3]: /uploads/2011/11/Bellon-Ford图示.png