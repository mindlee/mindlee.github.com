---
title: 图搜索算法(Graph Search Algorithm)
author: Wei Li
excerpt: 搜索一个图指是有序地沿着图的边访问所有顶点。搜索过程实际上是根据初始条件和扩展规则构造一棵解答树并寻找符合目标状态的节点的过程。
meta_description: 搜索一个图指是有序地沿着图的边访问所有顶点。搜索过程实际上是根据初始条件和扩展规则构造一棵解答树并寻找符合目标状态的节点的过程。
layout: post
permalink: /2011/10/28/graph-search/
views:
  - 7481
categories:
  - 算法学习
tags:
  - 图论
  - 数据结构
  - 算法
  - 算法导论
---
搜索一个图指是有序地沿着图的边访问所有顶点。搜索过程实际上是根据初始条件和扩展规则构造一棵解答树并寻找符合目标状态的节点的过程。另外，搜索算法是利用计算机的高性能来有目的地穷举一个问题的部分或所有的可能情况，从而求出问题的解的一种方法。许多图算法在开始时，都是通过搜索输入的图来获取图结构的信息，另外还有一些图的算法时由基本的图搜索算法经过简单扩充而成。因此，图的搜索技术是图算法领域的核心。

最常用的图表示方法有两种：邻接表，邻接矩阵。还有类似十字链表，邻接多重表等表示方法，不过应该极少使用。当年课程设计用 邻接多重表 表示图，直接被老师规劝了，邻接表适用于稀疏图，邻接矩阵适用于稠密图，应该不用解释，一切都是为了节省资源。通常使用的是邻接表，因为正常生活中遇到的图都不会过于稠密。另外，由于邻接矩阵简单明了，当图较小时，更多地采用邻接矩阵。如果一个图不是加权的，采用邻接矩阵还有一个好处：在储存邻接矩阵的每个元素时，可以只使用一个二进位，而不必用一个字的空间。
![Image][1]
上边图中，(a)是图的邻接表表示，(b)是图的邻接矩阵表示。图22-1 >>> a)一个有5个顶点和7条边的无向图G。b)G的邻接表表示。c)G的邻接矩阵表示。下图u示邻接矩阵表示法，图22-2 >>> a)有6个顶点和8条边的有向图G。b)G的邻接表表示。c)G的邻接矩阵表示。

基本的图搜索算法有两种：广度优先搜索（Breadth-first Search）,深度优先搜索（Depth-first Search）,另外很多高级图算法的核心过程都用到了这两种搜索思想，比如A*，IDA*，双向广搜，计算有向图强连通分量（比如Kosaraju算法和Tarjan算法）等等。本文只介绍搜索基础知识，以及一些简单扩展，比如利用BFS判断二分图，利用DFS构造拓扑排序等。如果是关于ACM的高级搜索算法，建议去看看Amb大牛的这篇《[HDU 搜索进阶专题](http://www.cnblogs.com/ambition/archive/2011/07/25/search_plus.html)》。

解释搜索过程，最好的方式就是演示，这里有个关于A*， Dijkstra，双向BFS的寻路算法演示视频：

<div align="center">
<embed src="http://player.youku.com/player.php/sid/XMjQzMTYzNjg4/v.swf" allowFullScreen="true" quality="high" width="600" height="500" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash">
</div>

### 一、广度优先搜索（BFS）

广度优先搜索是很多重要的图算法的原型。在Prim最小生成树和Dijkstra单源最短路径算法中，都采用了与广度优先搜索类似的思想。

在给定图G = （V,E）和一个特定的源顶点s的情况下，广度优先搜索系统地探索G中的边，以期“发现”可从s到达的所有顶点，并计算s到所有这些可达顶点之间的距离（即最少的边数）。BFS同时还能生成一棵根为s、且包括所有s的可达顶点的广度优先树。对从s可达的任意顶点v，广度优先树中从s到v的路径对应于图G中从s到v的一条最短路径，即包含最少边数的路径。BFS之所以称之为广度优先搜索，是因为，它自始至终都是从一个结点沿其广度方向向外扩展。BFS算法自始至终一直通过已找到和未找到顶点之间的边界向外扩展，就是说，算法首先搜索和s距离为k的所有顶点，然后再去搜索和S距离为k + l的其他顶点。

为了保持搜索的轨迹，广度优先搜索为每个顶点着色：白色、灰色或黑色。算法开始前所有顶点都是白色，随着搜索的进行，各顶点会逐渐变成灰色，然后成为黑色。在搜索中第一次碰到一顶点时，我们说该顶点被发现，此时该顶点变为非白色顶点。因此，灰色和黑色顶点都已被发现，但是，广度优先搜索算法对它们加以区分以保证搜索以广度优先的方式执行。若(u, v)∈E且顶点u为黑色，那么顶点v要么是灰色，要么是黑色，就是说，所有和黑色顶点邻接的顶点都已被发现。灰色顶点可以与一些白色顶点相邻接，它们代表着已找到和未找到顶点之间的边界。

伪代码：

	BFS(G, s)
	1  for each vertex u ∈ V [G] – {s}
	2       do color[u] ← WHITE
	3          d[u] ← ∞
	4          π[u] ← NIL
	5  color[s] ← GRAY
	6  d[s] ← 0
	7  π[s] ← NIL
	8  Q ← Ø
	9  ENQUEUE(Q, s)
	10  while Q ≠ Ø
	11      do u ← DEQUEUE(Q)
	12         for each v ∈ Adj[u]
	13             do if color[v] = WHITE
	14                   then color[v] ← GRAY
	15                        d[v] ← d[u] + 1
	16                        π[v] ← u
	17                        ENQUEUE(Q, v)
	18         color[u] ← BLACK

图示过程，下边是BFS在一个无向图上的处理过程。树中的边在由BFS产生时即用阴影显示。在每个顶点u内示出了d[u]。在过程的第10~18行中，在while循环的每一轮迭代的开始时显示队列Q。顶点间距离在队列中顶点的旁边示出。
![Image][2]
图示过程应该比较好理解。如果没接触过BFS，对于加深理解，可以看看几个简单BFS的题目，兴许比上边的伪代码好理解：

简单BFS，可以看看这个简单的模板：

{% highlight c %}
//queue  -->>  Q.front();
//priority_queue ->> Q.top();
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
using namespace std;
#define N 21
 
struct Node {
    int x, y;
};
 
int sx, sy, ex, ey;    
int n, m;
char map[N][N];
int mark[N][N];
int dir[4][2] = { {0, 1}, {0, -1}, {1, 0}, {-1, 0} };
 
//判断是否越界 
bool Overmap(int x, int y) {
    if (x < 0 || x >= n || y < 0 || y >= m) {
        return true;
    }
    else {
        return false;
    }
}
 
int Bfs() {
    memset(mark, 0, sizeof(mark));
    queue<Node> Q;
    Node first, next;
    first.x = sx, first.y = sy;
    mark[sx][sy] = 1;
    Q.push(first);
    int cnt = 0;
    while (!Q.empty()) {
        cnt++;
        first = Q.front();
        Q.pop();
        for (int i = 0; i < 4; i++) {
            next.x = first.x + dir[i][0];
            next.y = first.y + dir[i][1];
            if (Overmap(next.x, next.y) || mark[next.x][next.y] || map[next.x][next.y] == '#') {
                continue;
            }
            mark[next.x][next.y] = 1;
            Q.push(next);
        }//for
    }//while
    return cnt;
}
{% endhighlight %}
HDU一些BFS题目，后两道可能难点

1） HDU     1072     [Nightmare](http://acm.hdu.edu.cn/showproblem.php?pid=1072)

{% highlight c %}
/*
需要输出从起点到终点的最短步数
同时时间，6s递减，需要经过补充时间的点，
要让步数最少，
2  1  0   1  1
1  1  0   3  1
0  1  0   1  1
4  1  1   1  1
这个例子，2-》3，必定要过4，然后在绕出来
所以此时，bfs不是利用是否早过来标记
而是利用，同时防止倒着走回去
比如最后行是1-》4-》1
时间是          2-》6-》5
 
 对应      2          5
if (mark[next.x][next.y] < next.tme) {
    mark[next.x][next.y] = next.tme;
    Q.push(next);
}
 */
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
#include<stack>
using namespace std;
#define N 9
 
struct Node {
    int x;
    int y;
    int tme;
    int step;
};
 
int cas, n, m;
int sx, sy, ex, ey;
int map[N][N];
int mark[N][N];
int dir[4][2] = { {0, -1}, {0, 1}, {1, 0}, {-1, 0} };
 
 
bool Overmap(int x, int y) {
    if (x < 0 || x >= n || y < 0 || y >= m) {
        return true;
    }
    else {
        return false;
    }
}
 
int Bfs() {
    memset(mark, 0, sizeof(mark));
    queue<Node> Q;
    Node first, next;
    first.x = sx;
    first.y = sy;
    first.tme = 6;
    first.step = 0;
    Q.push(first);
    mark[first.x][first.y] = 6;
 
    while (!Q.empty()) {
        first = Q.front();
        Q.pop();
        if (first.x == ex && first.y == ey) {
            return first.step;
        }
 
        if (map[first.x][first.y] == 4) {
            first.tme = 6;
            mark[first.x][first.y] = 6;
        }
 
        for (int i = 0; i < 4; i++) {
            next.x = first.x + dir[i][0];
            next.y = first.y + dir[i][1];
            next.tme = first.tme - 1;
            next.step = first.step + 1;
            if (Overmap(next.x, next.y) || map[next.x][next.y] == 0 || next.tme <= 0) {
                continue;
            }
 
            if (mark[next.x][next.y] < next.tme) {
                mark[next.x][next.y] = next.tme;
                Q.push(next);
            }
        }
    }
    return -1;
}
 
int main() {
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d", &n, &m);
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                scanf("%d", &map[i][j]);
                if (map[i][j] == 2) {
                    sx = i, sy = j;
                }
                if (map[i][j] == 3) {
                    ex = i, ey = j;
                }
            }
        }//for
        printf("%d\n", Bfs());
    }
    return 0;
}
{% endhighlight %}

2） HDU     1728    [逃离迷宫](http://acm.hdu.edu.cn/showproblem.php?pid=1728)
{% highlight c %}
/*
需要记录每次的转弯次数，最后输出结果路径需要转弯次数小于k
这种方法是，采用横冲直撞的方法
每次都是走直线，走过的就不走了，最后肯定能到终点
然后期间转弯也比较好计算，在上次的基础上+1即可
mark[i][j]记录转弯次数
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
#include<stack>
using namespace std;
#define N 101
 
struct Node {
    int x;
    int y;
};
 
int cas, n, m, k;
int sx, sy, ex, ey;
char map[N][N];
int mark[N][N];//《保存到该点时已经过的转弯次数 
int dir[4][2] = { {0, 1}, {0, -1}, {1, 0}, {-1, 0} };
 
bool Overmap(int x, int y) {
    if (x < 0 || x >= n || y < 0 || y >= m) {
        return true;
    } else {
        return false;
    }
}
 
int Bfs() {
    queue<Node> Q;
    Node first, next;
    first.x = sx;
    first.y = sy;
    Q.push(first);
    int cnt;
 
    while (!Q.empty()) {
        first = Q.front();
        Q.pop();
        cnt = mark[first.x][first.y] + 1;//转弯次数 ，初始为0 
        for (int i = 0; i < 4; i++) {
            next.x = first.x + dir[i][0];
            next.y = first.y + dir[i][1];
            while ( !Overmap(next.x, next.y) && map[next.x][next.y] == '.') {
                if (mark[next.x][next.y] == -1) {
                    if (next.x == ex && next.y == ey && cnt <= k) {
                        return 1;
                    }
                    mark[next.x][next.y] = cnt;
                    Q.push(next);
                }
                next.x += dir[i][0];
                next.y += dir[i][1];
            }
        }
    }
    return 0;
}

int main() {
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d", &n, &m);
        cin.ignore();
 
        memset(mark, -1, sizeof(mark));
        for (int i = 0; i < n; i++){
            scanf("%s", &map[i]);
        }
        scanf("%d%d%d%d%d", &k, &sy, &sx, &ey, &ex);
        if (sx == ex && sy == ey) {
            puts("yes");
            continue;
        }
        sx--, sy--, ex--, ey--;
        if (Bfs()) {
            puts("yes");
        } else {
            puts("no");
        }
    }
    return 0;
}
{% endhighlight %}

3） HDU     1175    [连连看](http://acm.hdu.edu.cn/showproblem.php?pid=1175)

{% highlight c %}
/*
BFS（）
主要是要判断是否转弯次数小于2，
可以专门设一数组，用来记录，最小的转弯次数，
lsum 是总的转弯次数
判定条件是：
lsum <= mark[next.x][next.y]
//下一次的转弯次数必定要大于等于上一次的，因为如果，是>，就表示，又绕回去了，不走重复的
如果成立，就入队列
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<algorithm>
#include<queue>
#include<stack>
using namespace std;
#define N 1005
 
struct Node {
#include<cmath> 
    int x;
    int y;
    int di; //方向 
};        
 
int n, m, q;
int sx, sy, ex, ey, lsum;
int map[N][N];
int mark[N][N];
int dir[4][2] = { {0, 1}, {0, -1}, {1, 0}, {-1, 0} };
 
bool Overmap(int x, int y) {
    if (x < 0 || x >= n || y < 0 || y >= m) {
        return true;
    } else {
        return false;
    }
}

int Bfs() {
    memset(mark, 0, sizeof(mark));
    queue<Node> Q;
    Node first, next;
    first.x = sx, first.y = sy;
    first.di = -1;
    mark[first.x][first.y] = 0;
    Q.push(first);
 
    while (!Q.empty()) {
        first = Q.front();
        Q.pop();
        for (int i = 0; i < 4; i++) {
            next.x = first.x + dir[i][0];
            next.y = first.y + dir[i][1];
            next.di = i;
            lsum = 0;
            if (next.di != first.di)  {    //转弯次数，初始是1，所以用3判断
                lsum = mark[first.x][first.y] + 1;
            } else {         
                lsum = mark[first.x][first.y];
            }                
            if (Overmap(next.x, next.y)) { 
                continue;       
            }     
            if (lsum <= mark[next.x][next.y] ||
                mark[next.x][next.y] == 0) {
                if (next.x == ex && next.y == ey 
                    && map[ex][ey] == map[sx][sy] && lsum <= 3) {
                    return 1;
                }
                if (map[next.x][next.y] == 0) {
                    mark[next.x][next.y] = lsum;
                    if (lsum == 4) {//这一句优化了400MS
                        continue;
                    }
                    Q.push(next);
                }    
            }
        }//for
    }//while
    return 0;
}
 
int main() {
    while (scanf("%d%d", &n, &m), n || m) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                scanf("%d", &map[i][j]);
            }
        }
        scanf("%d", &q);
        for (int i = 0; i < q; i++) {
            scanf("%d%d%d%d", &sx, &sy, &ex, &ey);
            sx--, sy--, ex--, ey--;        
            if (map[sx][sy] != map[ex][ey] || map[sx][sy] == 0 || map[ex][ey] == 0) {
                puts("NO");
                continue;
            }
            if (Bfs()) {
                puts("YES");
            } else {
            puts("NO");
            }
        }
    }//while
    return 0;
} 
{% endhighlight %}     

4） HDU     1026    [Ignatius and the Princess I](http://acm.hdu.edu.cn/showproblem.php?pid=1026)
{% highlight c %}
/*
1，在各个点待的时间不一样长
2，要求输出路径
第一点：
用优先队列
struct Node {
    int x, y, x_next, y_next, time;
    friend bool operator < (const Node &a, const Node &b) {
        return a.tme > b.tme;
    }//优先队列
}path[N][N];
 
第二点：
利用数组path[N][N]记录前驱：
path[next.x][next.y] = first; //path用来记录前驱
然后从最后一点开始压栈，压到第一点，然后出栈：
核心代码：
            S.push(final);//先逆序入栈
            node temp = path[final.x][final.y];
            while(temp.x || temp.y)
            {//条件成立时，即到了（0,0）点
                S.push(temp);
                temp = path[temp.x][temp.y];
            }
            S.push(temp);//入栈的即（0,0）点
然后利用while (!Q.empty())出栈即可
*/
#include<iostream>
#include<queue>
#include<fstream>
#include<stack>
using namespace std;
#define N 105
 
struct node {
    int x, y, x_next, y_next, time;
    friend bool operator < (node a, node b) {
        return a.time > b.time;
    }//优先队列
}path[N][N];
 
int n, m, mark[N][N];//记录走过否
char map[N][N];
int dir[4][2] = { {0, -1}, {1, 0}, {0, 1}, {-1, 0} };
node final;
stack <node> S;//栈，输出路径时，出栈即可
 
bool Overmap(int x, int y) {//判断是否越界
    if(x < 0 || x >= n || y < 0 || y >= m)
        return true;
    else
        return false;
}
 
int  Bfs() {
    memset(mark, 0, sizeof(mark));
    priority_queue<node> Q;
    node first, next;
    first.x = 0, first.y = 0, first.time = 0;
    Q.push(first);
    while(!Q.empty()) {
        first = Q.top();
        Q.pop();
        if(first.x == n - 1 && first.y == m - 1) {//结果赋给final
            final.x = first.x;
            final.y = first.y;
            final.x_next = 0;
            final.y_next = 0;
            return first.time;
        }
 
        for(int i = 0; i < 4; i++) {
            next = first;
            next.x += dir[i][0];
            next.y += dir[i][1];
            if(Overmap(next.x, next.y) || 
            	map[next.x][next.y] == 'X' || mark[next.x][next.y])
                continue;
 
            mark[next.x][next.y] = 1;
 
            if(map[next.x][next.y] == '.')
                next.time++;
            else
                next.time += map[next.x][next.y] - '0' + 1;
 
            Q.push(next);
            first.x_next = next.x;
            first.y_next = next.y;
            path[next.x][next.y] = first;//path用来记录前驱
        }
    }
    return -1;
}
 
int main() {
    while(cin >> n >> m) {
        for(int i = 0; i < n; i++) {
            scanf("%s", &map[i]);
        }
        if(Bfs() == -1) {
            puts("God please help our poor hero.");
        } else {
            printf("It takes %d seconds to reach the target position, let me show you the way.\n", Bfs());
 
            S.push(final);//先逆序入栈
            node temp = path[final.x][final.y];
            while(temp.x || temp.y) {//条件成立时，即到了（0,0）点
                S.push(temp);
                temp = path[temp.x][temp.y];
            }
 
            S.push(temp);//入栈的即（0,0）点
            int num = 1;
            while(!S.empty()) {//输出路径时，出栈即可
                temp = S.top();
                S.pop();
                if(map[temp.x][temp.y] == '.' && 
                	!(temp.x_next == 0 && temp.y_next == 0)) {
                    printf("%ds:(%d,%d)->(%d,%d)\n",num++, temp.x, temp.y, temp.x_next, temp.y_next);
                } else if (map[temp.x][temp.y] >= '1' 
                	&& map[temp.x][temp.y] <= '9') {
                    for(int i = 0; i < map[temp.x][temp.y] - '0';  i++) {
                        printf("%ds:FIGHT AT (%d,%d)\n", num++, temp.x, temp.y);
                    }
 
                    if(!(temp.x_next==0 && temp.y_next==0)) {
                        printf("%ds:(%d,%d)->(%d,%d)\n",num++, temp.x, temp.y, temp.x_next, temp.y_next);
                    }
                }
            }
        }
        puts("FINISH");
    }
    return 0;
}
{% endhighlight %}

5） HDU     1253     [胜利大逃亡](http://acm.hdu.edu.cn/showproblem.php?pid=1253)
{% highlight c %}
/*三维广搜，其实一个样，裸广搜题，
注意剪枝就可以了*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
using namespace std;
const int maxn = 51;
 
int dir[6][3] = { {1, 0, 0}, {-1, 0, 0}, {0, 1, 0}, {0, -1, 0}, {0, 0, 1}, {0, 0, -1} };
 
struct Node {
    int x, y, z;
    int tme;//time
};
 
int A, B, C, T;
int map[maxn][maxn][maxn];
int res;
 
inline int Overmap(int x, int y, int z) {
    if (x < 0 || x >= A || y < 0 || y >= B || z < 0 || z >= C) {
        return true;
    } else {
        return false;
    }
}
 
void BFS() {
    queue<Node> que;
    Node first, next;
    first.x = 0, first.y = 0, first.z = 0, first.tme = 0;
    map[first.x][first.y][first.z] = 1;
    que.push(first);
     
    while (!que.empty()) {
        first = que.front();
        que.pop();
        if (first.x == A - 1 && first.y == B - 1 && first.z == C - 1) {
            res = first.tme;
            return ;
        }
 
        for (int i = 0; i < 6; i++) {
            next.x = first.x + dir[i][0];
            next.y = first.y + dir[i][1];
            next.z = first.z + dir[i][2];
            next.tme = first.tme + 1;
 
            if (!Overmap(next.x, next.y, next.z) 
            	&& map[next.x][next.y][next.z] == 0) {
                que.push(next);
                map[next.x][next.y][next.z] = 1;
            }
        }
    }
    res = -1;
    return ;
}
     
int main() { 
    int cas;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d%d%d", &A, &B, &C, &T);
        for (int i = 0; i < A; i++) {
            for (int j = 0; j < B; j++) {
                for (int k = 0; k < C; k++) {
                    scanf("%d", &map[i][j][k]);                 
                }
            }
        }
 
        if (A + B + C - 3 > T || map[A - 1][B - 1][C - 1] == 1) {
            puts("-1");
            continue;
        }
        BFS();
         
        if (res <= T) { 
            printf("%d\n", res);
        } else {
            puts("-1");
        }
    }
    return 0;
}
{% endhighlight %}

6） HDU     1242     [Rescue](http://acm.hdu.edu.cn/showproblem.php?pid=1242)

{% highlight c %}
/*有x点，入队时走到该点并不能保证此时到该点的cnt为a点到该点的最短时间。
必须用优先队列，按时间的从小到大来决定队列次序
然后r有多个，而a只有一个所以可以以a为起点，找到第一个r返回即可 */
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
using namespace std;
 
const int maxn = 201;
struct Node {
    int x, y, tme;
    friend bool operator < (const Node &a, const Node &b) {
        return a.tme > b.tme;
    }//时间从小到大
};
 
int n, m;
int sx, sy;
char map[maxn][maxn];
int mark[maxn][maxn];
int dir[4][2] = { {0, 1}, {0, -1}, {1, 0}, {-1, 0} };
 
bool Overmap(int x, int y) {
    if (x < 0 || x >= n || y < 0 || y >= m) {
        return true;
    }
    return false;
}
 
int BFS() {
    memset(mark, 0, sizeof(mark));
    priority_queue<Node> Q;
    Node first, next;
    first.x = sx, first.y = sy;
    first.tme = 0;
    mark[sx][sy] = true;
    Q.push(first);
    while (!Q.empty()) {
        first = Q.top();
        Q.pop();
        if (map[first.x][first.y] == 'r') {
            return first.tme;
        }
        for (int i = 0; i < 4; i++) {
            next.x = first.x + dir[i][0];
            next.y = first.y + dir[i][1];
            if (Overmap(next.x, next.y) || 
            	map[next.x][next.y] == '#' || mark[next.x][next.y]) {
                continue;
            }
            if (map[next.x][next.y] == 'x') {
                next.tme = first.tme + 2;
            } else {
                next.tme = first.tme + 1;
            }
            mark[next.x][next.y] = 1;
            Q.push(next);
        }
    }
    return -1;
}
 
 
int main() {
    while (~scanf("%d%d", &n, &m)) {
        for (int i = 0; i < n; i++) {
            scanf("%s", map[i]);
            for (int j = 0; j < m; j++) {
                if (map[i][j] == 'a') {
                    sx = i, sy = j;
                }
            }
        }//for(i)
        int res = BFS();
        if (res == -1) {
            puts("Poor ANGEL has to stay in the prison all his life.");
        } else {
            printf("%d\n", res);
        }
    }
    return 0;
}
{% endhighlight %}

7） HDU     1548     [A strange lift](http://acm.hdu.edu.cn/showproblem.php?pid=1548)

{% highlight c %}
/*搜索做，BFS，向上，向下搜索，标记走过的点
如果找到楼层跳出，否则输出-1*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
using namespace std;
 
const int maxn = 201;
int n, src, des;
int kp[maxn];
bool mark[maxn];
struct Node {
    int floor;
    int tme;
};
 
int BFS() {
    memset(mark, false, sizeof(mark));
    queue<Node> Q;
    Node first, next_up, next_down;
    first.tme = 0;
    first.floor = src;
    Q.push(first);
    mark[src] = true;
 
    while (!Q.empty()) {
        first = Q.front();
        Q.pop();
 
        if (first.floor == des) {
            return first.tme;
        }
        next_up.floor = first.floor + kp[first.floor];
        if (next_up.floor <= n && !mark[next_up.floor ]) {
            next_up.tme = first.tme + 1;
            mark[next_up.floor] = true;
            Q.push(next_up);
        }
        next_down.floor = first.floor - kp[first.floor];
        if (next_down.floor >= 1 && !mark[next_down.floor]) {
            next_down.tme = first.tme + 1;
            mark[next_down.floor] = true;
            Q.push(next_down);
        }
    }
    return -1;
}  
 
int main() {
 
    while (~scanf("%d", &n), n ) {
        scanf("%d%d", &src, &des);
        for (int i = 1; i <= n; i++) {
            scanf("%d", &kp[i]);
        }
        int res = BFS();
        printf("%d\n", res);
    }
}
{% endhighlight %}

8） HDU     2531     [Catch him](http://acm.hdu.edu.cn/showproblem.php?pid=2531)(需要把图转化)

{% highlight c %}
/*HDU 2531  比较有难度的BFS
题目是整块移动，而不是像普通题的一个点移动
需要把图翻译一下：
如果一个点周围，和防守球员身体一样
大小的区域都能走，那么这点可走，否则不行
主要是把图翻译一遍 这个没遇到过*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
#include<climits>
using namespace std;
 
const int dir[4][2] = { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };
struct Node {
    int x, y;
    int step;
};
 
struct Def {
    Node node[25];
    int size;
}def;
 
inline bool cmp (const Node &a, const Node &b) {
    if (a.x == a.y) return a.y < b.y;
    return a.x < b.x;
}
 
const int maxn = 101;
 
int tpx, tpy;
int n, m;//行，列
int sx, sy, ex, ey;
char kp[maxn][maxn];
int map[maxn][maxn];//0->'.'， 1-'o', 2-> 'Q';
bool mark[maxn][maxn];
 
void Transfer(int x, int y) {
    int flag = 0;
    for (int i = 0; i < def.size; i++) {
        int tpx = x + def.node[i].x;
        int tpy = y + def.node[i].y;
        if (kp[tpx][tpy] == 'O') {
            flag = 1;
            break;
        } else if (kp[tpx][tpy] == 'Q') {
            flag = 2;
        }
    }
    map[x][y] = flag;
}
 
void Transfer_map() {
    sort(def.node, def.node + def.size, cmp);
    for (int i = 0; i < def.size; i++) {
        def.node[i].x -= tpx;
        def.node[i].y -= tpy;
    }
    tpx = 0, tpy = 0;
    for (int i = 0; i < def.size; i++) {
        if (def.node[i].x > tpx) tpx = def.node[i].x;
        if (def.node[i].y > tpy) tpy = def.node[i].y;
    }
    n = n - tpx;
    m = m - tpy;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            Transfer(i, j);
        }
    }
}//Transfer_map
 
inline bool Overmap(int x, int y) {
    if (x < 0 || x >= n || y < 0 || y >= m) {
        return true;
    } 
    return false;
}
 
int Bfs() {
    memset(mark, 0, sizeof(mark));
    mark[sx][sy] = true;
    queue<Node>Q;
    Node first, next;
    first.x = sx, first.y = sy, first.step = 0;
    Q.push(first);
    while(!Q.empty()) {
        first = Q.front();
        Q.pop();
        if (map[first.x][first.y] == 2) {
            return first.step;
        }
        for (int i = 0; i < 4; i++) {
            next.x = first.x + dir[i][0];
            next.y = first.y + dir[i][1];
            next.step = first.step + 1;
            if (!Overmap(next.x, next.y) 
            	&& map[next.x][next.y] != 1 && !mark[next.x][next.y]) {
                mark[next.x][next.y] = true;
                Q.push(next);
            }
        }//for
    }//while(empty)
    return -1;
}
 
 
int main() {
    while (~scanf("%d %d", &n, &m), n || m) {
        def.size = 0;
        tpx = tpy = INT_MAX;
        for (int i= 0; i < n; i++) {
            scanf("%s", kp[i]);
            for (int j = 0; j < m; j++) {
                if (kp[i][j] == 'D') {
                    def.node[def.size].x = i;
                    def.node[def.size++].y = j;
                    if (i < tpx) tpx = i;
                    if (j < tpy) tpy = j;
                }
            }
        }//for(i)
        sx = tpx, sy = tpy;
        Transfer_map();
 
        int res = Bfs();
        if (res == -1) {
            puts("Impossible");
        } else {
            printf("%d\n", res);
        }
    }
    return 0;
}
{% endhighlight %}

9） HDU     1044    [Collect More Jewels](http://acm.hdu.edu.cn/showproblem.php?pid=1044)

{% highlight c %}
/* 规定时间内拿到最大价值的东西，
先广搜，记录有宝物两点的距离（时间）
然后DFS枚举，任意两点之间的时间，找到最小值）
dfs还是不大熟啊*/
#include<iostream>
#include<cstdio>
#include<cstdlib>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
#include<stack>
#define lson (idx << 1) 
#define rson (lson  1)
#define ll long long
using namespace std;
 
const double PI = acos(-1.0);
const double eps = 1e-7;
int dir[4][2] = { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };//up, down, left, right
 
 
const int maxn = 51;
bool mark[maxn][maxn], rink[maxn];
char map[maxn][maxn];
int dis[13][13], cost[13];
int limit, n, m, kind, sum, res;
 
struct Node {
    int x, y, tme;
};
 
int index(char ch) {
    if (ch == '@') {
        return 0;
    } else if (ch == '<') {
        return kind + 1;
    } else {
        return ch - 'A' + 1;
    }
}
 
bool Overmap(int x, int y) {
    if (x < 0 || x >= n || y < 0 || y >= m) {
        return true;
    } 
    return false;
}
 
void Bfs(int sx, int sy) {
    queue<Node> Q;
    while (!Q.empty()) {
        Q.pop();
    }
    Node first, next;
    first.x = sx, first.y = sy, first.tme = 0;
    int inx = index(map[sx][sy]);
    mark[sx][sy] = true;
    Q.push(first);
     
    while (!Q.empty()) {
        first = Q.front();
        Q.pop();
        for (int i = 0; i < 4; i++) {
            next.x = first.x + dir[i][0];
            next.y = first.y + dir[i][1];
             
            if(!Overmap(next.x, next.y) && 
            	!mark[next.x][next.y] && map[next.x][next.y] != '*') {
                if (map[next.x][next.y] != '.') {
                    int iny = index(map[next.x][next.y]);
                    dis[inx][iny] = first.tme + 1;
                    //printf("inx = %d, iny = %d dis = %d\n", inx, iny, dis[inx][iny]);
                }
                mark[next.x][next.y] = true;
                next.tme = first.tme + 1;
                Q.push(next);
            }//符合条件啊 
        }
    }//while
}
 
void print() {
    puts("____________________________________");
    for (int i = 0; i <= kind + 1; i++) {
        for (int j = 0; j <= kind + 1; j++) {
            printf("%d ", dis[i][j]);
        }
        puts("");
    }
}
 
void Dfs(int s, int tme, int key) {
    if (res == sum) return ;
    if (key > res) res = key;
    for (int i = 1; i <= kind; i++) {
        if (rink[i]) continue;
        if (tme + dis[s][i] + dis[i][kind + 1] > limit) continue;
        rink[i] = true;
        Dfs(i, tme + dis[s][i], key + cost[i]);
        rink[i] = false;
    }
}
 
int main() {
    int cas;
    int cases = 1;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d %d %d %d", &m, &n, &limit, &kind);
        sum = 0;
        for (int i = 1; i <= kind; i++) {
            scanf("%d", &cost[i]);
            sum += cost[i];
        }
        for (int i = 0; i < n; i++) {
            scanf("%s", map[i]);
        }
         
        memset(dis, 0x7f, sizeof(dis));
         
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if(map[i][j] != '.' && map[i][j] != '*')  {
                    memset(mark, false, sizeof(mark));
                    Bfs(i, j);
                }
            }
        }//for(i)
 
        if (cases != 1) {
            puts("");
        }
        printf("Case %d:\n", cases++);
        if (dis[0][kind + 1] > limit) {
            puts("Impossible");
            continue;
        }
        res = 0;
        memset(rink, false, sizeof(rink));
        Dfs(0, 0, 0);
        printf("The best score is %d.\n", res);
    }
    return 0;
}
{% endhighlight %}

### 二、深度优先搜索（DFS）

深度优先搜索所遵循的搜索策略是尽可能“深”地搜索一个图，对于新发现的顶点，如果它还有以此为起点而未探测到的边，就沿此边继续探测下去。当顶点v的所有边都已被探寻过后，搜索将回溯到发现顶点v有起始点的那些边。这一过程一直进行到已发现从源顶点可达的所有顶点时为止。如果还存在未被发现的顶点，则选择其中一个作为源顶点，并重复以上过程。整个过程反复进行，知道所有的顶点都被已发现为止。

伪代码：

	DFS(G)
	1  for each vertex u ∈ V [G]
	2       do color[u] ← WHITE
	3          π[u] ← NIL
	4  time ← 0
	5  for each vertex u ∈ V [G]
	6       do if color[u] = WHITE
	7             then DFS-VISIT(u)
	 

	DFS-VISIT(u)
	1  color[u] ← GRAY     ▹White vertex u has just been discovered.
	2  time ← time +1
	3  d[u] time
	4  for each v ∈ Adj[u]  ▹Explore edge(u, v).
	5       do if color[v] = WHITE
	6             then π[v] ← u
	7                         DFS-VISIT(v)
	8  color[u] BLACK      ▹ Blacken u; it is finished.
	9  f [u] ▹ time ← time +1

图示过程，当各条边被算法探索到时，他们或者被加以阴影（如果它们是树边），或者被标以虚线（否则的话）。对于非树边，根据他们是否反向、交叉或前向边等情况，将它们标记为B/C/F。对于各个顶点，根据它们的发现/完成时间，给它们加上时间戳。
![Image][3]
再来几个DFS的题目：

1） HDU     1016    [Prime Ring Problem](http://acm.hdu.edu.cn/showproblem.php?pid=1016)

{% highlight c %}
/*
貌似是最简单的DFS
核心是DFS：
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
bool prime[41] = {0, 
0, 1, 1, 0, 1, 0, 1, 0, 0, 0,
1, 0, 1, 0, 0, 0, 1, 0, 1, 0,
0, 0, 1, 0, 0, 0, 0, 0, 1, 0,
1, 0, 0, 0, 0, 0, 1, 0, 0, 0,};//0~40
int num[21], mark[21];
int n;
 
void Dfs(int t) {
    if (t == n && prime[num[0] + num[n - 1]]) {//如果到最后，输出结果
        printf("%d", num[0]);
        for (int i = 1; i < n; i++) {
            printf(" %d", num[i]);
        }
        puts("");
    } else {
        for (int i = 2; i <= n; i++) {//每次都从头到尾搜一遍有没有符合要求的
            if (!mark[i] && prime[i + num[t - 1]]) {//如果条件成立
                mark[i] = 1;//标记
                num[t] = i;
                Dfs(t + 1);
                mark[i] = 0;//递归返回是撤销标记
            }
        }
    }//else
}//dfs
 
int main() {
    int cases = 1;
    num[0] = 1;
    while (~scanf("%d", &n)) {
        memset(mark, 0, sizeof(mark));
        printf("Case %d:\n", cases++);
        Dfs(1);
        puts("");
    }
    return 0;
}
{% endhighlight %}
  
2） HDU     1035     [Robot Motion](http://acm.hdu.edu.cn/showproblem.php?pid=1035)

{% highlight c %}
/*简单搜索题，看注释：
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
char map[100][100];
int num[100][100];
int n, m, k, lop, t;
 
bool Overmap(int x, int y) {
    if (x < 1 || x > n || y < 1 || y > m) {
        return true;
    } else {
        return false;
    }
}
 
void Dfs(int x, int y) {
    if (Overmap(x, y) || lop > 0) {
           if (lop > 0) {
            printf("%d step(s) before a loop of %d step(s)\n", t - lop, lop);
        } else {
            printf("%d step(s) to exit\n", t);
        }
        return ;
    } 
    if (num[x][y] == 0) {
        num[x][y] = ++t;
    } else {//判断是否有循环
        lop = t - num[x][y] + 1;
    }//else
    
    switch(map[x][y]) {
        case 'N': x--; Dfs(x, y); x++; break; //Dfs最主要：前加的条件，在之后要返回
        case 'E': y++; Dfs(x, y); y--; break;
        case 'S': x++; Dfs(x, y); x--; break;
        case 'W': y--; Dfs(x, y); y++; break;
    }//switch
}//dfs
         
 
int main() {
    while (~scanf("%d%d%d", &n, &m, &k), n || m || k) {
        memset(map, 0, sizeof(map));
        memset(num, 0, sizeof(num));
         
        lop = 0;
        t = 0;
        getchar();
        for (int i = 1; i <= n; i++) {
            scanf("%s", map[i] + 1);
            getchar();
        }
        Dfs(1, k);
 
    }
}
{% endhighlight %}

3） HDU     1258    [Sum It Up](http://acm.hdu.edu.cn/showproblem.php?pid=1258)

{% highlight c %}
/*
Dfs参数也是可以两个的，把和也弄到参数里
刚开始时弄个总的cnt每次判断是否和t相等，没这样好
难点：
这题要判重，重复的式子只出现一次
可以从起点上排查，每次Dfs起点都不一样就可以了
另外，有人说可以用状态压缩，还没学过
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
int t, n, num[13], cnt;
int res[13], index;
bool flag;
 
void Dfs(int x, int cnt) {
    if (cnt > t) {
        return ;
    } else if (cnt == t) {
        flag = true;
        printf("%d", res[0]);
        for (int i = 1; i < index; i++) {
            printf("+%d", res[i]);
        }
        puts("");
        return ;
    }
    int tmp = -1;
    for (int i = x + 1; i <= n; i++) {
        if (num[i] != tmp) {//每次搜索的起点不能一样，因为这样会出项重复 
            res[index++] = num[i];
            tmp = num[i];
            Dfs(i, cnt + num[i]);
            index--;
        }
    }
}//Dfs
     
     
int main() {
    while (~scanf("%d%d", &t, &n) != EOF, t || n) {
        for (int i = 1; i <= n; i++) {
            scanf("%d", &num[i]);
        }
        index = 0, flag = false;
        printf("Sums of %d:\n", t);
         
        Dfs(0, 0);
         
        if (!flag) {
            puts("NONE");
        }
    }
    return 0;
}
{% endhighlight %}

4） HDU     1501     [Zipper](http://acm.hdu.edu.cn/showproblem.php?pid=1501)
{% highlight c %}
/*
代码：
此题没有在Dfs内部加FOR循环是因为，答案唯一，而且顺序是挨着的
for (int x; x < n; x++)
这个是用来如果有多个结果时用，结果唯一时，多传几个参数
注意：
Dfs中，比如Dfs(x1 + 1, x2, x + 1);
不能写成Dfs（x1++, x2, x++)
明显错的
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
#define N 205
bool mark[N][N];
char s1[N], s2[N], s[2 * N];
int cas, len1, len2, len;
bool flag;
 
void Dfs(int x1, int x2, int x) {
    if (x >= len) { flag = 1; return ;}
    if (mark[x1][x2]) return ;
    else mark[x1][x2] = 1;
    if (x1 > len1 || x2 > len2) return ;
    if (s1[x1] != s[x] && s2[x2] != s[x]) return ;
    if (x1 < len1 && s1[x1] == s[x] && flag == 0) Dfs(x1 + 1, x2, x + 1);
    if (x2 < len2 && s2[x2] == s[x] && flag == 0) Dfs(x1, x2 + 1, x + 1);
}//Dfs
                 
             
int main() {
    scanf("%d", &cas);
    for (int i = 1; i <= cas; i++) {
        scanf("%s%s%s", s1,s2, s);
        len = strlen(s);
        len1 = strlen(s1);
        len2 = strlen(s2);
        memset(mark, 0, sizeof(mark));
        flag = 0;
        printf("Data set %d: ", i);
        Dfs(0, 0, 0);
        if (flag) {
            puts("yes");
        } else {
            puts("no");
        } 
    }
}
{% endhighlight %}

5） HDU     2660    [Accepted Necklace](http://acm.hdu.edu.cn/showproblem.php?pid=2660)

{% highlight c %}
/*深搜题，个数和价值是固定的，搜索时只要发现数量或重量还没到要求就继续搜索
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
int cas, n, k, sum;
int value[22], weight[22];
int W, maxn;
bool mark[22];
 
//p:下标，num：数量, val: 价值，wei：重量 
void Dfs(int p, int num, int val, int wei) {
    if (num == k || wei == W) {//重量数量均满足
        maxn = max(maxn, val);
    }
    for (int i = p; i <= n; i++) {
        if (!mark[i]) {
            if ((num + 1 <= k) && (wei + weight[i] <= W)) {
                mark[i] = 1;
                Dfs(i + 1, num + 1, val + value[i], wei + weight[i]);
                mark[i] = 0;
            }
        }//if
    }//for
}//Dfs
                 
             
int main() {
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d", &n, &k);
        for (int i = 0; i < n; i++) {
            scanf("%d%d", &value[i], &weight[i]);
        }
        scanf("%d", &W);
        maxn = 0;
        Dfs(0, 0, 0, 0);
        printf("%d\n", maxn);
    }
    return 0;
}
{% endhighlight %}

6） HDU     1181    [变形课](http://acm.hdu.edu.cn/showproblem.php?pid=1181)
{% highlight c %}
/*纯Dfs题，稍微有点技巧是，设一个结构体来存放每个单词的首字母和尾字母，
这样操作起来比较简单
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
struct {
    char beg;
    char end;
}ss[101];
 
bool mark[101];
bool flag;
int k ;
 
void Dfs(char ch) {
    if (ch == 'm') {
        flag = true;
        return ;
    }
    for (int i = 0; i < k; i++) {
        if (!mark[i] && ss[i].beg == ch) {
            mark[i] = 1;
            //cout << str[i] << endl;
            Dfs(ss[i].end);
            mark[i] = 0;
        }
    }//for
    return ;
}
             
int main() {
    string str;
     
    while (cin >> str) {
        k = 0;
        while (str != "0") {
            ss[k].beg = str[0];
            ss[k].end = str[str.size() - 1];
            k++;
            cin >> str;
        }
        flag = false;
        memset(mark, 0, sizeof(mark));
        Dfs('b');
        if (flag) {
            puts("Yes.");
        } else {
            puts("No.");
        }
    }
}
{% endhighlight %}

7） HDU     2181     [哈密顿绕行世界问题](http://acm.hdu.edu.cn/showproblem.php?pid=2181)

{% highlight c %}
/*简单深搜题*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
int num[22][3];
int res[22];
bool mark[22];
int m, k, cases, index;
 
void Dfs(int x, int index) {
    if ((num[x][0] == m || num[x][1] == m || num[x][2] == m) 
        && index == 20) {//最后一个是5，不能在下面判断，因为一开始已经mark过了
        printf("%d: ", cases++);
        for (int i = 0; i < 20; i++) {
            printf(" %d", res[i]);
        }
        printf(" %d\n", res[0]);
        return ;
    }
    for (int i = 0; i <= 2; i++) {
        int tmp = num[x][i];
        if (!mark[tmp]) {
            res[index] = tmp;;
            mark[tmp] = 1;
            Dfs(tmp, index + 1);
            mark[tmp] = 0;
        }
    }//for
    return ;
}
         
 
int main() {
    for (int i =1; i <= 20; i++) {
        scanf("%d%d%d", &num[i][0], &num[i][1], &num[i][2]);
    }
    cases = 1;
    while (scanf("%d", &m), m) {
        memset(mark, 0, sizeof(mark));
        res[0] = m;
        mark[m] = 1;
        Dfs(m, 1);
    }
    return 0;
}
{% endhighlight %}

8） HDU     2553     [N皇后问题](http://acm.hdu.edu.cn/showproblem.php?pid=2553)

{% highlight c %}
/*直接利用Dfs枚举每个点，前提是要预处理，不然会超时*/
#include<iostream>
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
struct point {
    int x, y;
}a[11];
int num, cnt, ans[11];
 
bool Judge(int n, int x, int y) {
    for (int i = 0; i < n; i++) {
        if (a[i].y == y || abs(a[i].x - x) == abs(a[i].y - y)) {
            return 0;
        }
    }
    return 1;
}
 
void Dfs(int n, int x) {//n是数量，r是x轴 
    if (n == num) {
        cnt++;
        return ;
    }
    int y;
    for (y = 1; y <= num; y++) {
        if (Judge(n, x, y)) {
            a[n].x = x;
            a[n].y = y;
            Dfs(n + 1, x + 1);
        }
    }
    return ;
}
 
 
int main() {
    int ans[11];
    for (num = 1; num < 11; num++) {//总共10个答案，每个都搜一遍
        cnt = 0;
        Dfs(0, 1);
        ans[num] = cnt;
    }
    while (~scanf("%d", &num), num) {
        printf("%d\n", ans[num]);
    }
    return 0;
}
{% endhighlight %}

9） HDU     1584     [蜘蛛牌](http://acm.hdu.edu.cn/showproblem.php?pid=1584)

{% highlight c %}
/*模拟蜘蛛纸牌,找出最小移动次数*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
#include<stack>
#include<climits>
using namespace std;
 
int kp[11];
int mark[11];
int res;
//num,已经移动了几张牌，sum移动耗费的步数 
 
void Dfs(int num, int sum) {
    if (sum >= res) return ;//剪枝 ,还没到9就已经大于原来的步数，直接回溯 
    if (num == 9) {//能到9，本身说明sum < res 
        res = sum;
        return ;
    }
     
    for (int i = 1; i < 10; i++) {
        if (!mark[i]) {
            mark[i] = true;
            for (int j = i + 1; j <= 10; j++) {
                if (!mark[j]) {
                    /*比如要移1了，如果2,3,4,5都已经被移动过了 
                    那么这几张牌必定叠放在6的下面，所以要移到6的位置
                    */
                    Dfs(num + 1, sum + abs(kp[i] - kp[j]));
                    break;
                    /*加入，此处是2放倒6上，如果在此处回溯，意思是 
                    枚举2放6上，2放7上，2放8上，事实上，2只能放6上
                    跳出，然后代表2已经找到对应的排列即，65432 
                    */
                }
            }
            mark[i] = false;
        }
    }
}
 
     
int main() {
    int n, cas;
 
    scanf("%d", &cas);
    while (cas--) {
        int a;
        for (int i = 1; i <= 10; i++) {    
            scanf("%d", &a);
            kp[a] = i;
        }
        memset(mark, 0, sizeof(mark));
        res = INT_MAX;
        Dfs(0, 0);
        printf("%d\n", res);
    }
    return 0;
}
{% endhighlight %}

10） HDU     1455     [Sticks](http://acm.hdu.edu.cn/showproblem.php?pid=1455)

{% highlight c %}
/*DFS好题，给出一些长度，找出最小的长度
让它可以平分 */
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
#include<stack>
#include<climits>
using namespace std;
 
int kp[64];
bool mark[64];
int n, maxn;
int sum, len, parts;
bool flag;
bool cmp(const int &a, const int &b) {
    return a > b;
}
 
void Dfs(int pos, int cur, int cnt) {//pos->下标，cur->每根长度, total 棍子数
    if (flag) return ;//已经找到解
    if (cur == len) { //满足长度的组合
        cnt++;
        if (cnt == parts) flag = true;
        Dfs(0, 0, cnt);//从剩下的棍子中继续选取相加长度为len的组合
        return ;
    }
    if (pos == n) return ;
    int pre = -1;
    for (int i = pos; i < n; i++) {
        if (!mark[i] && kp[i] != pre && cur + kp[i] <= len) {
            //两个连续的值相同或者相加长度超出len则剪枝
            pre = kp[i];
            mark[i] = true;
            Dfs(i + 1, cur + kp[i], cnt);
            mark[i] = false;
            if (flag || pos == 0) 
                //如果已经有解或者选取第一个尚未被选取的元素时取不到len的长度，则剪枝
                return ;
        }
    }
}
 
int main() {
    while (~scanf("%d", &n), n) {
        sum = 0, maxn = -INT_MAX;
        for (int i = 0; i < n; i++) {
            scanf("%d", &kp[i]);
            if (kp[i] > maxn) maxn = kp[i];
            sum += kp[i];
        }
        sort(kp, kp + n, cmp);
 
        for (len = maxn; len < sum; len++) {
            if (sum % len == 0) {
                parts = sum / len;
                memset(mark, 0, sizeof(mark));
                flag = 0;
                Dfs(0, 0, 0);
                if (flag) break;        
            }
        }
        printf("%d\n", len);
    }
    return 0;
}
{% endhighlight %}

11） HDU     3290     [The magic apple tree](http://acm.hdu.edu.cn/showproblem.php?pid=3290)

{% highlight c %}
/*
建好图，还要排序，貌似vector很方便
根据题意模拟Dfs
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<vector>
using namespace std;
 
const int maxn = 20001;
 
 
int kp[maxn], in[maxn], out[maxn];
vector<int> gra[maxn];
 
inline bool cmp(const int x, const int y) {
    return kp[x] < kp[y];
}
 
void Dfs(int i) {
    int r = gra[i].size();
    if (r == 0) return ;
 
    for (int j = 0; j < r; j++) {
        int u = gra[i][j];
        if (kp[u] == 0) {
            Dfs(u);
        }
    }
    sort(gra[i].begin(), gra[i].end(), cmp);
 
    r = ((r + 1) >> 1) - 1;
    kp[i] = kp[gra[i][r]];
}
 
int main() {
    int n;
    while (~scanf("%d", &n)) {
        int k, p, s; 
        for (int i = 1; i <= n; i++) {
            gra[i].clear();
            in[i] = out[i] = kp[i] = 0;
        }
 
        for (int i = 1; i <= n; i++) {
            scanf("%d", &p);
            for (int j = 1; j <= p; j++) {
                scanf("%d", &s);
                gra[i].push_back(s);
                in[s]++;
                out[i]++;
            }
        }
 
        for (int i = 1; i <= n; i++) {
            if (out[i] == 0)  kp[i] = i;
            else kp[i] = 0;
        }
 
        int tmp;
        for (int i = 1; i <= n; i++) {
            if (in[i] == 0) {
                Dfs(i);
                tmp = i;
                break;
            }
        }
        printf("%d\n", kp[tmp]);
    }
    return 0;
}
{% endhighlight %}

### 三、BFS判断二分图

判断二分图，使用的是黑白染色法，这个在之前《[二分匹配小结](/2011/07/14/binary-matching/)》里运用过，这里单独拿出来。黑白染色法简单的说就是在BFS的过程中给每个点染色，当发现相邻两个点颜色相同时，这个图就不是二分图，如果BFS结束，仍未跳出，则是二分图。

判断二分图代码：

1）邻接矩阵表示：

{% highlight c %}
bool Bfs() {//判断是否是二分图 
    memset(col, -1, sizeof(col));
    col[1] = 1;
    queue<int> Q;
    Q.push(1);
    while (!Q.empty()) {
        int first = Q.front();
        Q.pop();
        for (int i = 1; i <= n; i++) {
            if (map[first][i]) {
                if (col[i] == -1) {
                    Q.push(i);
                    col[i] = col[first] ^ 1;//染色 
                } else if (col[i] == col[first]) {//相邻如果同色，则不是二分图 
                    return false;
                }
            }
        }
    }
    return true;
}
{% endhighlight %}

2）邻接表表示：

{% highlight c %}
bool Bfs() {
    memset(col, -1, sizeof(col));
    col[1] = 1;
    queue<int>Q;
    Q.push(1);
    while (!Q.empty()) {
        int first = Q.front();
        Q.pop();
        for (int i = G.head[first]; i != -1; i = G.E[i].next) {
            int pos = G.E[i].v;
            if (col[pos] == -1) {
                Q.push(pos);
                col[pos] = col[first] ^ 1;
            }
            if (col[pos] == col[first]) {
                return false;
            }
        }
    }//while
    return true;
}
{% endhighlight %}
应用到题目中，可以看看下边两道题目：

HDU     2444     [The Accomodation of Students](http://acm.hdu.edu.cn/showproblem.php?pid=2444)

{% highlight c %}
/*
邻接表作法：
题目意思是：先把人分成两组，组1和组2， 
组1内的人互不认识，组2内的人互不认识
然后两个组内的人进行最大匹配，问最大匹配数是多少
sample2:
6 5
1 2
1 3
1 4
2 5
3 6
process:
先分成两组：(1, 5, 6), (2, 3, 4)
（这个可以通过黑白染色法,判断是不是二分图）
然后匹配（1,4）（2,5）（3,6）最大匹配数是3
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
using namespace std;
 
const int NV = 205;
const int NE = 10000;
int col[NE];
  
struct MaxMatch {
    int n, size;
    int head[NV];
    int aug[NV];
    bool mark[NV];
     
    struct Edge {
        int v, w, next;
        Edge () {}
        Edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) {}
    }E[NE];
 
   void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    int Hungary(int n) {
        int match = 0;
        memset(aug, -1, sizeof(aug));
        for (int i = 1; i <= n; i++) {
            if (aug[i] == -1) {//剪枝
                memset(mark, false, sizeof(mark));
                match += Find(i);
            }
        }
        return match;
    }//Hungray
 
    bool Find(int u) {
        for (int i = head[u]; i != -1; i = E[i].next) {
            int v = E[i].v;
            if (mark[v]) continue;
            mark[v] = true;
            if (aug[v] == -1 || Find(aug[v])) {
                aug[v] = u, aug[u] = v;
                return true;
            }
        }
        return false;
    }//Find
}G;
 
 
bool Bfs() {
    memset(col, -1, sizeof(col));
    col[1] = 1;
    queue<int>Q;
    Q.push(1);
    while (!Q.empty()) {
        int first = Q.front();
        Q.pop();
        for (int i = G.head[first]; i != -1; i = G.E[i].next) {
            int pos = G.E[i].v;
            if (col[pos] == -1) {
                Q.push(pos);
                col[pos] = col[first] ^ 1;
            }
            if (col[pos] == col[first]) {
                return false;
            }
        }
    }//while
    return true;
}
             
int main() {
    int n, m;
    while (~scanf("%d%d", &n, &m)) {//n->vetex, m->edge
        G.init(n);
         
        for (int i = 0; i < m; i++) {
            int a, b;
            scanf("%d%d", &a, &b);
            G.insert(a, b);
        }
             
         if (!Bfs()) {
            puts("No");
            continue;
         }//不能分为两组
         printf("%d\n", G.Hungary(n));
        }
    return 0;
}
{% endhighlight %}

邻接矩阵做法：

{% highlight c %}
/*
邻接矩阵作法：
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
using namespace std;
 
const int NV = 205;
//const int NE = 4000;
 
bool map[NV][NV];
bool mark[NV];
int aug[NV];
int col[NV];
 
int n, m;
 
bool Bfs() {//判断是否是二分图 
    memset(col, -1, sizeof(col));
    col[1] = 1;
    queue<int> Q;
    Q.push(1);
    while (!Q.empty()) {
        int first = Q.front();
        Q.pop();
        for (int i = 1; i <= n; i++) {
            if (map[first][i]) {
                if (col[i] == -1) {
                    Q.push(i);
                    col[i] = col[first] ^ 1;//染色 
                } else if (col[i] == col[first]) {//相邻如果同色，则不是二分图 
                    return false;
                }
            }
        }
    }
    return true;
}
 
bool Find (int t) {
    for (int i = 1; i <= n; i++) {
        if (!mark[i] && map[t][i]) {
            mark[i] = true;
            if (aug[i] == -1 || Find(aug[i])) {
                aug[i] = t;
                return true;
            }
        }
    }//for(i)
    return false;
}
 
int Hungary(int n, int m) {
    int match = 0;
    memset(aug, -1, sizeof(aug));
    for (int i = 1; i <= n; i++) {
        memset(mark, false, sizeof(mark));
        match += Find(i);
    }
    return match;
}
 
int main() {
    while (~scanf("%d%d", &n, &m)) {
        memset(map, 0, sizeof(map));
        int a, b;
        for (int i = 0; i < m; i++) {
            scanf("%d%d", &a, &b);
            map[a][b] = 1;
            map[b][a] = 1;
        }
        if (!Bfs()) {
            puts("No");
            continue;
        }
        int cnt = 0;
        printf("%d\n", Hungary(n, n) / 2);
    }
    return 0;
}
{% endhighlight %}

HDU     3478     [Catch](http://acm.hdu.edu.cn/showproblem.php?pid=3478)

{% highlight c %}
/*如果图是连通的且不是二分图，那么YES
判断二分图，可以用黑白染色法，如果相邻两个点颜色相同
那么不是二分图
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
using namespace std;
 
const int NV = 100001;
const int NE = 500001;
int col[NV];
int cases = 1;
int n, m, s;
 
struct MaxMatch {
    int n, size;
    int head[NV];
    int aug[NV];
    bool mark[NV];
 
    struct Edge {
        int v, w, next;
        Edge () {}
        Edge (int V, int NEXT, int W) : v(V), next(NEXT), w(W) {}
    }E[NE];
 
    void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
}G;
 
 
void Bfs() {
    memset(col, -1, sizeof(col));
    col[s] = 1;
    queue<int>Q;
    Q.push(s);
    int res = 0;//判断连通图 
    bool flag = false;//判断是否是二分图 
    while (!Q.empty()) {
        int first = Q.front();
        Q.pop();
        res++;
        for (int i = G.head[first]; i != -1; i = G.E[i].next) {
            int pos = G.E[i].v;
            if (col[pos] == -1) {
                Q.push(pos);
                col[pos] = col[first] ^ 1;
            }
            if (col[pos] == col[first]) {
                flag = true; 
            }
        }
    }//while
    if (flag && res == n) {
        printf("Case %d: YES\n", cases++);
    } else {
        printf("Case %d: NO\n", cases++);
    }
}           
 
int main() {
    int cas;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d%d", &n, &m, &s);
        G.init(NV);
        for (int i = 0; i < m; i++) {
            int u, v;
            scanf("%d%d", &u, &v);
            G.insert(u, v);
            G.insert(v, u);
        }
        Bfs();
    }
    return 0;
}
{% endhighlight %}

### 四、拓扑排序（Topological Sort）

一个图的拓扑排序可以看成是图中所有顶点沿水平线排列而成的一个序列，使得所有的有向边均从左指向右。拓扑排序的作用是：有向无环图用于说明事件的发生的先后次序。

如图，左边是有向无环图，右边是拓扑排序后的同一图形。这个图模拟的是教授衣服的穿戴次序：
![Image][4]
拓扑排序伪代码：

	TOPOLOGICAL-SORT(G)
	1  call DFS(G) to compute finishing times f[v] for each vertex v
	2  as each vertex is finished, insert it onto the front of a linked list
	3  return the linked list of vertices

几道很裸的拓扑排序题目：

1）HDU     3342     [Legal or Not](http://acm.hdu.edu.cn/showproblem.php?pid=3342)

{% highlight c %}
#include<iostream>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
#include<climits>
using namespace std;
 
const int NV = 101;
const int NE = 101;
 
struct Topo {
    int n, size;
    int dis[NV], head[NV];
    bool in[NV];
    int indegree[NV];
 
    struct edge {
        int v, w, next;
        edge () {}
        edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) {}
    }E[NE];
 
    void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i < n; i++) {
            head[i] = -1;
            indegree[i] = 0;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = edge(v, head[u], w);
        head[u] = size++;
        indegree[v]++;
    }
 
    bool topo() {
        queue<int> Q;
        for (int i = 0; i < n; i++) {
            if (indegree[i] == 0) {
                Q.push(i);
                indegree[i]--;
            }
        }//for(i)
 
        int ans = 0;
        while (!Q.empty()) {
            int x = Q.front();
            Q.pop();
 
            for (int i = head[x]; i != -1; i = E[i].next) {
                indegree[ E[i].v ]--;
                if (indegree[ E[i].v ] == 0) {
                    Q.push (E[i].v);
                }
            }//for(i)
        }//Q
 
        for (int i = 0; i < n; i++) {
            if (indegree[i] > 0) {
                return false;
            }
        }
        return true;
    }
 
    void print() {
        for (int i = 0; i <= n; i++) {
            printf("%d: ", i);
            for (int j = head[i]; j != -1; j = E[j].next) {
                printf(" %d", E[j].v);
            }puts("");
        }
    }
}G;
 
int main() {
    int n, m;
    while (~scanf("%d%d", &n, &m), n) {
        G.init(n);
        int a, b;
        while (m--) {
            scanf("%d%d",&a,&b);
            G.insert(a, b);
        }
 
        if (G.topo()) {
            puts("YES");
        } else {
            puts("NO");
        }
    }
    return 0;
}    
{% endhighlight %}

2）HDU     2647     [Reward](http://acm.hdu.edu.cn/showproblem.php?pid=2647)    

{% highlight c %}
//拓扑排序，然后记录每个点之前 有几个前驱，记录这个值得总和
#include<iostream>
#include<cstdio>
#include<cstring>
#include<queue>
using namespace std;
 
 
const int NV = 10001;
const int NE = 20001;
 
struct Topo {
    int n, size, res;
    int dis[NV], head[NV], cnt[NV];//cnt记录的是i点的前驱个数
    bool in[NV];
    int indegree[NV];
 
    struct edge {
        int v, w, next;
        edge () {}
        edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) {}
    }E[NE];
 
    void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i < n; i++) {
            head[i] = -1;
            indegree[i] = 0;
            cnt[i] = 0;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = edge(v, head[u], w);
        head[u] = size++;
        indegree[v]++;
    }
 
    bool topo() {
        queue<int> Q;
        for (int i = 0; i < n; i++) {
            if (indegree[i] == 0) {
                Q.push(i);
                indegree[i]--;
            }
        }//for(i)
 
        res = 0;
        while (!Q.empty()) {
            int x = Q.front();
            res += cnt[x];
            Q.pop();
 
            for (int i = head[x]; i != -1; i = E[i].next) {
                indegree[ E[i].v ]--;
                if (indegree[ E[i].v ] == 0) {
                    Q.push (E[i].v);
                    cnt [E[i].v] = cnt[x] + 1;
                }
            }//for(i)
        }//Q
 
        for (int i = 0; i < n; i++) {
            if (indegree[i] > 0) {
                return false;
            }
        }
        return true;
    }
 
    void print() {
        for (int i = 0; i <= n; i++) {
            printf("%d: ", i);
            for (int j = head[i]; j != -1; j = E[j].next) {
                printf(" %d", E[j].v);
            }puts("");
        }
    }
}G;
 
int main() {
    int n, m;
    while (~scanf("%d%d", &n, &m)) {
        G.init(n);
        int a, b;
        while (m--) {
            scanf("%d%d",&a,&b);
            G.insert(b - 1, a - 1);
        }
 
        if (G.topo()) {
            printf("%d\n", G.res + 888 * n);
        } else {
            printf("-1\n");
        }
    }
    return 0;
}    
{% endhighlight %}

3）HDU     1285     [确定比赛名次](http://acm.hdu.edu.cn/showproblem.php?pid=1285)

{% highlight c %}
//拓扑排序 + 优先队列的排序规则
//要求输出拓扑排序，非常常用，里边优先队列排序的方法非常常用
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<queue>
using namespace std;
 
 
const int NV = 500;
const int NE = 1000;
 
struct Topo {
    int n, size, res;
    int dis[NV], head[NV], cnt[NV];//cnt记录的是i点的前驱个数
    bool in[NV];
    int indegree[NV];
 
    struct edge {
        int v, w, next;
        edge () {}
        edge (int V, int NEXT, int W = 0) : v(V), next(NEXT), w(W) {}
    }E[NE];
 
    void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i < n; i++) {
            head[i] = -1;
            indegree[i] = 0;
            cnt[i] = 0;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = edge(v, head[u], w);
        head[u] = size++;
        indegree[v]++;
    }
 
    vector<int> ans;
    struct cmp {
        bool operator() (const int &a, const int &b) {
            return a > b;
        }
    };
 
    bool topo() {
        priority_queue<int, vector<int>, cmp> Q;
        ans.clear();//不可少
        for (int i = 0; i < n; i++) {
            if (indegree[i] == 0) {
                Q.push(i);
                indegree[i]--;
            }
        }//for(i)
 
        res = 0;
        while (!Q.empty()) {
            int x = Q.top();
            ans.push_back(x);//每次出队列都送进ans
            res += cnt[x];
            Q.pop();
 
            for (int i = head[x]; i != -1; i = E[i].next) {
                indegree[ E[i].v ]--;
                if (indegree[ E[i].v ] == 0) {
                    Q.push (E[i].v);
                    cnt [E[i].v] = cnt[x] + 1;
                }
            }//for(i)
        }//Q
 
        for (int i = 0; i < n; i++) {
            if (indegree[i] > 0) {
                return false;
            }
        }
        return true;
    }
 
    void print() {
        for (int i = 0; i <= n; i++) {
            printf("%d: ", i);
            for (int j = head[i]; j != -1; j = E[j].next) {
                printf(" %d", E[j].v);
            }puts("");
        }
    }
}G;
 
int main() {
    int n, m;
    while (~scanf("%d%d", &n, &m)) {
        G.init(n);
        int a, b;
        while (m--) {
            scanf("%d%d",&a,&b);
            G.insert(a - 1, b - 1);
        }
 
        G.topo();
 
        int des = G.ans.size() - 1;
        for (int i = 0; i < des; i++) {
            printf("%d ", G.ans[i] + 1);
        }
        printf("%d\n", G.ans[des] + 1);
    }
    return 0;
}
{% endhighlight %}

### 五、强连通分支（Strongly connected components）

深度优先搜索的经典应用：把一个有向图分解为各强连通分支。有向图G = （V, E）的一个强连通分支就是一个最大顶点集合C ∈V，对于C中的每一对顶点u和v，有u->v和v->u；亦即，顶点u和v是互相可达的。
![Image][5]
在寻找 图G = （V, E）的强连通分支的算法中，用到了G的转置，其定义是GT = (V，ET)，ET = {(u, v) : (v, u) ∈ E}。亦即，由ET是由G中的边改变方向后所组成的，在给定图G的邻接表表示的情况下，建立GT所需要时间为O(V + E)。G和GT有着相同的强连通分支，亦即，在G中，u和v互为可达，当且仅当在GT中它们互为可达。

如图，a)图中的阴影区域就是G的各强连通子图。对每个顶点都标出了其发现时刻与完成时刻，阴影覆盖的边为树边。b)G的装置图GT。图中示出了伪代码第三行计算出的深度优先森林，其中阴影覆盖的边是树边。每个强连通子图对应于一棵深度优先树。图中涂上深阴影的顶点b、c、g和h是对GT进行深度优先搜索所产生的深度优先树的树根。c)把G的每个强连通子图缩减为每分支只有单个顶点所得到的无回路子图GSCC。

可以得出强连通分支的伪代码：

	STRONGLY-CONNECTED-COMPONENTS (G)
	1  call DFS (G) to compute finishing times f[u] for each vertex u
	2  compute GT
	3  call DFS (GT), but in the main loop of DFS, consider the vertices
	          in order of decreasing f[u] (as computed in line 1)
	4  output the vertices of each tree in the depth-first forest formed in line 3 as a
	          separate strongly connected component

常用的求强连通分量的算法有两个，一个是Kosaraju算法，这个算法是基于两次Dfs来实现的；还有一个就是Tarjan算法，这个算法完成一次Dfs就可以找到图中的强连通分支。Tarjan算法效率要高于Kosaraju算法。

关于讲解求有向图强连通分量的Tarjan算法，BYVoid的这篇文章绝对是网上最好的:《[有向图强连通分量的Tarjan算法](http://www.byvoid.com/blog/scc-tarjan/zh-hans/)》。关于Kosaraju和Tarjan这里就不多叙述了，Tarjan算法BYVoid的一篇文章足矣，Kosaraju算法可以边看下边实际应用代码理解，Kosaraju算法很好理解。

一些关于强连通的题目：

1） HDU     2767     [Proving Equivalences](http://acm.hdu.edu.cn/showproblem.php?pid=2767)

Kosaraju算法代码， 可以当模板：

{% highlight c %}
/*Kosaraju算法解法，加多少条边，可以让图变成强连通图*/
/*
2
4 0
3 2
1 2
1 3
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<vector>
using namespace std;
 
const int N = 20050;
 
vector<int> v1[N], v2[N];
int order[N], id[N];
int in[N], out[N];
bool mark[N];
int n, m, cnt, scc;
 
 
void DFS1 (int x) {//正向深搜一遍
    int size = v1[x].size();
    mark[x] = true;
    for (int i = 0; i < size; i++) {
        if (!mark[v1[x][i]]) {
            DFS1(v1[x][i]);
        }
    }
    order[cnt++] = x;//sample，储存的顺序是 1 2 5 3 4
}
 
void DFS2(int x) {//反向深搜一遍，目的是找到没有入度的点
    int size = v2[x].size();
    mark[x] = true;
    id[x] = scc;//id[1 …… 5] = {0 0 1 2 0}
    for (int i = 0; i < size; i++) {
        if (!mark[v2[x][i]]) {
            DFS2(v2[x][i]);
        }
    }
}
 
void Kosaraju() {
    cnt = 0;
    memset(mark, false, sizeof(mark));
    for (int i = 0; i < n; i++) {
        if (!mark[i]) {
            DFS1(i);
        }
    }
    memset(mark, false, sizeof(mark));
    scc = 0;
    for (int i = n - 1; i >= 0; i--) {
        if (mark[order[i]]) continue;
        DFS2(order[i]);
        scc++;
    }
}
 
int main() {
    int size, ans1, ans2;
    int cas;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d", &n, &m);
        if (m == 0) {
            printf("%d\n", n);
            continue;
        }
        for (int i = 0; i < n; i++) {
            v1[i].clear();
            v2[i].clear();
        }
        for (int i = 0; i < m; i++) {
            int a, b;
            scanf("%d%d", &a, &b);
            v1[a - 1].push_back(b - 1);
            v2[b - 1].push_back(a - 1);
        }
 
        Kosaraju();
 
        if (scc == 1) {
            puts("0");
            continue;
        }//只有一个强连通图，只要一个点就可以到达所有的点，且不用加边
 
        for (int i = 0; i < n; i++) {
            in[i] = out[i] = 0;
        }
 
        for (int i = 0; i < n; i++) {
            size = v1[i].size();
            for (int j = 0; j < size; j++) {
                if (id[i] == id[v1[i][j]]) continue;
                out[ id[i] ]++;//0强连通图出度+1
                in[ id[v1[i][j]] ]++;//另外一个强连通图入度+1
            }
        }
 
 
        ans1 = ans2 = 0;
        for (int i = 0; i < scc; i++) {
            if (in[i] == 0) ans1++;
            if (out[i] == 0) ans2++;
        }
        printf("%d\n", max(ans1, ans2));
    }
    return 0;
}
{% endhighlight %}

Tarjan算法代码， 可以当模板：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<stack>
using namespace std;
 
const int NV = 20005;
const int NE = 50005;
 
struct Tarjan {
    int dfn[NV];//深度优先搜索访问次序
    int low[NV];//能追溯到的最早的次序
    int index, scc, n;//index->索引号，scc->强连通分量个数
    bool instack[NV];//检查是否在栈中
    int root[NV];//记录每个点在第几号强连通分量里
 
    stack<int> S;
    struct Edge{
        int v , next, w;
        Edge () {}
        Edge (int V, int NEXT, int W) : v(V), next(NEXT), w(W){}
    }E[NE];
    int head[NV], size;
     
    inline void init (int nn) {
        n = nn, size = 0;
        memset(head, -1, sizeof(int) * (n + 1));
    }
     
    inline void insert (int u , int v, int w = 1) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
     
    void tarjan (int i) {
        dfn[i] = low[i] = ++index;
        instack[i] = true;
        S.push(i);
        for (int k = head[i]; k != -1; k = E[k].next) {
            int v = E[k].v;
            if (!dfn[v]) {
                tarjan(v);
                if(low[v] < low[i]) low[i] = low[v];
            } else if (instack[v] && dfn[v] < low[i])
                low[i] = dfn[v];
        }
        if (dfn[i] == low[i]) {
            scc++;
            int x;
            do{
                x = S.top();
                S.pop();
                instack[x] = false;
                root[x] = scc;
            }while (x != i);
        }
    }
     
    void Solve(){
        while (!S.empty()) S.pop();
        memset(dfn,0,sizeof(dfn));
        index = scc = 0;
        for (int i = 1; i <= n; i++) {
            if(!dfn[i]) tarjan(i);
        }
    }
 
    struct Relation {
        int u,v;
    }re[NE];
}G;
 
 
int main() {
    int n, m, in[NV], out[NV];
    int cas;
    scanf("%d",&cas);
    while(cas--) {
        scanf("%d%d", &n, &m);
        G.init(n);
        if (m == 0) {
            printf("%d\n", n);
            continue;
        }
        for (int i = 0; i < m; i++) {
            int a, b;
            scanf("%d%d",&a, &b);
            G.insert(a, b);
            G.re[i].u = a, G.re[i].v = b;
        }
         
        G.Solve();
        if (G.scc == 1) {
            puts("0");
            continue;
        }
        for (int i = 1; i <= G.scc; i++) {
            in[i] = out[i] = 0;
        }
        for (int i = 0; i < m; i++) {
            if (G.root[G.re[i].u] == G.root[G.re[i].v]) continue;
            in[ G.root[ G.re[i].u] ]++;
            out[ G.root[G.re[i].v] ]++;
        }
        int cnt_in = 0 , cnt_out = 0;
        for (int i = 1; i <= G.scc; i++ ) {
            if (in[i] == 0) cnt_in++;
            if (out[i] == 0) cnt_out++;
        }
        printf("%d\n", max(cnt_in,cnt_out));
    }
    return 0;
}
{% endhighlight %}

2） HDU     1827     [Summer Holiday](http://acm.hdu.edu.cn/showproblem.php?pid=1827)
{% highlight c %}
/*1、求出入度为0的点的个数，
即强连通分量的root个数
 2、求出每个强连通分量里花费最小的总和*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<stack>
#include<climits>
using namespace std;
const int NV = 1005;
const int NE = 2005;
 
int kp[1005];
struct Tarjan {
    int dfn[NV];//深度优先搜索访问次序
    int low[NV];//能追溯到的最早的次序
    int index, scc, n;//index->索引号，scc->强连通分量个数
    bool instack[NV];//检查是否在栈中
    int root[NV];//记录每个点在第几号强连通分量里
 
    stack<int> S;
    struct Edge{
        int v , next, w;
        Edge () {}
        Edge (int V, int NEXT, int W) : v(V), next(NEXT), w(W){}
    }E[NE];
    int head[NV], size;
     
    inline void init (int nn) {
        n = nn, size = 0;
        memset(head, -1, sizeof(int) * (n + 1));
    }
     
    inline void insert (int u , int v, int w = 1) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
     
    void tarjan (int i) {
        dfn[i] = low[i] = ++index;
        instack[i] = true;
        S.push(i);
        for (int k = head[i]; k != -1; k = E[k].next) {
            int v = E[k].v;
            if (!dfn[v]) {
                tarjan(v);
                if(low[v] < low[i]) low[i] = low[v];
            } else if (instack[v] && dfn[v] < low[i])
                low[i] = dfn[v];
        }
        if (dfn[i] == low[i]) {
            scc++;
            int x;
            do{
                x = S.top();
                S.pop();
                instack[x] = false;
                root[x] = scc;
            }while (x != i);
        }
    }
     
    void Solve(){
        while (!S.empty()) S.pop();
        memset(dfn,0,sizeof(dfn));
        index = scc = 0;
        for (int i = 1; i <= n; i++) {
            if(!dfn[i]) tarjan(i);
        }
    }
 
    struct Relation {
        int u,v;
    }re[NE];
     
    void print() {
        puts("_______________________");
        for (int i = 1; i <= n; i++) {
            printf("i = %d,  root[%d] = %d\n", i, i,  root[i]);
        }
    }
}G;
 
int inque[NV];
int minval[NV];
int main() {
    int n, m;
    while (~scanf("%d %d", &n, &m)) {
        for (int i = 1; i <= n; i++) {
            scanf("%d", &kp[i]);
        }
        G.init(n);
        for (int i = 1; i <= m; i++) {
            int u, v;
            scanf("%d %d", &u, &v);
            G.insert(u, v);
            G.re[i].u = u, G.re[i].v = v;
        }
        G.Solve();
    //    G.print();
        for (int i = 1; i <= G.scc; i++) {
            minval[i] = INT_MAX;
            inque[i] = 0;
        }
        for (int i = 1; i <= m; i++) {
            int v = G.root[G.re[i].v];
            int u = G.root[G.re[i].u];
            if (u == v) continue;
            inque[v]++;
        }
        int sum_cnt = 0;
        for (int i = 1; i <= G.scc; i++) {
            if (inque[i] == 0) {
                sum_cnt++;
            }
        }
        for (int i = 1; i <= n; i++) {
            minval[G.root[i]] = min(minval[G.root[i]], kp[i]);
        }
        int sum = 0;
        for (int i = 1; i <= G.scc; i++) {
            if (inque[i] == 0) {
                sum += minval[i];
            }
        }
        printf("%d %d\n", sum_cnt, sum);
    }
    return 0;
}
{% endhighlight %}

3） HDU     3072     [Intelligence System](http://acm.hdu.edu.cn/showproblem.php?pid=3072)

{% highlight c %}
/*求出每个强连通分量里花费最小的总和
最小树形图？*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<stack>
using namespace std;
#define Min(a, b) ((a) < (b) ? (a) : (b))
const int NV = 50005;
const int NE = 100005;
#define inf 0x7f7f7f7f
 
int kp[1005];
struct Tarjan {
    int dfn[NV];//深度优先搜索访问次序
    int low[NV];//能追溯到的最早的次序
    int index, scc, n;//index->索引号，scc->强连通分量个数
    bool instack[NV];//检查是否在栈中
    int root[NV];//记录每个点在第几号强连通分量里
 
    stack<int> S;
    struct Edge{
        int v , next, w;
        Edge () {}
        Edge (int V, int NEXT, int W) : v(V), next(NEXT), w(W){}
    }E[NE];
    int head[NV], size;
     
    inline void init (int nn) {
        n = nn, size = 0;
        memset(head, -1, sizeof(int) * (n + 1));
    }
     
    inline void insert (int u , int v, int w = 1) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
     
    void tarjan (int i) {
        dfn[i] = low[i] = ++index;
        instack[i] = true;
        S.push(i);
        for (int k = head[i]; k != -1; k = E[k].next) {
            int v = E[k].v;
            if (!dfn[v]) {
                tarjan(v);
                if(low[v] < low[i]) low[i] = low[v];
            } else if (instack[v] && dfn[v] < low[i])
                low[i] = dfn[v];
        }
        if (dfn[i] == low[i]) {
            scc++;
            int x;
            do{
                x = S.top();
                S.pop();
                instack[x] = false;
                root[x] = scc;
            }while (x != i);
        }
    }
     
    void Solve(){
        while (!S.empty()) S.pop();
        memset(dfn,0,sizeof(dfn));
        index = scc = 0;
        for (int i = 0; i < n; i++) {
            if(!dfn[i]) tarjan(i);
        }
    }
    struct Relation {
        int u,v, w;
    }re[NE];
     
    void print() {
        puts("_______________________");
        for (int i = 0; i < n; i++) {
            printf("i = %d,  root[%d] = %d\n", i, i,  root[i]);
        }
    }
}G;
 
int main() {
    int n, m;
    int kp[NV];
    while (~scanf("%d %d", &n, &m)) {
        G.init(n);
        for (int i = 0; i < m; i++) {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            G.insert(u, v, w);
            G.re[i].u = u, G.re[i].v = v, G.re[i].w = w;
        }
         
        G.Solve();
        memset(kp, 0x7f, sizeof(*kp) * (G.scc + 1));
        for (int i = 0; i < m; i++) {
            int v = G.root[G.re[i].v];
            int u = G.root[G.re[i].u];
            if (u != v) {
                kp[v] = Min(kp[v], G.re[i].w);
            }
        }
        int res = 0;
        for (int i = 1; i <= G.scc; i++) {
            if (kp[i] < inf) {
                res += kp[i];
            }
        }
        printf("%d\n", res);
    }
    return 0;
}
{% endhighlight %}

4） HDU     3639    [Hawk-and-Chicken](http://acm.hdu.edu.cn/showproblem.php?pid=3639)（缩点)

{% highlight c %}
/*缩点 + 输出出度为0的点 ，反向图即可*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<stack>
using namespace std;
const int NV = 5005;
const int NE = 30050;
 
struct Tarjan {
    int dfn[NV];//深度优先搜索访问次序
    int low[NV];//能追溯到的最早的次序
    int index, scc, n;//index->索引号，scc->强连通分量个数
    bool instack[NV];//检查是否在栈中
    int root[NV];//记录每个点在第几号强连通分量里
 
    stack<int> S;
    struct Edge{
        int v , next, w;
        Edge () {}
        Edge (int V, int NEXT, int W) : v(V), next(NEXT), w(W){}
    }E[NE];
    int head[NV], size;
     
    inline void init (int nn) {
        n = nn, size = 0;
        memset(head, -1, sizeof(int) * (n + 1));
    }
     
    inline void insert (int u , int v, int w = 1) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
     
    void printG() {
        puts("Graph______________________________");
        for (int i = 0; i <= n; i++) {
            printf("%d: ", i);
            for (int j = head[i]; j != -1; j = E[j].next) {
                printf(" %d", E[j].v);
            }puts("");
        }
    }
     
    void tarjan (int i) {
        dfn[i] = low[i] = ++index;
        instack[i] = true;
        S.push(i);
        for (int k = head[i]; k != -1; k = E[k].next) {
            int v = E[k].v;
            if (!dfn[v]) {
                tarjan(v);
                if(low[v] < low[i]) low[i] = low[v];
            } else if (instack[v] && dfn[v] < low[i])
                low[i] = dfn[v];
        }
        if (dfn[i] == low[i]) {
            scc++;
            int x;
            do{
                x = S.top();
                S.pop();
                instack[x] = false;
                root[x] = scc;
            }while (x != i);
        }
    }
     
    void Solve(){
        while (!S.empty()) S.pop();
        memset(dfn,0,sizeof(dfn));
        index = scc = 0;
        for (int i = 0; i < n; i++) {
            if(!dfn[i]) tarjan(i);
        }
    }
     
    bool isloop(int u, int v, int w) {
        for (int p = head[u]; p; p = E[p]. next) {
            if (E[p].v == v) {
                E[p].w = min(w, E[p].w);
                return true;
            }
        }
        return false;
    }
     
    void printS() {
        puts("Strong_______________________");
        for (int i = 0; i < n; i++) {
            printf("i = %d,  root[%d] = %d\n", i, i,  root[i]);
        }
    }
     
    struct Relation {
        int u,v, w;
    }re[NE];
}G;
 
int kp[NV], in[NV], val[NV];
bool mark[NV];
int sum;
 
void Dfs( int u ) {
    sum += val[u];
    mark[u] = true;
    for( int i = G.head[u] ; i != -1 ; i = G.E[i].next ) {
        int v = G.E[i].v;
        if( !mark[v] ) Dfs(v);
    }
}
 
int main() {
    int cas;
    scanf("%d", &cas);
    int cases = 1;
    while (cas--) {
        int n, m;
        scanf("%d %d", &n, &m);
        G.init(n);
        for (int i = 0; i < m; i++) {
            scanf("%d %d", &G.re[i].u, &G.re[i].v);
            G.insert(G.re[i].u, G.re[i].v);
        }
        G.Solve();
    //    G.printS();
         
        G.init(G.scc + 1);
         
        for (int i = 1; i <= G.scc; i++) {
            kp[i] = val[i] = in[i] = 0;
        }
        for (int i = 0; i < n; i++) {
            val[G.root[i]]++;
        }
         
        for (int i = 0; i < m; i++) {
            int u = G.re[i].u;
            int v = G.re[i].v;
            if (G.root[u] == G.root[v]) continue;
            G.insert(G.root[v], G.root[u], 1);
            in[G.root[u]]++;
        }
    //    G.printG();
        int maxx = 0;
        for (int i = 1; i <= G.scc; i++) {
            if (!in[i]) {
                memset(mark, false, sizeof(mark));
                sum = 0;
                Dfs(i);
                sum--;
                kp[i] = sum;
                maxx = max(maxx, sum);
            }
        }
        printf("Case %d: %d\n",cases++, maxx);
        bool flag = true;
        for( int i = 0 ; i < n ; i++ ) {
            if( kp[ G.root[i] ] == maxx ) {
                if( !flag ) printf(" ");
                else flag = false;
                printf("%d",i);
            }
        }
        puts("");
    }
    return 0; 
}
{% endhighlight %}

[1]: /uploads/2011/10/图表示方法.png
[2]: /uploads/2011/10/BFS.png
[3]: /uploads/2011/10/DFS.png
[4]: /uploads/2011/10/拓扑排序.png
[5]: /uploads/2011/10/强连通分支.png

