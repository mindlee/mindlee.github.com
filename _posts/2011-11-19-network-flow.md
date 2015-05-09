---
title: 网络流(Network Flow)
author: Wei Li
excerpt: 将每条有向边想象成传输物质的管道。每个管道都有一个固定的容量，可以看作是物质能流经该管道的最大速度（譬如可以想象为水流和河槽）。顶点是管道间的连接点，除了源点（S，Source）和汇点（T，Target）以外，物质只流经这些顶点。而不聚集在顶点中。
meta_description: 将每条有向边想象成传输物质的管道。每个管道都有一个固定的容量，可以看作是物质能流经该管道的最大速度（譬如可以想象为水流和河槽）。顶点是管道间的连接点，除了源点（S，Source）和汇点（T，Target）以外，物质只流经这些顶点。而不聚集在顶点中。
layout: post
permalink: /2011/11/19/network-flow/
views:
  - 15718
categories:
  - 算法学习
tags:
  - 图论
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
![Image][1]
将每条有向边想象成传输物质的管道。每个管道都有一个固定的容量，可以看作是物质能流经该管道的最大速度（譬如可以想象为水流和河槽）。顶点是管道间的连接点，除了源点（S，Source）和汇点（T，Target）以外，物质只流经这些顶点。而不聚集在顶点中。

注：下文提到的数字，基本都可加单位 “单位流量”来理解。

###一、网络流基础

网络流中的最大流研究的问题是：在不违背容量限制的条件下，把物质从源点传输到汇点的最大速率是多少？上边的图示中，还有些需要说明，设G = （V，E）是一个流网络，其容量函数为c（c(u, v)，Capicity）。s 为源点，t 为汇点。G 的流是一个函数 f （Flow）。上图中每条边标记的是流网络的容量 c(u, v)，右图中，G中的流 \|f\| = 19。图中只显示正网络流。如果 f（u，v） > 0，则标记（u，v）为 f（u，v）/ c（u，v）（斜杠仅仅用来区分流和容量，不表示相除）。如果 f（u，v）<= 0，边（u，v）只标记它的容量。譬如（s，v1）这条边，最大流量限制是16，但实际流过11，所以表示为11 / 16。

网络流算法要基于三种思想：残留网络（Residual Network），增广路径（Augmenting Path）和割（Cut）。
![Image][2]
和开头同一个例子，上图，图（b）就是**残留网络**，仔细观察一下，应该就可以明白，譬如边（s，v1），从 s 到 v1 流过11，还剩5，所以残余容量是5，当然，流量是守恒的，所以反向从 v1 到 s 流过11。剩余的容量 + 反向平衡的流量共同构成了残留网络。对于名词“残留容量（Residual Capacity）”的定义：在不超过容量c（u，v）的条件下，从 u 到 v 之间可以压入的额外网络流量，就是（u，v）的残留容量（Residual Capacity），公式定义是：cf（u，v） = c（u，v） – f（u，v）。而残留网络Gf = （V，Ef）。

图（b），阴影覆盖的边为**增广路径** p ，其残留容量为 cf（p） = cf（v2，v3） = 4。看图应该就可以大概理解什么是增广路了。看图可以发现 s 到 v2还可以通过 5，v2到v3还可以通过 4，v3到 t 还可以流 5，按照常识，也可得出，这条路径还可以再流过 4 的结论。而从 s 输送 4 到 t 的这条路就是增广路。"增广路径 p”的定义：p 为残留网络Gf中从 s 到 t 的一条简单路径。在不违反边的容量限制条件下，增广路径上的每条边（u，v）可以容纳从 u 到 v 的某额外正网络流。Starvae师兄在讲解增广路时，有一段通俗的描述：

>假如有这么一条路，这条路从源点开始一直一段一段的连到了汇点，并且，这条路上的每一段都满足流量<容量，注意，是严格的<,而不是<=。那么，我们一定能找到这条路上的每一段的(容量-流量)的值当中的最小值delta。我们把这条路上每一段的流量都加上这个delta，一定可以保证这个流依然是可行流。这样我们就得到了一个更大的流，他的流量是之前的流量+delta，而这条路就叫做增广路。

另外，这里得出的，计算增广路流量的公式非常重要：cf（p） = min { cf（u，v）：（u，v）在 p 上}，因为最大流算法的核心基本也就是寻找增广路了。

**最大流最小割定理：一个流是最大流，当且仅当它的残留网络不包含增广路径。**
![Image][3]
流网络的**割（S，T）**将V划分为 S 和 T = V – S两部分，使得 s ∈ S，t ∈ T。如果 f 是一个流，则穿过割 （S，T）的净流被定义为 f (S，T）。割（S，T）的容量为 c（S，T）。一个网络的最小割就是网络中所有割中最小容量的割。

上图中，流网络的割（S = {s，v1，v2}，T = {v3，v4，t}），S中的顶点是黑色，T中的顶点是白色。穿越（S，T）的净流量为：f（v1，v3） + f（v2，v3） + f（v2，v4） = 12 + （-4） + 11 = 19；容量为：c（v1，v3） + c（v2，v4） = 12 + 14 = 26。可以看出，割的净流由双向的网络流组成。而割的容量仅由 S 到 T 的边计算而得。
![Image][4]
还有一个很有重要的知识：**反向边**。如图（a），1是源点，4是汇点。很明显如果第一次迭代找到的增广路是1→2→4和1→3→4，则最大流是2，但是如果第一次迭代找到的增广路是1→2→3→4，那么流量只有1，如图（b）是残留网络，这时候，反向边就起作用了，由于反向边的原因，第二次迭代的时候，又找到一条增广路1→2→3→4。这样下来，总的流量还是2。还是Starvae师兄的解释：

>当我们第二次的增广路走 3→2 这条反向边的时候，就相当于把 2→3 这条正向边已经是用了的流量给“退”了回去，不走 2→3 这条路，而改走从 2 点出发的其他的路也就是 2→4。（有人问如果这里没有 2→4 怎么办，这时假如没有 2→4 这条路的话，最终这条增广路也不会存在，因为他根本不能走到汇点）同时本来在 3→4 上的流量由 1→3→4 这条路来“接管”。而最终 2→3 这条路正向流量1，反向流量1，等于没有流量。反向边的作用就是给程序一个可以后悔的机会。

###二、Ford-Fulkerson算法

Ford-Fulkerson算法在实际中并不常用，但是它提供了一种思想：先找到一条从源点到汇点的增广路径，这条路径的容量是其中容量最小的边的容量。然后，通过不断找增广路，一步步扩大流量，当找不到增广路时，就得到最大流了（最大流最小割定理）。可以看看Ford-Fulkerson算法的伪代码，

	FORD-FULKERSON(G, s, t)
	1  for each edge (u, v) ∈ E[G]
	2       do f[u, v] ← 0
	3          f[v, u] ← 0
	4  while there exists a path p from s to t in the residual network Gf
	5      do cf(p) ← min {cf(u, v) : (u, v) is in p}
	6         for each edge (u, v) in p
	7             do f[u, v] ← f[u, v] + cf(p)
	8                f[v, u] ← -f[u, v]

从伪代码可以看出，Ford-Fulkerson的核心过程就是：第4~8行的 while 循环反复找出 Gf 中的增广路径 p，并把沿 p 的流 f 加上其残留容量cf（p）。当不再有增广路径时，流 f 就是一个最大流。完整图示：
![Image][5]
![Image][6]

(f）最后在 while 循环测试的残留网络，它没有增光路径，所以（e）中显示的流就是最大流。

###三、Edmonds-Karp（EK）算法

EK算法基于Ford-Fulkerson算法，唯一的区别是将第 4 行用BFS（广度优先搜索）来实现对增广路径 p 的计算。EK算法伪代码基本和上边的Ford-Fulkerson算法一样。类似用DFS实现的还有Dinic算法。它们都属于SAP（Shortest Augmenting Path）算法，从英文即可看出，它们每次都在寻找最短增广路。对于EK算法，每次用一遍 BFS 寻找从源点 s 到终点 t 的最短路作为增广路径，然后增广流量 f 并修改残量网络，直到不存在新的增广路径。E-K 算法的时间复杂度为 O(VE2)，由于 BFS 要搜索全部小于最短距离的分支路径之后才能找到终点，因此频繁的 BFS 效率是比较低的。实践中此算法使用的机会较少。

关于EK算法的实现，先从一道题目开始，最简单的算法练习题：[HDU 3549 Flow Problem](http://acm.hdu.edu.cn/showproblem.php?pid=3549) ，题目大意：N个点，M条边。每条边都是以X to Y，Capacity 是 C的格式。问最后 1 到 N 的最大流？对于算法实现的讲解，在下边邻接矩阵的例子里看注释即可。

1）邻接矩阵实现，便于理解的方式：
{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
#include<queue>
#include<climits>
using namespace std;
 
#define N 16
int capacity[N][N];//容量
int flow[N];//残余流量
int pre[N];//前趋
 
int n, m;
queue<int> Q;
 
int BFS(int src, int des) {
    //初始化
    while (!Q.empty()) {
        Q.pop();
    }
    for (int i = 1; i < n + 1; i++) {
        pre[i] = -1;
    }
    pre[src] = 0;
    flow[src] = INT_MAX;//初始化源点的流量为无穷大 
    Q.push(src);
    while (!Q.empty()) {
        int index = Q.front();
        Q.pop();
        if (index == des) {//找到了增广路径 
            break;
        }
        for (int i = 1; i < n + 1; i++) {
            if (i != src && capacity[index][i] > 0 && pre[i] == -1) {
                pre[i] = index;
                //增广路残容流量
                flow[i] = min(capacity[index][i], flow[index]);
                Q.push(i);
            }
        }
    }//while
    if (pre[des] == -1) {
        return -1;  //残留图中不存在增广路径
    } else {
        return flow[des];
    }
}
 
int MaxFlow(int src, int des) {
        int aug = 0;
        int sumflow = 0;
        while ((aug = BFS (src, des)) != -1) {
            int k = des;  //利用前驱寻找路径 
            while (k != src) {
                int last = pre[k];
                capacity[last][k] -= aug;
                capacity[k][last] += aug;
                k = last;
            }
            sumflow += aug;
        }
        return sumflow;
}
             
int main() {
    int cas, cases = 1;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d", &n, &m);
        memset(capacity, 0, sizeof(capacity));
        memset(flow, 0, sizeof(flow));
        for (int i = 0; i < m; i++) {
            int u, v, w;
            scanf("%d%d%d", &u, &v, &w);
            if (u == v) {//起点终点相同
                continue;
            }
            capacity[u][v] += w;
        }
        printf("Case %d: ", cases++);
        printf("%d\n", MaxFlow(1, n));
    }
    return 0;
}
{% endhighlight %}

2）邻接表也是可以的，实现：
{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<climits>
using namespace std;
 
const int NV = 16;
const int NE = 2002;
 
struct EK {
    int n, size;
    int head[NV];
    int que[NV], pre[NV], cur[NV];
    int front, rear;
    int maxflow;
 
    struct Edge {
        int v, w, next;
        Edge () {}
        Edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) { }
    }E[NE];
 
    void init(int x) {
        n = x, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
        E[size] = Edge(u, head[v], 0);
        head[v] = size++;
    }
 
    int MaxFlow(int src, int des) {
        maxflow = 0;
        while (true) {
            memset(pre, -1, sizeof(pre));
            que[front = rear = 0] = src;
            while (front <=  rear) {
                int u = que[front++];
                for (int i = head[u]; i != -1; i = E[i].next) {
                    int v = E[i].v;
                    if (pre[v] == -1 && E[i].w) {
                        pre[v] = u;
                        cur[v] = i;
                        que[++rear] = v;
                    }
                }//for
                if (pre[des] != -1) {
                    break;
                }
            }//while
            if (pre[des] == -1) {
                break;
            }
            int aug = INT_MAX;
            for (int v = des; v != src; v = pre[v]) {
                aug = min(aug, E[cur[v]].w);
            }
            for (int v = des; v != src; v = pre[v]) {
                E[cur[v]].w -= aug;
                E[cur[v]^1].w += aug;
            }
            maxflow += aug;
        }
        return maxflow;
    }
}G;
 
int main() {
    int cas, n, m;
    int cases = 1;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d", &n, &m);
        G.init(n);
        while (m--) {
            int u, v, w;
            scanf("%d%d%d", &u, &v, &w);
            if (u == v) {
                continue;
            }
            G.insert(u, v, w);
        }
        printf("Case %d: ", cases++);
        printf("%d\n", G.MaxFlow(1, n));
    }
    return 0;
}
{% endhighlight %}

###四、Improved SAP（ISAP）算法

ISAP字面意思是改良的最短增广路算法。关于ISAP，一位叫 DD_engi 的神牛讲非常清楚，引用一下：

	SAP算法（by dd_engi）：求最大流有一种经典的算法，就是每次找增广路时用BFS找，保证找到的增广路是弧数最少的，也就是所谓的 Edmonds-Karp 算法。可以证明的是在使用最短路增广时增广过程不超过 V * E次，每次 BFS 的时间都是O(E)，所以 Edmonds-Karp 的时间复杂度就是O(V * E^2)。

	如果能让每次寻找增广路时的时间复杂度降下来，那么就能提高算法效率了，使用距离标号的最短增广路算法就是这样的。所谓距离标号，就是某个点到汇点的最少的弧的数量（另外一种距离标号是从源点到该点的最少的弧的数量，本质上没什么区别）。设点 i 的标号为D[i]，那么如果将满足D[i] = D[j] + 1的弧（i，j）)叫做允许弧，且增广时只走允许弧，那么就可以达到“怎么走都是最短路”的效果。每个点的初始标号可以在一开始用一次从汇点沿所有反向边的BFS求出，实践中可以初始设全部点的距离标号为0，问题就是如何在增广过程中维护这个距离标号。

	维护距离标号的方法是这样的：当找增广路过程中发现某点出发没有允许弧时，将这个点的距离标号设为由它出发的所有弧的终点的距离标号的最小值加一。这种维护距离标号的方法的正确性我就不证了。由于距离标号的存在，由于“怎么走都是最短路”，所以就可以采用DFS找增广路，用一个栈保存当前路径的弧即可。当某个点的距离标号被改变时，栈中指向它的那条弧肯定已经不是允许弧了，所以就让它出栈，并继续用栈顶的弧的端点增广。为了使每次找增广路的时间变成均摊O(V)，还有一个重要的优化是对于每个点保存“当前弧”：初始时当前弧是邻接表的第一条弧；在邻接表中查找时从当前弧开始查找，找到了一条允许弧，就把这条弧设为当前弧；改变距离标号时，把当前弧重新设为邻接表的第一条弧，还有一种在常数上有所优化的写法是改变距离标号时把当前弧设为那条提供了最小标号的弧。当前弧的写法之所以正确就在于任何时候我们都能保证在邻接表中当前弧的前面肯定不存在允许弧。

	还有一个常数优化是在每次找到路径并增广完毕之后不要将路径中所有的顶点退栈，而是只将瓶颈边以及之后的边退栈，这是借鉴了Dinic算法的思想。注意任何时候待增广的“当前点”都应该是栈顶的点的终点。这的确只是一个常数优化，由于当前边结构的存在，我们肯定可以在O(n)的时间内复原路径中瓶颈边之前的所有边。

	优化：
	
	1.邻接表优化：

	如果顶点多的话，往往N^2存不下，这时候就要存边：
	存每条边的出发点，终止点和价值，然后排序一下，再记录每个出发点的位置。以后要调用从出发点出发的边时候，只需要从记录的位置开始找即可（其实可以用链表）。优点是时间加快空间节省，缺点是编程复杂度将变大，所以在题目允许的情况下，建议使用邻接矩阵。
	
	2.GAP优化：

	如果一次重标号时，出现距离断层，则可以证明ST无可行流，此时则可以直接退出算法。
	
	3.当前弧优化：

	为了使每次找增广路的时间变成均摊O(V)，还有一个重要的优化是对于每个点保存“当前弧”：初始时当前弧是邻接表的第一条弧；在邻接表中查找时从当前弧开始查找，找到了一条允许弧，就把这条弧设为当前弧；改变距离标号时，把当前弧重新设为邻接表的第一条弧。

为了使每次找增广路的时间变成均摊O(V)，还有一个重要的优化是对于每个点保存“当前弧”：初始时当前弧是邻接表的第一条弧；在邻接表中查找时从当前弧开始查找，找到了一条允许弧，就把这条弧设为当前弧；改变距离标号时，把当前弧重新设为邻接表的第一条弧。
另外，ISAP简化的描述是：程序开始时用一个反向 BFS 初始化所有顶点的距离标号，之后从源点开始，进行如下三种操作：(1)当前顶点 i 为终点时增广 (2) 当前顶点有满足 dist[i] = dist[j] + 1 的出弧时前进 (3) 当前顶点无满足条件的出弧时重标号并回退一步。整个循环当源点 s 的距离标号 dist[s] >= n 时结束。对 i 点的重标号操作可概括为 dist[i] = 1 + min{dist[j] : (i,j)属于残量网络Gf}。

#####对于EK算法与ISAP算法的区别：

EK算法每次都要重新寻找增广路，寻找过程只受残余网络的影响，如果改变残余网络，则增广路的寻找也会随之改变；SAP算法预处理出了增广路的寻找大致路径，若中途改变残余网络，则此算法将重新进行。EK处理在运算过程中需要不断加边的最大流比SAP更有优势。

还是上边那道题，用ISAP算法实现：（可当模板）

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
#include<climits>
using namespace std;
 
const int NV = 16;
const int NE = 2002;
 
struct ISAP {
    int n, size;
    int head[NV];
    int dis[NV], gap[NV], pre[NV], cur[NV];
    int maxflow;
 
    struct Edge {
        int v, w, next;
        Edge () {}
        Edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) { }
    }E[NE];
 
    void init(int x) {
        n = x, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
        E[size] = Edge(u, head[v], 0);
        head[v] = size++;
    }
 
    int MaxFlow(int src, int des) {
        maxflow = 0;
        gap[0] = n;
        for (int i = 0; i <= n; i++) {
            dis[i] = gap[i] = 0;
        }
        for (int i = 0; i <= n; i++) {
            cur[i] = head[i];
        }
        int u = pre[src] = src;
        int aug = -1;
        while (dis[src] < n) {//结束条件1
loop: 
            for (int &i = cur[u]; i != -1; i = E[i].next) {
                int v = E[i].v;
                if (E[i].w && dis[u] == dis[v] + 1) {
                    aug = min(aug, E[i].w);
                    pre[v] = u;
                    u = v;
                    if (v == des) {//找到一条增广路，更新
                        maxflow += aug;
                        // //修改残余网络
                        for (u = pre[u]; v != src; v = u, u = pre[u]) {
                            E[cur[u]].w -= aug;//正向边
                            E[cur[u]^1].w += aug;//反向边
                        }
                        aug = INT_MAX;
                    }//if
                    goto loop;
                }//for
            }//for
            //寻找最小的距离标号，并修改当前点    为最小的标号+1
            int mdis = n;
            for (int i = head[u]; i != -1; i = E[i].next) {
                int v = E[i].v;
                if (E[i].w && mdis > dis[v]) {
                    cur[u] = i;
                    mdis = dis[v];
                }
            }//for
 
            //GAP 优化 断层则跳出 结束条件2
            if ((--gap[dis[u]]) == 0) {
                break;
            }
            gap[dis[u] = mdis + 1]++;//将拥有该标号的数量加1
            u = pre[u];//当前节点 迁移一个
        }//while
        return maxflow;
    }//ISAP
}G;
 
int main() {
    int cas, n, m;
    int cases = 1;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d", &n, &m);
        G.init(n);
        while (m--) {
            int u, v, w;
            scanf("%d%d%d", &u, &v, &w);
            if (u == v) {
                continue;
            }
            G.insert(u, v, w);
        }
        printf("Case %d: ", cases++);
        printf("%d\n", G.MaxFlow(1, n));
    }
    return 0;
}
{% endhighlight %}

本文只介绍基础的网络流知识，网络流非常强大，许多问题都可以转化为网络流模型，进一步的，可以去看看Starfall大神的这篇《【网络流】总结》，这篇只是一个目录性质，里边很多超链接，后面资源更丰富。其中提到了6篇国家集训队论文，正好电脑里都有，就打包传到CSDN了，[点击跳转到下载页面](http://download.csdn.net/detail/welon123/3813929)。我只看过其中的两三篇，这帮高中生写的实在牛叉。

记得当年看时，觉得这篇《最大流在信息学竞赛中应用的一个模型》当做入门非常好，看完后会发现，原来组合数学都可以用最大流来解，还有什么不可以的。另外，最经典，最全面的要数胡波涛这篇的《最小割模型在信息学竞赛中的应用》，如果想深入学习网络流，这帮家伙的论文绝对不能错过。
<hr/>

###一些评论
wangyu说道：2012/06/15 08:52  

: 看了lz的代码，有个疑问对于a→b和b→a的容量同时存在的话，邻接表实现要如何处理？

酷~行天下说道：2012/06/22 23:09  
: 不好意思，最近几天不在学校，现在才回复。
a→b, b→a容量同时存在，你的意思，是不是担心在建边的时候，会出现下边的代码，造成a→b寻找增广路的混乱？

{% highlight c %}
E[size] = Edge(v, head[u], w1);
head[u] = size++;
E[size] = Edge(u, head[v], 0);
head[v] = size++;
______________________________
E[size] = Edge(u, head[v], w2);
head[v] = size++;
E[size] = Edge(v, head[u], 0);
head[u] = size++;
{% endhighlight %}
这种情况应该不会妨碍寻找增广路，假如a→b，w = 3; b→a = 2, 寻找从a到b的最大流，邻接表建边：
a b 3
b a 0
b a 2
a b 0
b→a的容量是2，但是它建的反向边是a b 0，这个值对从a到b找增广路不造成任何影响。因为aug = 0没有任何影响。

[1]: /uploads/2011/11/流网络图示.png
[2]: /uploads/2011/11/残留网络.png
[3]: /uploads/2011/11/网络流_割.png
[4]: /uploads/2011/11/反向边.png
[5]: /uploads/2011/11/Ford-Fulkerson1.png
[6]: /uploads/2011/11/Ford-Fulkerson2.png
