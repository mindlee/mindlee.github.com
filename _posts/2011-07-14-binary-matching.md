---
title: 二分匹配小结
author: Wei Li
excerpt: 匈牙利算法关键，是找增广路，每次找到增广路，匹配数+1（所谓的取反），直到找不到增广路，此时匹配最大。包含十几道二分题目的代码。
layout: post
permalink: /2011/07/14/binary-matching/
views:
  - 4219
categories:
  - 算法学习
tags:
  - ACM
  - 二分匹配
  - 算法
---
二分匹配所用算法：匈牙利算法。关于匈牙利算法这位牛讲的很不错：http://imlazy.ycool.com/post.1603708.html

匈牙利算法关键，是找增广路，每次找到增广路，匹配数+1（所谓的取反），直到找不到增广路，此时匹配最大

匈牙利算法三个常见类型：

1、DAG图（有向无环图）的最小路径覆盖，
最小路径覆盖数 = 顶点数 – 最大匹配数
解释：一条路径覆盖两个点，意思是最大匹配可以覆盖->(2 * 最大匹配)个点
一个匹配，一条路径，剩下的点都需要单独一条边覆盖
所以: 最大匹配数 + (n – 2 * 最大匹配数）= 最小路径覆盖

2、同理，也可推出：
二分图的最大独立点集 = 顶点数 – 最大匹配数

3、最小顶点覆盖 = 最大匹配数
最小顶点覆盖：假如选了一个点就相当于覆盖了以它为端点的所有边，你需要选择最少的点来覆盖所有的边。
我的理解：每条边代表一个关系，最小顶点覆盖就是，用最少得关键点（看题目设置）来满足全部的关系，典型例题：HDU 3360
Matrix67大神的讲解：http://www.matrix67.com/blog/archives/116

我的二分匹配模板：（模板这题应该是HDU2063吧）
邻接矩阵建图模板：
{% highlight c %}
/*
模板，map从0开始or从1开始，Find和Hungary里都
要修改，另外，有些题用矩阵方便
比如HDU 1281, 需要删边。
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 505;
 
int map[maxn][maxn];
bool mark[maxn];//是否访问过 
int aug[maxn];//增广路径
 
int n, m;
  
inline bool Find (int t) {
    for (int i = 0; i < m; i++) {
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
 
inline int Hungary(int n, int m) {
    int match = 0;
    memset(aug, -1, sizeof(aug));
    for (int i = 0; i < n; i++) {
        memset(mark, false, sizeof(mark));
        match += Find(i);
    }
    return match;
}
 
int main() {
    int k, a, b;
    while (~scanf("%d", &k), k) {
        memset(map, 0, sizeof(map));//矩阵清零 
        scanf("%d%d", &n, &m);
        while(k--) {
            scanf("%d%d",&a,&b);
            map[a - 1][b - 1] = 1;
        }
        int res = Hungary(n, m);
        printf("%d\n", res);
    }
    return 0;
}
{% endhighlight %}

邻接表建图模板：
{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int NV = 1005;
const int NE = 3000;
 
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
 
    inline void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline int Hungary(int n) {
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
 
    inline bool Find(int u) {
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
 
int main() {
       int k, m, n, a, b;
       while (~scanf("%d", &k), k) {
           scanf("%d%d", &m, &n);
           G.init(m + n);
           while (k--) {
               scanf("%d%d", &a, &b);
               G.insert(a, b + m); 
         }//while(k--)
         printf("%d\n", G.Hungary(m));
      }//while
   return 0;
}
{% endhighlight %}

题目：

HDU 2063 过山车 裸的二分匹配（代码见模板）

HDU 1150 Machine Schedule 最小顶点覆盖（HDU 1054也是）
{% highlight c %}
/*最小的启动次数 ，意思就是用最少得点覆盖所有的边
很明显的最小顶点覆盖*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int NV = 1500;
const int NE = 10000;
 
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
 
    inline void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline int Hungary(int n) {
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
 
    inline bool Find(int u) {
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
 
 
int main(void) {
       int n, m, k, c, a, b;
       while (~scanf("%d", &n), n) {
           scanf("%d%d", &m, &k);
           G.init(m + n);
           while (k--) {
               scanf("%d%d%d", &c, &a, &b);
               if (a == 0 || b == 0) continue;
               G.insert(a, b + n); 
               G.insert(b + n, a);
         }//while(k--)
         printf("%d\n", G.Hungary(n));
      }//while
   return 0;
}
{% endhighlight %}

HDU 1068 Girls and Boys 最大独立点集
{% highlight c %}
/*
没有 romantically involved”的集合，即没有匹配的集合
即最大的独立点集  
最大独立点集 = n - 最大匹配数
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int NV = 1005;
const int NE = 10000;
 
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
 
    inline void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline int Hungary(int n) {
        int match = 0;
        memset(aug, -1, sizeof(aug));
        for (int i = 0; i < n; i++) {
            if (aug[i] == -1) {//剪枝
                memset(mark, false, sizeof(mark));
                match += Find(i);
            }
        }
        return match;
    }//Hungray
 
    inline bool Find(int u) {
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
 
int main(void) {
    int n, m, k, a;
    while (~scanf("%d", &n)) {
        G.init(n);
        for (int i = 0; i < n; i++) {
            scanf("%d: (%d)", &m, &k);
            while(k--) {
                scanf("%d", &a);
                G.insert(m, a);
            }
        }//for
        printf("%d\n", n - G.Hungary(n));
    }
    return 0;
}
{% endhighlight %}

HDU 1151 Air Raid DAG图（有向无环图）的最小路径覆盖
{% highlight c %}
/*DAG图的最小路径覆盖，
最小路径覆盖数 = 顶点数 - 最大匹配数
解释：一条路径覆盖两个点，意思是最大匹配可以覆盖->(2 * 最大匹配)个点
一个匹配，一条路径
剩下的点都需要单独一条边覆盖
所以:最大匹配数 + (n - 2 * 最大匹配数）= 最小路径覆盖
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int NV = 250;
const int NE = 250;
 
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
 
    inline void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline int Hungary(int n) {
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
 
    inline bool Find(int u) {
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
 
 
int main(void) {
       int cas, n, m, a, b;
       scanf("%d", &cas);
       while (cas--) {
           scanf("%d %d", &n, &m);
           //n->vertex, m->edge
           G.init(n);
           while (m--) {
               scanf("%d%d", &a, &b);
               G.insert(a, n + b);
               G.insert(b + n, a);
           }
           printf("%d\n",  n - G.Hungary(n));
       }
       return 0;
}
{% endhighlight %}

HDU 1281 棋盘游戏 二分匹配 + 删边
{% highlight c %}
/*
有一句很重要的话：“注意不能放车的地方不影响车的互相攻击。 ”
如果影响的话，这题的建图是类似HDU 1045的
因为不影响，所以可以以整行，整列为单位建图 
  
因为要删边，矩阵方便点
求最大匹配数，但是这题还有一个要求是：
求有多少个关键点，那就模拟下，原来有边的点除去,检测最大匹配数是否和原来一样
如果一样，则不是关键点，如果是，则是关键点，记录之 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 101;
 
int map[maxn][maxn];
int kp[maxn * maxn][2];
bool mark[maxn];
int aug[maxn];//增广路径
 
int n, m;
  
inline bool Find (int t) {
    for (int i = 1; i <= m; i++) {
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
 
inline int Hungary(int n, int m) {
    int match = 0;
    memset(aug, -1, sizeof(aug));
    for (int i = 1; i <= n; i++) {
        memset(mark, false, sizeof(mark));
        match += Find(i);
    }
    return match;
}
 
int main() {
    int k;
    int cases = 1;
    while (~scanf("%d%d%d", &n, &m, &k)) {
        memset(map, 0, sizeof(map));
         for (int i = 0; i < k; i++) {
             int a, b;
             scanf("%d%d", &a, &b);
             map[a][b] = 1;
             kp[i][0] = a;
             kp[i][1] = b;
         }
          
         int res = Hungary(n, m);
         int cnt = 0;
         for (int i = 0; i < k; i++) {
             map[kp[i][0]][kp[i][1]] = 0;
             int tmp = Hungary(n, m);
             if (tmp < res) cnt++; 
             map[kp[i][0]][kp[i][1]] = 1;
         }
          printf("Board %d have %d important blanks for %d chessmen.\n", cases++, cnt, res);
    }
    return 0;
}
{% endhighlight %}

HDU 3478 Catch 判断是否是二分图，奇偶染色，然后广搜
{% highlight c %}
/*如果图是连通的且不是二分图，那么YES
判断二分图，可以用黑白染色法，如果相邻两个点颜色相同
那么不是二分图*/
 
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
 
    inline void init(int vx) {
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
 
 
inline void BFS() {
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
        BFS();
    }
    return 0;
}
{% endhighlight %}
HDU 2444 The Accomodation of Students 先判断是不是二分图，如果是，再求最大匹配
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
 
    inline void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline int Hungary(int n) {
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
 
    inline bool Find(int u) {
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
 
 
inline bool BFS() {
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
             
         if (!BFS()) {
            puts("No");
            continue;
         }//不能分为两组
         printf("%d\n", G.Hungary(n));
        }
    return 0;
}
 
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
 
bool BFS() {//判断是否是二分图 
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
     
     
inline bool Find (int t) {
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
 
inline int Hungary(int n, int m) {
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
        if (!BFS()) {
            puts("No");
            continue;
        }
        int cnt = 0;
        printf("%d\n", Hungary(n, n) / 2);
    }
    return 0;
}
{% endhighlight %}

HDU 1045 Fire Net 把每行每列相连的空地看为一个点，然后二分匹配
{% highlight c %}
/*
障碍物隔开的blockhouse不相互攻击，所以建图
需要把 每行内，相连的空地看为一个点； 
每列内，相连的空地看为一个点； 
然后，建图，二分匹配
最大流应该也能做吧 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 20;
 
int map[maxn][maxn];
char kp[5][5];
int kp1[5][5], kp2[5][5];
bool mark[maxn];
int aug[maxn];//增广路径
 
int n, m;
  
inline bool Find (int t) {
    for (int i = 0; i < m; i++) {
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
 
inline int Hungary(int n, int m) {
    int match = 0;
    memset(aug, -1, sizeof(aug));
    for (int i = 0; i < n; i++) {
        memset(mark, false, sizeof(mark));
        match += Find(i);
    }
    return match;
}
 
 
int main(void) {
    while (~scanf("%d", &n), n) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                map[i][j] = 0;
                kp1[i][j] = -1;
                kp2[i][j] = -1;
                kp[i][j] = 0;
            }
        }//初始化
 
        int cnt1 = 0, cnt2 = 0;
        for (int i = 0; i < n; i++) {
            scanf("%s", kp[i]);
        }
 
 
        /* 以列缩点
        1 0 2 3
        1 4 2 3
        0 0 2 3
        5 6 2 3  */
        for (int i = 0; i < n; i++) {//以列缩点, k是行,然后往下推进
            for (int j = 0; j < n; j++) {
                if (kp[i][j] == '.' && kp1[i][j] == -1) {
                    for (int k = i; kp[k][j] == '.' && k < n; k++) {
                        kp1[k][j] = cnt1;
                    }
                    cnt1++;
                }
            }
        }
 
        /*以行缩点：
        1 0 2 2
        3 3 3 3
        0 0 4 4
        5 5 5 5 */
 
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (kp[i][j] == '.' && kp2[i][j] == -1) {
                    for (int k = j; kp[i][k] == '.' && k < n; k++) {
                        kp2[i][k] = cnt2;
                    }
                    cnt2++;
                }
            }
        }
 
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (kp[i][j] == '.') {
                    map[kp1[i][j]][kp2[i][j]] = map[kp2[i][j]][kp1[i][j]] = 1;
                }
            }
        }
        n = cnt1, m = cnt2;
 
        int res = Hungary(cnt1, m);
        printf("%d\n", res);
    }
    return 0;
}
{% endhighlight %}

HDU 3081 Marriage Match  II 最大匹配 + 删边，问有多少种最大匹配方案
{% highlight c %}
/*总共三种关系：不吵架关系，同性朋友关系，异性朋友关系 
a,b不吵架，a,b可以成为异性朋友；(题目中的m） 
a和b是朋友，b和c是朋友，那么a和c是朋友(同性朋友)  (题目中的f)
要找的是异性朋友关系，它的要求是：
x(Girl)和y(Girl)是同性朋友，y(Girl)和z(boy)不吵架，
那么x（Girl）和z(Boy) 可以成为异性朋友 
然后问，有多少中全部异性全部匹配的方案？
步骤： 
 首先，不吵架的男女之间建边（m个关系本身就满足条件）
 然后通过朋友关系 + 不吵架关系 = 把所有可能的异性关系 （建边） 
匹配一次，把匹配过的边去掉，然后再次寻找匹配
 直到不能全部匹配为止 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 110;
 
bool fmy[maxn][maxn];
bool frd[maxn][maxn]; 
bool mark[maxn];
int aug[maxn];//增广路径
 
int n;
  
inline bool Find (int t) {
    for (int i = 1; i <= n; i++) {
        if (!mark[i] && fmy[t][i]) {
            mark[i] = true;
            if (aug[i] == -1 || Find(aug[i])) {
                aug[i] = t;
                return true;
            }
        }
    }//for(i)
    return false;
}
 
inline bool Hungary(int n) {
    int match = 0;
    memset(aug, -1, sizeof(aug));
    for (int i = 1; i <= n; i++) {
        memset(mark, false, sizeof(mark));
        match += Find(i);
    }
    for (int i = 1; i <= n; i++) {//建立过家庭的边删去 
        if (aug[i] != -1) {
            fmy[aug[i]][i] = 0;
        }
    }
    if (match == n) return 1;
    else return 0;
}
 
int main()
{
    int T,i,j,m,f,k;
    int cas;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d%d%d", &n, &m, &f);
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                fmy[i][j] = frd[i][j] = 0;
            }
        }
         
        int a, b;
        for (int i = 1; i <= m; i++) {
            scanf("%d%d", &a, &b);
            fmy[a][b] = 1;
        }
        for (int i = 1; i <= f; i++) {
            scanf("%d%d", &a, &b);
            frd[a][b] = frd[b][a] = 1;
        }
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                for (int k = 1; k <= n; k++) {
                    if (frd[i][k] && frd[k][j]) frd[i][j] = 1;
                }
            }
        }
         
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) if (frd[i][j]) {
                for (int k = 1; k <= n; k++) {
                    if (fmy[j][k]) {
                        fmy[i][k] = 1;
                    }
                }
            }
        }
        int res = 0;
        while (Hungary(n)) {
            res++;
        }
        printf("%d\n", res);
    }
    return 0;
}
{% endhighlight %}



HDU 1507 Uncle Tom’s Inherited Land* 最大匹配 + 匹配输出
{% highlight c %}
/*
很明显的二分匹配，奇数点左边，偶数点右边，建边 
主要是要路径输出；
这个可以输出点--增广路找到的点 ，同时要标记一下 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
int dir[4][2] = { {0, 1}, {0, -1}, {1, 0}, {-1, 0} };
const int NV = 10005;
const int NE = 40005;
const int maxn = 105;
int kp[maxn][maxn];
int n, m;
 
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
 
    inline void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline int Hungary(int n) {
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
 
    inline bool Find(int u) {
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
 
 
int Overmap(int x, int y) {
    if (x < 1 || x > n || y < 1 || y > m) {
        return true;
    }
    return false;
}
 
int main(void) {
    while (~scanf("%d%d", &n, &m), n || m) {
        int k;
        scanf("%d", &k);
        memset(kp, 0, sizeof(kp));
        while (k--) {
            int a, b;
            scanf("%d%d", &a, &b);
            kp[a][b] = 1;
        }
         
        G.init(NV);
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) if (!kp[i][j]) {//空余 
                for (int k = 0; k < 4; k++) {//遍历周围的点 
                    int tpx = i + dir[k][0];
                    int tpy = j + dir[k][1];
                    if (!Overmap(tpx, tpy) && !kp[tpx][tpy]) {
                        if ((i + j) & 1) {
                            G.insert((i - 1) * m + j, (tpx - 1) * m + tpy);
                        } else {
                            G.insert((tpx - 1) * m + tpy, (i - 1) * m + j);
                        }
                    }
                }
            }
        }//for(i)
         
        int res = G.Hungary(n * m);
        printf("%d\n", res);
        memset(G.mark, false, sizeof(G.mark));
        for (int i = 1; i <= n * m; i++) {
            if (G.aug[i] != -1 && !G.mark[i] && !G.mark[G.aug[i]]) {
                G.mark[i] = true;
                G.mark[G.aug[i]] = true;
                int x = (i - 1) / m + 1;
                int y = (i - 1) % m + 1;
                int x1 = (G.aug[i] - 1) / m + 1;
                int y1 = (G.aug[i] - 1) % m + 1;
                printf("(%d,%d)--(%d,%d)\n", x, y, x1, y1);
            }
        }
        puts("");
    }
    return 0;
}      
{% endhighlight %}


HDU 3360 National Treasures 比较有意思的最小顶点覆盖
{% highlight c %}
/*
题目意思很简单，每个文物周围都最多有
12个关键点，而这12个关键点，第i个的有无 是由他的二进制从
右到左是否是1来决定（这个可以通过移位判断）
如果关键点上有其他文物，那么建边，一条边代表一个冲突
最少的保安，意思是，用最少的点覆盖掉所有的冲突，即
最小顶点覆盖 = 最大匹配 
 
另外，这个还不许二分图，只能奇数点向偶数点建边，不然就不是二分图了
这个没注意过 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
int dir[12][2] = { {-1, -2}, {-2, -1}, {-2, 1}, {-1, 2},
                  {1,  2}, {2, 1}, {2, -1}, {1, -2},
                  {-1, 0}, {0, 1}, {1, 0}, {0, -1} };
const int maxn = 55;
int kp[maxn][maxn];
 
const int NV = 2500;
const int NE = 30000;    
int n, m;
 
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
 
    inline void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline int Hungary(int n) {
        int match = 0;
        memset(aug, -1, sizeof(aug));
        for (int i = 0; i < n; i++) {
            if (aug[i] == -1) {//剪枝
                memset(mark, false, sizeof(mark));
                match += Find(i);
            }
        }
        return match;
    }//Hungray
 
    inline bool Find(int u) {
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
 
bool Overmap(int x, int y) {
    if (x < 0 || x >= n || y < 0 || y >= m) {
        return true;
    }
    return false;
}
 
int main(void) {
    int cases = 1;
    while (~scanf("%d%d", &n, &m), n || m) {
        memset(kp, 0, sizeof(kp));
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                scanf("%d", &kp[i][j]);
            }
        }
        G.init(maxn * maxn);
        for (int i = 0; i < n; i++) {
            for (int  j = 0; j < m; j++) if (kp[i][j] != -1){//是文物 
                    for (int k = 0; k < 12; k++)  if((kp[i][j] >> k) & 1) {//是关键点                 
                        int tpx = i + dir[k][0];
                        int tpy = j + dir[k][1];
                        if (!Overmap(tpx, tpy) && kp[tpx][tpy] != -1) {
                            if ((i + j) & 1) {//奇数点 
                                 G.insert(i * m + j, tpx * m + tpy);
                            } else {
                                G.insert(tpx * m + tpy, i * m + j);
                            }
                        }
                }//for(k)
            }//for(j)
        }//for(i)
         
        int res = G.Hungary(n * m);
        printf("%d. %d\n", cases++, res);
    }
    return 0;
}
{% endhighlight %}

HDU 2768 Cat vs. Dog 要让冲突最小，就是最大独立点集了（HDU 3829和这题一样）
{% highlight c %}
/*
每个观众不是cat_lover,就是dog_lover 
当观众想要留下来的宠物出局时，这个观众就会离开，
为了保证观众留下的最多，必须让互相冲突的数量最小，
可以把冲突的人之间建边，然后，最大独立点集，就是
冲突最小的点集，也就是留下来人最多的情况
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int NV = 505;
const int NE = 70000;
 
struct Node {
    int cat, dog;
}cat_lover[NV], dog_lover[NV];
 
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
 
    inline void init(int vx) {
        n = vx, size = 0;
        for (int i = 0; i <= n; i++) {
            head[i] = -1;
        }
    }
 
    inline void insert(int u, int v, int w = 0) {
        E[size] = Edge(v, head[u], w);
        head[u] = size++;
    }
 
    inline int Hungary(int n) {
        int match = 0;
        memset(aug, -1, sizeof(aug));
        for (int i = 0; i < n; i++) {
            if (aug[i] == -1) {//剪枝
                memset(mark, false, sizeof(mark));
                match += Find(i);
            }
        }
        return match;
    }//Hungray
 
    inline bool Find(int u) {
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
 
 
int main() {
    int cas;
    scanf("%d", &cas);
    while (cas--) {
        int c, d, v;
        scanf("%d%d%d", &c, &d, &v);
        int cnt1 = 0, cnt2 = 0;
        for (int i = 0; i < v; i++) {
            getchar ();
            char ch1, ch2;
            int tmp1, tmp2;
            scanf("%c%d", &ch1, &tmp1);
            getchar();
            scanf("%c%d", &ch2, &tmp2);
 
            if (ch1 == 'C') {
                cat_lover[cnt1].cat = tmp1;
                cat_lover[cnt1].dog = tmp2;
                cnt1++;
            } else {
                dog_lover[cnt2].cat = tmp2;
                dog_lover[cnt2].dog = tmp1;
                cnt2++;
            }
        }
 
        G.init(NV);
        for (int i = 0; i < cnt1; i++) {
            for (int j = 0; j < cnt2; j++) {
                if (cat_lover[i].cat == dog_lover[j].cat || 
                    cat_lover[i].dog == dog_lover[j].dog) {
                    G.insert(i, cnt1 + j);
                }
            }
        }
         
        printf("%d\n", v - G.Hungary(cnt1));
    }
}
{% endhighlight %}

更新一道多校联合赛的二分匹配题

HDU 3861 The King’s Problem 强连通缩点 + 二分匹配最小路径覆盖
{% highlight c %}
/*多校联合赛2011 Multi-University Training Contest 3 - Host by BIT
强连通缩点 + 二分匹配 
标程：
题意是给定一个有向图，问能把这个有向图至少分为多少个没有重复点的部分，
使得每一部分内的任意两点之间可以不经过其它部分中的点实现单向连通。
由于有向图是任意的，里面可能有环，对于环内的点而言，任意两点是双向可达的，
所以可以把每个环缩成一个点，缩点之后的有向无环图内，一个单侧连通分量实际上对应应该是一条链，
这样问题就变成了求解至少能够拆成多少条没有公共点的链，其实就是最小路径覆盖问题，
直接通过二分图对缩点后的图求最小路径覆盖即可。 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
#include<stack>
using namespace std;
#define inf 0x7f7f7f7f
 
const int NV = 10005;
const int NE = 100005;
 
struct Tarjan {
    int dfn[NV];//深度优先搜索访问次序
    int low[NV];//能追溯到的最早的次序
    int index, scc, n;//index->索引号，scc->强连通分量个数
    bool instack[NV];//检查是否在栈中
    int root[NV];//记录每个点在第几号强连通分量里
    int aug[NV];
    bool mark[NV];
     
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
     
    inline void tarjan (int i) {
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
     
    inline void Solve(){
        while (!S.empty()) S.pop();
        for (int i = 0; i <= n; i++) {
            instack[i] = 0;
            dfn[i] = 0;
            low[i] = 0;
        }
        index = scc = 0;
        for (int i = 1; i <= n; i++) {
            if(!dfn[i]) tarjan(i);
        }
    }
 
    inline int Hungary(int n) {
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
 
    inline bool Find(int u) {
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
     
    struct Relation {
        int u,v;
    }re[NE];
}G;
 
 
int main() {
    int cas;
    scanf("%d", &cas);
    while (cas--) {
        int n, m;
        scanf("%d %d", &n, &m);
        G.init(n);
         
        for (int i = 0; i < m; i++) {
            scanf("%d %d", &G.re[i].u, &G.re[i].v);
            G.insert(G.re[i].u, G.re[i].v);
        }
        G.Solve();
         
        if (G.scc == 1) {
            puts("1");
            continue;
        }
        G.init(G.scc * 2 + 2);
 
        for (int i = 0; i < m; i++) {//缩点，建立新图 
            int u = G.re[i].u;
            int v = G.re[i].v;
            if (G.root[u] != G.root[v]) {
                 G.insert(G.root[u], + G.scc + G.root[v], 1);
            }
        }
         
        int res = G.scc - G.Hungary(G.scc);
        printf("%d\n", res);
    }
    return 0; 
} 
{% endhighlight %}