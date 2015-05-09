---
title: 最长公共子序列和最优二叉查找树
author: Wei Li
excerpt: 这里要探讨是动态规划中的两个经典子问题：最长公共子序列（longest common subsequence，LCS）和最优二叉查找树。算法导论这一章中，最重要的应该是里边讲到的分析DP的方法（按步骤分析），下边两个问题都是按这种方法分析。
meta_description: 这里要探讨是动态规划中的两个经典子问题：最长公共子序列（longest common subsequence，LCS）和最优二叉查找树。算法导论这一章中，最重要的应该是里边讲到的分析DP的方法（按步骤分析），下边两个问题都是按这种方法分析。
layout: post
permalink: /2011/09/06/lcs-and-osbst/
views:
  - 5407
categories:
  - 算法学习
tags:
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
  - 读书
---
如果你看完这个题目(HDU 2084 数塔)，能看懂下边的代码，说明你已经知道什么是DP动态规划了。

HDU2084代码：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cmath>
using namespace std;
 
int map[101][101];
int main() {
    int cas, n;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= i; j++) {
                scanf("%d", &map[i][j]);
            }
        }
         
        for (int i = n - 1; i >= 1; i--) {
            for (int j = 1; j <= n - 1; j++) {
                map[i][j] += max(map[i + 1][j], map[i + 1][j + 1]);
            }
        }//for(i)
         
        printf("%d\n", map[1][1]);
    }
}
{% endhighlight %}

这里要探讨是动态规划中的两个经典子问题：最长公共子序列（longest common subsequence，LCS）和最优二叉查找树。算法导论这一章中，最重要的应该是里边讲到的分析DP的方法（按步骤分析），下边两个问题都是按这种方法分析。                       

###一、最长公共子序列（LCS）

如果Z是X的一个子序列又是Y的一个最长公共子序列。例如，如果X = {A, B, C, B, D, A, B}，Y = {B, D, C, A, B, A}，则{B，C, B, A}是X和Y的一个LCS， 序列{B, D, A, B}也是，因为没有长度为5的或更大公共序列里。

1) 梳理状态的变化情况，找到最优子结构：

设X =（x1,x2,x3….xm)，Y = (y1,y2,y3…yn)为两个序列，并设Z = (z1,z2,z3…zk)为X和Y的任意一个LCS。

1. 如果Xm == Yn,那么Zk == Xm == Yn 而且Z(k-1)是X(m-1)和Y(n-1)的一个LCS;

2. 如果Xm != Yn,那么Zk != Xm 蕴含Z是X(m-1)和Y的一个LCS;

3. 如果Xm != Yn,那么Zk != Yn 蕴含Z是X和Y(n-1)的一个LCS;

2) 找到一个递归的解:

如果xm == ym,必须找到X(m – 1)和Y( n – 1)的一个LCS。将xm = ym添加到这个LCS上，可以产生X和Y的一个LCS；如果xm != yn， 就必须解决两个子问题：找到X(m – 1)和Y的一个LCS， 以及找出X和Y(n – 1)的一个LCS， 这两个LCS中，较长的就是X和Y的一个LCS。由LCS的最优子结构可得到递归式（c[i, j]为序列Xi和Yi的一个LCS长度)：
![Image](/uploads/2011/09/DPLCS.jpg)
3) 计算LCS的长度，代码实现：（下边代码可以A掉HDU 1159 Common Subsequence):

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
#define N 1002
int num[N][N];
char str1[N], str2[N];
 
int LCS(char *str1, char *str2, int num[][1002]) {
    int len1 = strlen(str1);
    int len2 = strlen(str2);
    for (int i = 0; i < len1; i++) {
        num[i][0] = 0;
    }
    for (int i = 0; i < len2; i++) {
        num[0][i] = 0;
    }
 
    for (int i = 0; i < len1; i++) {
        for (int j = 0; j < len2; j++) {
            if (str1[i] == str2[j]) {
                num[i + 1][j + 1] = num[i][j] + 1;
            } else {
                if (num[i][j + 1] >= num[i + 1][j]) {
                    num[i + 1][j + 1] = num[i][j + 1];
                } else {
                    num[i + 1][j + 1] = num[i + 1][j];
                }
            }//if
        }//for(j)
    }//for(i)
    return num[len1][len2];
}
 
int main() {
    while (~scanf("%s %s", str1, str2)) {
        int res = LCS(str1, str2, num);
 
        printf("%d\n", res);
    }
    return 0;
}
{% endhighlight %}
4) 构造出一个LCS，可以输出路径：

表b[1……m, 1……n]代表所选择最优子问题的解，可以理解为最优解的路径来源。

函数LCS-LNGTH(X, Y)用来返回LCS长度， PRINT-LCS用来输出LCS，图代表X = {A, B, C, B, D, A, B}，Y = {B, D, C, A, B, A},执行LCS-LNGTH(X, Y)的过程：  
![Image](/uploads/2011/09/DPLCS_PIG.jpg)
代码实现：
{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
using namespace std;
 
const int N = 1002;
const bool left_side = 0;//代表朝左箭头 
const bool up = 1;//代表朝上箭头 
int c[N][N], b[N][N];
char str1[N], str2[N];
int len1, len2;
 
// str1 = {A, B, C, B, D, A, B}
// str2 = {B, D, C, A, B, A}
int LCSLength(int b[][1002], int c[][1002]) {
    for (int i = 0; i < len1; i++) {
        c[i][0] = 0;
    }
    for (int i = 0; i < len2; i++) {
        c[0][i] = 0;
    }
 
    for (int i = 0; i < len1; i++) {
        for (int j = 0; j < len2; j++) {
            if (str1[i] == str2[j]) {
                c[i + 1][j + 1] = c[i][j] + 1;
                b[i + 1][j + 1] = 0;
            } else {
                if (c[i][j + 1] >= c[i + 1][j]) {
                    c[i + 1][j + 1] = c[i][j + 1];
                    b[i + 1][j + 1] = 2; 
                } else {
                    c[i + 1][j + 1] = c[i + 1][j];
                    b[i + 1][j + 1] = 1;
                }
            }//if
        }//for(j)
    }//for(i)
    return c[len1][len2];
}
 
void PrintLCS(int b[][1002], int i, int j) {
    if (i == 0 || j == 0) {
        return ;
    }
    if (b[i][j] == left_side) {
        PrintLCS(b, i - 1, j - 1);
        printf("%c", str1[i - 1]);
    } else if (b[i][j] == up) {
        PrintLCS(b, i - 1, j);
    } else {
        PrintLCS(b, i, j - 1);
    }
}
 
//打印辅助函数 
void Print(int aa[][1002]) {
    for (int i = 0; i <= len1; i++) {
        for (int j = 0; j <= len2; j++) {
            cout << aa[i][j] << ' ';
            }cout << endl;
   }
}
     
int main() {
    while (~scanf("%s %s", str1, str2)) {
        len1 = strlen(str1);
        len2 = strlen(str2);
        int res = LCSLength(b, c);
        printf("LCS长度为：%d\n", res);
        printf("DP过程：\n"); 
        Print(c);
         
        printf("LCS为： ");
        PrintLCS(b, len1, len2);    
    }
    return 0;
}
 
/*
ABCBDAB BDCABA
 
LCS长度为：4
DP过程：
0 0 0 0 0 0 0
0 0 0 0 1 1 1
0 1 1 1 1 2 2
0 1 1 2 2 2 2
0 1 1 2 2 3 3
0 1 2 2 2 3 3
0 1 2 2 3 3 4
0 1 2 2 3 4 4
LCS为： BDAB
 
*/
{% endhighlight %}
###二、最优二叉查找树

给定一个由n个互异的关键字组成的序列K = (k1, k2, ……， kn）,且关键字有序（因此有k1 < k2 < …… < kn)，从这些关键字中构造一棵二叉查找树。对每个关键字ki， 一次搜索为ki的概率是pi。某些搜索的值可能不在K内，因此还有n + 1个“虚拟键”d0, d1, d2, ……dn代表不在K内的值，且ki ≤ di ≤ k(i+1)，di概率为qi。

因为搜索每个关键字的概率不同，因此最优二叉查找树即一棵期望搜索代价最小的二叉查找树。

从下图a), b)中可以看出，最优二叉查找树并不是高度最低的树，因为第一棵树期望是2.80，而第二棵是2.75。公式(15.10)显而易见。假设一次搜索的实际代价为检查的结点个数，亦即，在T内搜索所发现的结点的深度加上1， 所以一次搜索的期望代价为公式(15.11)，亦即，期望 = 深度 * 概率； 
![Image](/uploads/2011/09/The-optimal-binary-search-trees.jpg)


1）梳理状态的变化情况，找到最优子结构：

如果有一棵子树T” 其期望比T’小，那么我们可以把T’从T中剪下，然后贴上T”， 而产生一个期望代价比T小的二叉查找树。

2）找到一个递归的解:

子问题是，找一个包含关键字ki, ……kj的最优二叉查找树，其中i >= 1, j <= n而且j >= i – 1， j <= n而且j >= i – 1（当j=i-1时没有真实的关键在，只有虚拟键d(i-1)）, 定义e[i,j]为一棵包含关键字ki~kj的最优二叉树的期望代价。

当j = i – 1时，此时只有虚拟键d(i – 1), 期望搜索代价是e[i,i-1]=q(i-1)。

当 j >= i 时，需要选择合适的kr作为根节点，然后其余节点ki ~ K(r-1)和k(r+1) ~ kj分别构造左右孩子。

当一棵树成为一个结点的子树时，它的搜索代价的变化：子树中每个结点的深度增加1，由公式（15.11），这个子树的期望搜索代价增加为子树中所有概率的总和。w[i, j]为概率总和,e[i, j]为最优二叉查找树的期望搜索代价。（下边图中的推导过程非常牛逼），得到递归式。
![Image](/uploads/2011/09/OBST_forluma.jpg)

3）计算一棵最优二叉查找树的期望搜索代价

把e[i, j]保存在表e[1……n + 1, 0……n], 第一维的下标需要达到n + 1而不是n， 原因是为了有一个只包含虚拟键dn的子树，需要计算和保存e[n + 1, n]。第二维的下标需要从0开始，是为了保存e[1, 0]。root[I, j]来记录包含关键字k1, ……，kj的子树的根。对于w[i, j]可以用公式w[i, j] 来计算，这样效率更高。

最优二叉查找树伪代码：
![Image](/uploads/2011/09/obst_code1.jpg)

第5, 6行两层循环：

第一次迭代中，l = 1, 循环计算e[i, i]和w[i, i]，对于i = 1, 2, ……, n。

第二次迭代中，l = 2, 循环计算e[i, i + 1]和w[i, i + 1], 对于i = 1, 2, ……, n – 1；

第三次迭代中,  l = 3, 循环计算e[i, i + 2]和w[i, i + 2], 对于i = 1, 2, ……, n – 2；

如果观察下边图，可以发现，它是自底向上一层一层往上迭代。看似像树的分布实际上是由表旋转而来，原来的可以看箭头所指的图（旋转了45°), 9~13行即利用(15.14)的公式跟新root和e，类似文章刚开始的数塔。
![Image](/uploads/2011/09/OBST_PIC2.jpg)

{% highlight c %}
/*
e[i][j] :
0.05 0.45 0.90 1.25 1.75 2.75
     0.10 0.40 0.70 1.20 2.00
          0.05 0.25 0.60 1.30
               0.05 0.30 0.90
                    0.05 0.50
                         0.10
 
w[i][j]：
0.05 0.30 0.45 0.55 0.70 1.00
     0.10 0.25 0.35 0.50 0.80
          0.05 0.15 0.30 0.60
               0.05 0.20 0.50
                    0.05 0.35
                         0.10
 
root[i][j]:
1 1 2 2 4
  2 2 2 4
    3 4 5
      4 5
        5
*/
#include<iostream>
#include<cstdio>
#include<climits>
using namespace std;
 
//以5个元素为例 
//k1 < k2 < k3 < k4 < k5
double p[6] = {0, 0.15, 0.1, 0.05, 0.1, 0.2};
double q[6] = {0.05, 0.1, 0.05, 0.05, 0.05, 0.1};
 
double e[10][10], w[10][10];
int  root[10][10];
 
const int N = 5;
 
void OptimalBST(int n) {
    for (int i = 1; i <= n + 1; i++) {
        e[i][i - 1] = q[i - 1];
        w[i][i - 1] = q[i - 1];
    }
    for (int l = 1; l <= n; l++) {
        for (int i = 1; i <= n - l + 1; i++) {
            int j = i + l - 1;
            e[i][j] = INT_MAX;
            w[i][j] = w[i][j - 1] + p[j] + q[j];
            for (int r = i; r <= j; r++) {
                double t = e[i][r - 1] + e[r + 1][j] + w[i][j];
                if (t < e[i][j]) {
                    e[i][j] = t;
                    root[i][j] = r;
                }
            }//for(r) 
        }//for(i) 
    }//for(l) 
}
 
int main() {
     OptimalBST(N);
     puts("e[i][j] :");
     for (int i = 1; i <= 6; i++) {
        for (int j = 0; j <= 5; j++) {
            if (!e[i][j]) {
                printf("     ");
                continue;
            } 
            printf("%.2f ", e[i][j]);
        }puts("");
     }
 
     puts("\nw[i][j]："); 
     for (int i = 1; i <= 6; i++) {
        for (int j = 0; j <= 5; j++) {
            if (!w[i][j]) {
                printf("     ");
                continue;
            } 
            printf("%.2f ", w[i][j]);
        }puts("");
     }
      
     puts("\nroot[i][j]:");
     for (int i = 1; i <= 5; i++) {
        for (int j = 1; j <= 5; j++) {
            if (!root[i][j]) {
                printf("  ");
                continue;
            } 
            printf("%d ", root[i][j]);
        }puts("");
     }
     return 0;
}
{% endhighlight %}
            
