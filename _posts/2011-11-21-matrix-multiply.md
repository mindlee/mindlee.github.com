---
title: 矩阵乘法(Matrix Multiply)
author: Wei Li
excerpt: '<p>&#160;&#160;&#160;&#160; 两个矩阵 A，B：A 的列数等于 B 的行数，则A、B可以相乘。即，如果 A = （a<sub>ij</sub>）是一个m * n的矩阵，B =（b<sub>jk</sub>）是一个 n * p 的矩阵，则它们的乘积 C = AB 是 m * p 矩阵 C = （c<sub>ik</sub>）。</p>'
layout: post
permalink: /2011/11/21/matrix-multiply/
views:
  - 11574
categories:
  - 算法学习
tags:
  - 数据结构
  - 矩阵乘法
  - 笔记
  - 算法
  - 算法导论
---
两个矩阵 A，B：A 的列数等于 B 的行数，则A、B可以相乘。即，如果 A = （aij）是一个m * n的矩阵，B =（bjk）是一个 n * p 的矩阵，则它们的乘积 C = AB 是 m * p 矩阵 C = （cik）。

学过计算机图形学，会发现，不管是二维图形的平移，旋转，缩放，三维图形的取景变换，投影变换等都是通过矩阵乘法来实现，例如，二维点P（x，y）平移（tx，ty）后得到 P'(x’，y’)，可以通过矩阵计算：

$$
\begin{pmatrix} x'\\ y'\\ 1 \end{pmatrix}= \begin{pmatrix} 1 & 0 & tx\\ 0 & 1 & ty\\ 0 & 0 & 1 \end{pmatrix}\begin{pmatrix} x\\ y\\ 1 \end{pmatrix}
$$

$$
x'= 1 * x + 0 * y + tx * 1 = x + tx 
$$

$$
y'= = 0 * x + 1 * y + ty * 1 = y + ty
$$

求矩阵相乘的通用公式为：

$$
AB_{ij} = \sum_{r = 1}^{n}a_{ir}b_{rj} =a_{i1}b_{1j} + a_{i2}b_{2j}+……+a_{in}+b_{nj} 
$$

还有一个常见用法，求斐波那契数列：Fi+1 = Fi + Fi-1

$$
\begin{pmatrix} 1 & 1\\ 1& 0 \end{pmatrix} * \begin{pmatrix} Fi\\ Fi-1 \end{pmatrix} = \begin{pmatrix} Fi + Fi-1\\ Fi \end{pmatrix} = \begin{pmatrix} Fi+1\\ Fi \end{pmatrix}
$$

$$
\begin{vmatrix} 1 &1 \\ 1 & 0 \end{vmatrix} * \begin{vmatrix} 8\\ 5 \end{vmatrix}=\begin{vmatrix} 13\\ 8 \end{vmatrix}
$$

所以当求 Fn 时可以用求幂来快速求出，而求幂也是建立在矩阵乘法的基础上：

$$
\begin{vmatrix} 1 &1 \\ 1&0 \end{vmatrix} * \begin{vmatrix} 1 &1 \\ 1&0 \end{vmatrix} * \begin{vmatrix} 1 &1 \\ 1&0 \end{vmatrix} * ……* \begin{vmatrix} 1 &1 \\ 1&0 \end{vmatrix} * \begin{vmatrix} F1\\ F0 \end{vmatrix} = \begin{vmatrix} Fn\\ Fn-1 \end{vmatrix}
$$

$$
A = \begin{vmatrix} F1\\ F0 \end{vmatrix}B =\begin{vmatrix} 1 &1 \\ 1& 0 \end{vmatrix}C = \begin{vmatrix} Fn\\ Fn-1 \end{vmatrix}
$$

$$
C = B^{n-1}*A
$$

矩阵相乘有两种方法，普通的矩阵乘法（Matrix Multiply）和Strassen算法。

###一、矩阵乘法（Matrix Multiply）

一个矩阵在计算机里就是一个二维数组，所以矩阵ADT：

{% highlight c %}
const int N = 11;
static int mod = 10000;
 
struct Matrix {
    int  mat[N][N]; //修改long long or int
    int n, m;
 
    Matrix() {//初始化
        n = m = N; 
        memset(mat, 0, sizeof(mat));
    }
     
    inline void init(int row, int column) {//初始矩阵大小
        n = row, m = column;
    }
 
    void init_e() {//单位矩阵
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                mat[i][j] = (i == j);
            }
        }
    }
};
{% endhighlight %}

最普通的矩阵乘法甚至不算是算法，只是直接三个for循环直接计算而已，所以复杂度是O（$n^3$），伪代码：

	MATRIX-MULTIPLY(A, B)
	1  n ← rows[A]
	2  let C be an n × n matrix
	3  for i ← 1 to n
	4      do for j ← 1 to n
	5           do cij ← 0
	6              for k ← 1 to n
	7                do cij ← cij + aik · bkj
	8  return C
代码实现是：

{% highlight c %}
//n行m列  *  m行p列   =  n行p列
Matrix operator * (Matrix a, Matrix b) {
    Matrix ret; 
    ret.init(a.n, b.m);
    for (int i = 0; i < a.n; i++) {
        for (int k = 0; k < a.m; k++) if (a.mat[i][k]) {
            for (int j = 0; j < b.m; j++) if (b.mat[k][j]) {
                ret.mat[i][j] = ret.mat[i][j] + a.mat[i][k] * b.mat[k][j];  
                if (ret.mat[i][j] >= mod) {
                    ret.mat[i][j] %= mod;
                }//if
            }//for(j)
        }//for(k)
    }//for(i)
    return ret;
}//乘法
{% endhighlight %}

仔细观察，可以发现，C代码比伪代码多出两个 if 判断语句，不要小看它们，加上之后，除去不必要的计算，剪枝效果非常好，如果不清楚为什么，模拟一下即可。

另外，矩阵加法只需简单的将两个矩阵的元素相加：

{% highlight c %}
//加法
Matrix operator + (Matrix a, Matrix b) {
    Matrix ret; 
    ret.init(a.n, a.m);
    for (int i = 0; i < a.n; i++) {
        for (int j = 0; j < a.m; j++) {
            ret.mat[i][j] = a.mat[i][j] + b.mat[i][j];
            if (ret.mat[i][j] >= mod) {
                ret.mat[i][j] %= mod;
            }
        }
    }
    return ret;
}//加法
{% endhighlight %}

ACM中，还有一个很常见的矩阵计算，求幂，这里用到的求幂方式是，二分求幂，加入K = 25，则它的二进制表示为11001，所以：

$$
A^{k} = A^{11001(2)}\\~~~~~~~~~=A^{1 \times 2^{0} + 0 \times 2^{1} + 0 \times 2^{2} +1 \times 2^{3} + 1 \times 2 ^{4}}\\~~~~~~~~~=A^{1} \times A^{0} \times A^{0} \times A^{8} \times A^{16}\\~~~~~~~~~=A^{1} \times E \times E \times (((A^{1})^2)^2)^2 \times ((((A^{1})^2)^2)^2)^2
$$

所以，求幂可以利用二进制都转化到乘法上，代码实现：

{% highlight c %}
//求幂一般都是正方形矩阵，所以ret = a;
Matrix operator ^ (Matrix a, int b) {
    Matrix ret = a,  tmp = a;
    ret.init_e();
    for ( ; b; b >>= 1) {
        if (b & 1) {
            ret = ret * tmp;
        }
        tmp = tmp * tmp;
    }
    return ret;
}//
{% endhighlight %}

另外，很经常的还有一种非常常见的，幂求和，即求$S = A + A^2 + A^3 +… + A^k$  ，同样的二分思想，但是利用递归，可以很快求出：

{% highlight c %}
//递归幂求和
//用二分法求矩阵和,递归实现  
Matrix Power_Sum1(Matrix a, int b) {
    Matrix ret = a;
    ret.init_e();
    if (b == 1) {
        return a;
    } else if (b & 1) {
        return (a ^ b) + Power_Sum1(a, b - 1);
    } else {
        return Power_Sum1(a, b >> 1) * ((a ^ (b >> 1)) + ret);
    }
}//Power_Sum1
{% endhighlight %}

上边的递归代码虽然优雅，但是如果k很大，会非常占内存，更高效的方式是非递归方式，但是思想相同：

{% highlight c %}
//非递归幂求和
Matrix Power_Sum2(Matrix a, int b) {
    int k = 0 ,ss[32];
    Matrix tp1, tp2, ret;
    tp1 = tp2 = ret = a;
    ret.init_e();
    while (b) { 
        ss[k++] = b & 1;
        b >>= 1;
    } 
    for (int i = k - 2; i >= 0; i--) {
        tp1 = tp1 * (tp2 + ret);
        tp2 = tp2 * tp2;
        if (ss[i]) {
            tp2 = tp2 * a;
            tp1 = tp1 + tp2;
        }
    }
    return tp1;
}//Power_Sum2
{% endhighlight %}

完整的矩阵模板，如下：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int N = 52;
const int mod = 1111; 
 
struct Matrix {
    int  mat[N][N]; //修改long long or int
    int n, m;
 
    Matrix() {//初始化
        n = m = N; 
        memset(mat, 0, sizeof(mat));
    }
 
    inline void init(int row, int column) {//初始矩阵大小
        n = row, m = column;
    }
 
    void init_e() {//单位矩阵
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                mat[i][j] = (i == j);
            }
        }
    }
 
    void print() {//测试检查用
        puts("——————————————"); 
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                printf("%d ", (int)mat[i][j]);
            }puts("");
        }puts("");
    }
 
};
 
//加法
Matrix operator + (Matrix a, Matrix b) {
    Matrix ret; 
    ret.init(a.n, a.m);
    for (int i = 0; i < a.n; i++) {
        for (int j = 0; j < a.m; j++) {
            ret.mat[i][j] = a.mat[i][j] + b.mat[i][j];
            if (ret.mat[i][j] >= mod) {
                ret.mat[i][j] %= mod;
            }
        }
    }
    return ret;
}//加法
 
//n行m列  *  m行p列   =  n行p列
Matrix operator * (Matrix a, Matrix b) {
    Matrix ret; 
    ret.init(a.n, b.m);
    for (int i = 0; i < a.n; i++) {
        for (int k = 0; k < a.m; k++) if (a.mat[i][k]) {
            for (int j = 0; j < b.m; j++) if (b.mat[k][j]) {
                ret.mat[i][j] = ret.mat[i][j] + a.mat[i][k] * b.mat[k][j];  
                if (ret.mat[i][j] >= mod) {
                    ret.mat[i][j] %= mod;
                }//if
            }//for(j)
        }//for(k)
    }//for(i)
    return ret;
}//乘法
 
//求幂一般都是正方形矩阵，所以ret = a;
Matrix operator ^ (Matrix a, int b) {
    Matrix ret = a,  tmp = a;
    ret.init_e();
    for ( ; b; b >>= 1) {
        if (b & 1) {
            ret = ret * tmp;
        }
        tmp = tmp * tmp;
    }
    return ret;
}//
 
//递归幂求和
//用二分法求矩阵和,递归实现  
Matrix Power_Sum1(Matrix a, int b) {
    Matrix ret = a;
    ret.init_e();
    if (b == 1) {
        return a;
    } else if (b & 1) {
        return (a ^ b) + Power_Sum1(a, b - 1);
    } else {
        return Power_Sum1(a, b >> 1) * ((a ^ (b >> 1)) + ret);
    }
}//Power_Sum1
 
//非递归幂求和
Matrix Power_Sum2(Matrix a, int b) {
    int k = 0 ,ss[32];
    Matrix tp1, tp2, ret;
    tp1 = tp2 = ret = a;
    ret.init_e();
    while (b) { 
        ss[k++] = b & 1;
        b >>= 1;
    } 
    for (int i = k - 2; i >= 0; i--) {
        tp1 = tp1 * (tp2 + ret);
        tp2 = tp2 * tp2;
        if (ss[i]) {
            tp2 = tp2 * a;
            tp1 = tp1 + tp2;
        }
    }
    return tp1;
}//Power_Sum2
 
int main() {
    Matrix a, b;
    a.init(2, 2);
    b.init(2, 2);
    a.mat[0][0] = a.mat[0][1] = a.mat[1][0] = 1;
    b = a ^ 2;
    b.print();
    b = a + a;
    b.print();
    b = a * a;
    b.print();
}
/*
——————————————
2 1
1 1
 
——————————————
2 2
2 0
 
——————————————
2 1
1 1
 
*/
{% endhighlight %}
ACM中的纯矩阵乘法题目，大致有两类，当然这只是基础的两种类型，去看看HDU 2243，会发现需要用矩阵乘法 + AC自动机两种算法组合求解，这里只探讨最基本的两种：

####1）根据递归式构造矩阵

根据推导出的递推式来构造矩阵，然后利用上边的模板直接计算，这个例子太多，譬如HDOJ的[1757 A Simple Math Problem](http://acm.hdu.edu.cn/showproblem.php?pid=1757)，题目意思很简单，给出俩公式，求f (k) % m，可以很快的得出构造矩阵为：

$$
\begin{vmatrix} a_{0} &a_{1} & a_{2} &a_{3} & a_{4}& a_{5} &a_{6} & a_{7}& a_{8} &a_{9} \\ 1&0 &0 &0 &0 &0 & 0 & 0 & 0 & 0\\ 0& 1& 0& 0& 0 &0 &0 &0 &0 &0 \\ 0& 0& 1& 0& 0& 0& 0& 0& 0& 0\\ 0& 0& 0& 1& 0& 0& 0& 0& 0& 0\\ 0& 0& 0& 0& 1& 0& 0& 0& 0& 0\\ 0& 0& 0& 0& 0& 1& 0& 0& 0& 0 \\ 0& 0& 0& 0& 0& 0& 1& 0& 0& 0\\ 0& 0& 0& 0& 0& 0& 0& 1& 0& 0\\ 0& 0& 0& 0& 0& 0& 0& 0& 1& 0 \end{vmatrix} 
$$

这样乘出来 b 矩阵会不断更新

初始： 9，8，7， 6， 5， 4， 3， 2， 1， 0

一次矩阵乘法后是：f(10),      9，    8，7， 6， 5， 4， 3， 2， 1

两次矩阵乘法后是：f(11),  f(10),     9， 8，7， 6， 5， 4， 3， 2

三次矩阵乘法后是：f(12),  f(11),  f(10),  9， 8，7， 6， 5， 4， 3

代码实现：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int N = 11;
static int mod = 10000;
 
struct Matrix {
    int  mat[N][N]; //修改long long or int
    int n, m;
 
    Matrix() {//初始化
        n = m = N; 
        memset(mat, 0, sizeof(mat));
    }
 
    inline void init(int row, int column) {//初始矩阵大小
        n = row, m = column;
    }
 
    void init_e() {//单位矩阵
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                mat[i][j] = (i == j);
            }
        }
    }//init_e
 
    void print() {//测试检查用
        puts("——————————————"); 
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                printf("%d ", (int)mat[i][j]);
            }puts("");
        }puts("");
    }
 
};
 
//加法
Matrix operator + (Matrix a, Matrix b) {
    Matrix ret; 
    ret.init(a.n, a.m);
    for (int i = 0; i < a.n; i++) {
        for (int j = 0; j < a.m; j++) {
            ret.mat[i][j] = a.mat[i][j] + b.mat[i][j];
            if (ret.mat[i][j] >= mod) {
                ret.mat[i][j] %= mod;
            }
        }
    }
    return ret;
}//加法
 
//n行m列  *  m行p列   =  n行p列
Matrix operator * (Matrix a, Matrix b) {
    Matrix ret; 
    ret.init(a.n, b.m);
    for (int i = 0; i < a.n; i++) {
        for (int k = 0; k < a.m; k++) if (a.mat[i][k]) {
            for (int j = 0; j < b.m; j++) if (b.mat[k][j]) {
                ret.mat[i][j] = ret.mat[i][j] + a.mat[i][k] * b.mat[k][j];  
                if (ret.mat[i][j] >= mod) {
                    ret.mat[i][j] %= mod;
                }//if
            }//for(j)
        }//for(k)
    }//for(i)
    return ret;
}//乘法
 
//求幂一般都是正方形矩阵，所以ret = a;
Matrix operator ^ (Matrix a, int b) {
    Matrix ret = a,  tmp = a;
    ret.init_e();
    for ( ; b; b >>= 1) {
        if (b & 1) {
            ret = ret * tmp;
        }
        tmp = tmp * tmp;
    }
    return ret;
}//
 
//递归幂求和
//用二分法求矩阵和,递归实现  
Matrix Power_Sum1(Matrix a, int b) {
    Matrix ret = a;
    ret.init_e();
    if (b == 1) {
        return a;
    } else if (b & 1) {
        return (a ^ b) + Power_Sum1(a, b - 1);
    } else {
        return Power_Sum1(a, b >> 1) * ((a ^ (b >> 1)) + ret);
    }
}//Power_Sum1
 
//非递归幂求和
Matrix Power_Sum2(Matrix a, int b) {
    int k = 0 ,ss[32];
    Matrix tp1, tp2, ret;
    tp1 = tp2 = ret = a;
    ret.init_e();
    while (b) { 
        ss[k++] = b & 1;
        b >>= 1;
    } 
    for (int i = k - 2; i >= 0; i--) {
        tp1 = tp1 * (tp2 + ret);
        tp2 = tp2 * tp2;
        if (ss[i]) {
            tp2 = tp2 * a;
            tp1 = tp1 + tp2;
        }
    }
    return tp1;
}//Power_Sum2
 
int main() {
    int k, m;
    Matrix b;
    b.init(10, 1);
    for (int i = 0; i < 10; i++) {
        b.mat[i][0] = 9 - i;
    }
    //b.print();
    while (~scanf("%d %d", &k, &m)) {
        mod = m;
        if (k < 10) {
            int tmp;
            for (int i = 0; i < 10; i++) {
                scanf("%d", &tmp);
            }
            printf("%d\n", k % m);
        }
        else {
            Matrix a;
            a.init(10, 10);
            for (int i = 0;  i < 10; i++) {
                scanf("%d", &a.mat[0][i]);
                if(i) {
                    a.mat[i][i - 1] = 1;
                }
            }
            // a.print();
            a = a ^ (k - 9);
            a = a * b;
            printf("%d\n", a.mat[0][0] % m);
        }
    }
    return 0;
}
{% endhighlight %}
####2）有向图中的应用

在有向图中的应用，典型的题目有[HDOJ 2254](http://acm.hdu.edu.cn/showproblem.php?pid=2254) 奥运，它的问法是：在 t1 到 t2 天（包括 t1, t2）内从 v1 到 v2 共有多少种走法？

如果 a 到 b 有路，则map[a][b] = 1，否则map[a][b] = 0，譬如如果1→3，1→4，2→1，2→3，3→4，4→2，4→3有路，则可以构造矩阵：

$$
\begin{vmatrix} 0 & 0&1 &1 \\ 1& 0&1 & 0\\ 0& 0 &0 & 1\\ 0&1 &1 &0 \end{vmatrix}
$$

这样构造好矩阵之后，如果求从 1 到 2 走 5 天（每条路走一天），有几条路，就可以这样计算：

$$
\begin {vmatrix} 0 & 0&1 &1 \\ 1& 0&1 & 0\\ 0& 0 &0 & 1\\ 0&1 &1 &0 \end{vmatrix}^{5}
$$

结果中，第 n 行第 m 列就是 n 到 m 要走 5 天的路数。

此题代码实现：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<map>
#include<vector>
#include<cmath>
#include<algorithm>
using namespace std;
   
const int N = 33;
const int mod = 2008;
 
struct Matrix {
    int  mat[N][N];
    int n, m;
 
    Matrix() {//初始化
        n = m = N; 
        memset(mat, 0, sizeof(mat));
    }
     
    inline void init(int row, int column) {//初始矩阵大小
        n = row, m = column;
    }
 
    void init_e() {//单位矩阵
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                mat[i][j] = (i == j);
            }
        }
    }//init_e
 
    void print() {//测试检查用
        puts("——————————————"); 
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                printf("%d ", (int)mat[i][j]);
            }puts("");
        }puts("");
    }
};
 
//加法
Matrix operator + (Matrix a, Matrix b) {
    Matrix ret; 
    ret.init(a.n, a.m);
    for (int i = 0; i < a.n; i++) {
        for (int j = 0; j < a.m; j++) {
            ret.mat[i][j] = a.mat[i][j] + b.mat[i][j];
            if (ret.mat[i][j] >= mod) {
                ret.mat[i][j] %= mod;
            }
        }
    }
    return ret;
}//加法
 
//n行m列  *  m行p列   =  n行p列
Matrix operator * (Matrix a, Matrix b) {
    Matrix ret; 
    ret.init(a.n, b.m);
    for (int i = 0; i < a.n; i++) {
        for (int k = 0; k < a.m; k++) if (a.mat[i][k]) {
            for (int j = 0; j < b.m; j++) if (b.mat[k][j]) {
                ret.mat[i][j] = ret.mat[i][j] + a.mat[i][k] * b.mat[k][j];  
                if (ret.mat[i][j] >= mod) {
                    ret.mat[i][j] %= mod;
                }//if
            }//for(j)
        }//for(k)
    }//for(i)
    return ret;
}//乘法
 
//求幂一般都是正方形矩阵，所以ret = a;
Matrix operator ^ (Matrix a, int b) {
    Matrix ret = a,  tmp = a;
    ret.init_e();
    for ( ; b; b >>= 1) {
        if (b & 1) {
            ret = ret * tmp;
        }
        tmp = tmp * tmp;
    }
    return ret;
}//
 
//递归幂求和
//用二分法求矩阵和,递归实现  
Matrix Power_Sum1(Matrix a, int b) {
    Matrix ret = a;
    ret.init_e();
    if (b == 1) {
        return a;
    } else if (b & 1) {
        return (a ^ b) + Power_Sum1(a, b - 1);
    } else {
        return Power_Sum1(a, b >> 1) * ((a ^ (b >> 1)) + ret);
    }
}//Power_Sum1
 
//非递归幂求和
Matrix Power_Sum2(Matrix a, int b) {
    int k = 0 ,ss[32];
    Matrix tp1, tp2, ret;
    tp1 = tp2 = ret = a;
    ret.init_e();
    while (b) { 
        ss[k++] = b & 1;
        b >>= 1;
    } 
    for (int i = k - 2; i >= 0; i--) {
        tp1 = tp1 * (tp2 + ret);
        tp2 = tp2 * tp2;
        if (ss[i]) {
            tp2 = tp2 * a;
            tp1 = tp1 + tp2;
        }
    }
    return tp1;
}//Power_Sum2
 
int main(void) {
   int cas, n;
   int v1, v2, t1, t2, x, y;
   while (~scanf("%d", &cas)) {
       map <int, int> mp;//最多只有30条路，所以用map映射
       mp.clear();
 
       Matrix a, b, c;
 
       int cnt = 1;
       for (int i = 0; i < cas; i++) {
           scanf("%d %d", &x, &y);
           if (!mp[x]) {
               mp[x] = cnt++;
           }
           if (!mp[y]) {
               mp[y] = cnt++;
           }
           a.mat[mp[x]][mp[y]] ++;
       }
        
       a.init(cnt, cnt);
      // a.print();
       b.init(cnt, cnt);
       c.init(cnt, cnt);
 
       scanf("%d", &n);
       while (n--) {
           scanf("%d%d%d%d", &v1, &v2, &t1, &t2);     
           x = mp[v1];
           y = mp[v2];
           if (x == 0 || y == 0 || (t1 == 0 && t2 == 0)) {
               puts("0");
               continue;
           }
           int sum;
 
           if (t1 <= 1) {
               b = Power_Sum2(a, t2);
              // b.print();
               sum = b.mat[x][y] % mod;
               printf("%d\n",sum);
               continue;
           }
            b = Power_Sum2(a, t2); 
            c = Power_Sum2(a, t1 - 1);
            sum = (b.mat[x][y] % mod - c.mat[x][y] % mod + mod) % mod;
            printf("%d\n", sum);
       }//while(n)
   }//while(cas)
   return 0;
}
{% endhighlight %}
###二、Strassen算法

Strassen算法核心思想是分治，是一种递归算法，运行时间为$O（n^{lg7}） = O（n^{2.81}）$，当 n 很大时，优化很明显，在普通的矩阵乘法中，C = A * B，按照：

$$
C[i,j] = \sum_{k = 1}^{n}A[i, k] * B[k, j]
$$

每计算一个元素C[i，j]，需要做 n 个乘法和 n – 1 次加法。因此，求出矩阵 C 的 $n^2$ 个元素所需的计算时间为0（$n^3$）。Strassen算法的分治体现在：假设 n 是 2 的幂，将将矩阵A，B和C中每一矩阵都分块成为 4 个大小相等的子矩阵，每个子矩阵都是n / 2 × n / 2的方阵。由此可将方程C = AB重写为：

$$
\begin{vmatrix} C_{0} &C_{1}\\ C_{2} &C_{3} \end{vmatrix} = \begin{vmatrix} A_{0} &A_{1}\\ A_{2} &A_{3} \end{vmatrix} \times \begin{vmatrix} B_{0} &B_{1}\\ B_{2} &B_{3} \end{vmatrix}
$$

由此可得：

$$
C_{0} = A_{0} * B_{0} + A_{1} * B_{2}\\~~~~~ C_{1} = A_{0} * B_{1} + A_{1} * B_{3}\\~~~~~ C_{2} = A_{2} * B_{0} + A_{3} * B_{2}\\~~~~~ C_{3} = A_{2} * B_{1} + A_{3} * B_{3}
$$

可以看出，进行了8次乘法，4次加法，当子矩阵的阶大于2时，为求2个子矩阵的积，可以继续将子矩阵分块，直到子矩阵的阶降为2，利用这个简单的分治策略，最后可以得出：T（n） = 8T（n / 2) + O（$n^2$），但是这个式子的解任然为T（n）= O（$n^3$），和普通的方法效率一样，没有任何提高，原因是上边的四个式子并没有减少矩阵乘法的次数（乘法极其耗费时间，学过底层二进制计算的，必然了解，而加减操作非常轻松），所以改进算法的关键是，减少乘法次数。

Strassen算法的高效之处，就在于，它成功的减少了乘法次数，将8次乘法，减少到7次。不要小看这减少的一次，每递归计算一次，效率就可以提高1 / 8，比如一个大的矩阵递归5次后，（7 / 8）$^5$ = 0.5129，效率提升一倍。不过，这只是理论值，实际情况中，有隐含开销，并不是最常用算法，《算法导论》中给出四条理由：1）隐含的常数因子比简单的O（n3）方法中的常数因子要大。2）矩阵是稀疏矩阵时，为稀疏矩阵设计的方法更快。还有两点已经被缓解，可以不考虑。所以，稠密矩阵上的快速矩阵乘法实现一般采用Strassen算法。

	M1 = (A0 + A3) × (B0 + B3)
	M2 = (A2 + A3) × B0
	M3 = A0 × (B1 – B3)
	M4 = A3 × (B2 – B0)
	M5 = (A0 + A1) × B3
	M6 = (A2 – A0) × (B0 + B1)
	M7 = (A1 – A3) × (B2 + B3)
	C0 = M1 + M4 – M5 + M7
	C1 = M3 + M5
	C2 = M2 + M4
	C3 = M1 – M2 + M3 + M6
	
 求解M1，……，M7总共7次乘法，其他都是加法和减法，比如将C0扩展开后，最后结果是，C0 = A0 * B0 + A1 * B2，《算法导论》里有一句奇怪的话：“现在我们还不清楚Strassen当时是如何找出算法正常运行的关键——子矩阵乘积”，一次乘法的消失过程真的这么吊诡？

所以Strassen算法只需按照上边的11个式子计算即可。实现过程没多少技术含量：

{% highlight c %}
/*
矩阵A为：
——————————————————————————
1 4 9 8
2 5 1 1
5 7 1 2
2 1 8 7
 
矩阵B为:
——————————————————————————
7 0 4 8
4 5 7 1
2 6 4 3
2 6 5 6
 
矩阵A * B = C：
——————————————————————————
57 122 108 87
38 37 52 30
69 53 83 62
48 95 82 83
 
*/
#include <iostream>
#include<cstdio>
#include<cstdlib>
#include<cstring>
#include<algorithm>
#include<cmath>
using namespace std;
 
const int N = 4; 
 
//输出
void print(int n, int C[][N]) {
    puts("——————————————————————————");
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            printf("%d ", C[i][j]);
        }puts("");
    }puts("");
}
 
//加法
void Matrix_Add(int n, int X[][N], int Y[][N], int Z[][N]) {
    for(int i=0; i<n; i++) {
        for(int j=0; j<n; j++) {
            Z[i][j] = X[i][j] + Y[i][j];        
        }        
    }     
}
 
//减法
void Matrix_Sub(int n, int X[][N], int Y[][N], int Z[][N]) {
    for(int i=0; i<n; i++) {
        for(int j=0; j<n; j++) {
            Z[i][j] = X[i][j] - Y[i][j];        
        }        
    }     
}
 
//乘法，A * B = C
void Matrix_Multiply(int A[][N], int B[][N], int C[][N]) {
    for(int i = 0; i < 2; i++) {
        for(int j = 0; j < 2; j++) {
            C[i][j] = 0;      
            for(int t = 0; t < 2; t++) {
                C[i][j] = C[i][j] + A[i][t]*B[t][j];        
            }  
        }        
    }
}
 
/**
参数n指定矩阵A,B,C的阶数，因为随着递归调用Strassen函数
矩阵A,B,C的阶数是递减的N只是预留足够空间而已
*/
void Strassen(int n, int A[][N], int B[][N], int C[][N]) {
    int A11[N][N], A12[N][N], A21[N][N], A22[N][N];
    int B11[N][N], B12[N][N], B21[N][N], B22[N][N];     
    int C11[N][N], C12[N][N], C21[N][N], C22[N][N];
    int M1[N][N], M2[N][N], M3[N][N], M4[N][N], M5[N][N], M6[N][N], M7[N][N];
    int AA[N][N], BB[N][N];
 
    if(n == 2) {   //2-order
        Matrix_Multiply(A, B, C);     
    } else {
        //将矩阵A和B分成阶数相同的四个子矩阵，即分治思想。
        for(int i = 0; i < n / 2; i++) {
            for(int j = 0; j < n / 2; j++) {
                A11[i][j] = A[i][j];
                A12[i][j] = A[i][j + n / 2];
                A21[i][j] = A[i + n / 2][j];
                A22[i][j] = A[i + n / 2][j + n / 2];
 
                B11[i][j] = B[i][j];
                B12[i][j] = B[i][j + n / 2];
                B21[i][j] = B[i + n / 2][j];
                B22[i][j] = B[i + n / 2][j + n / 2];    
            }        
        }  
 
        //M1 = (A0 + A3) × (B0 + B3)
        Matrix_Add(n / 2, A11, A22, AA);
        Matrix_Add(n / 2, B11, B22, BB);
        Strassen(n / 2, AA, BB, M1);
 
        //M2 = (A2 + A3) × B0
        Matrix_Add(n / 2, A21, A22, AA);
        Strassen(n / 2, AA, B11, M2);
 
        //M3 = A0 × (B1 - B3)
        Matrix_Sub(n / 2, B12, B22, BB);
        Strassen(n / 2, A11, BB, M3);
 
        //M4 = A3 × (B2 - B0)
        Matrix_Sub(n / 2, B21, B11, BB);
        Strassen(n / 2, A22, BB, M4);
 
        //M5 = (A0 + A1) × B3
        Matrix_Add(n / 2, A11, A12, AA);
        Strassen(n / 2, AA, B22, M5);
 
        //M6 = (A2 - A0) × (B0 + B1)
        Matrix_Sub(n / 2, A21, A11, AA);
        Matrix_Add(n / 2, B11, B12, BB);
        Strassen(n / 2, AA, BB, M6);
 
        //M7 = (A1 - A3) × (B2 + B3)
        Matrix_Sub(n / 2, A12, A22, AA);
        Matrix_Add(n / 2, B21, B22, BB);
        Strassen(n / 2, AA, BB, M7);
 
        //C0 = M1 + M4 - M5 + M7
        Matrix_Add(n / 2, M1, M4, AA);
        Matrix_Sub(n / 2, M7, M5, BB);
        Matrix_Add(n / 2, AA, BB, C11);
 
        //C1 = M3 + M5
        Matrix_Add(n / 2, M3, M5, C12);
 
        //C2 = M2 + M4
        Matrix_Add(n / 2, M2, M4, C21);
 
        //C3 = M1 - M2 + M3 + M6
        Matrix_Sub(n / 2, M1, M2, AA);
        Matrix_Add(n / 2, M3, M6, BB);
        Matrix_Add(n / 2, AA, BB, C22);
 
        for(int i = 0; i < n / 2; i++) {
            for(int j = 0; j < n / 2; j++) {
                C[i][j] = C11[i][j];
                C[i][j + n / 2] = C12[i][j];
                C[i + n / 2][j] = C21[i][j];
                C[i + n / 2][j + n / 2] = C22[i][j];        
            }        
        }
    }
}
 
int main() {
    int A[N][N],B[N][N],C[N][N];    
    //对A和B矩阵赋值，下边赋值测试用
    for(int i = 0; i < N; i++) {
        for(int j = 0; j < N; j++) {
            A[i][j] = rand() % 10;
            B[i][j] = rand() % 10;   
        }        
    }
 
    Strassen(N, A, B, C);
    puts("矩阵A为：");
    print(N, A);
    puts("矩阵B为: ");
    print(N, B);
    puts("矩阵A * B = C：");
    print(N, C);
    return 0;
}
{% endhighlight %}
关于Strassen算法，算法导论讲的不够透彻，参考过以下资料：

《[Strassen矩阵算法分析及其实现](http://riddickbryant.iteye.com/blog/546463)》，WiKi里的《[施特拉森算法](http://zh.wikipedia.org/wiki/Strassen%E6%BC%94%E7%AE%97%E6%B3%95)》，《[Strassen矩阵乘法](http://218.22.18.86/info/Data_Structures_and_Algorithms/algorithm/commonalg/misc/strassen/strassen.htm#)》
