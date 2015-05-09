---
title: 树状数组小结
author: Wei Li
excerpt: 树状数组是一个查询和修改复杂度都为log(n)的数据结构，假设数组a[1..n]，那么查询a[1]+...+a[n]的时间是log级别的，而且是一个在线的数据结构，支持随时修改某个元素的值，复杂度也为log级别。 
meta_description: 树状数组是一个查询和修改复杂度都为log(n)的数据结构，假设数组a[1..n]，那么查询a[1]+…+a[n]的时间是log级别的，而且是一个在线的数据结构，支持随时修改某个元素的值，复杂度也为log级别。
layout: post
permalink: /2011/07/10/binary-indexed-trees/
views:
  - 10646
categories:
  - 算法学习
tags:
  - ACM
  - 树状数组
  - 算法
---
最近做了HDU几道树状数组的题，小小总结一下……  
树状数组是一个查询和修改复杂度都为log(n)的数据结构，假设数组a[1..n]，那么查询a[1]+…+a[n]的时间是log级别的，而且是一个在线的数据结构，支持随时修改某个元素的值，复杂度也为log级别。 

Cn = A(n – 2^k + 1) + … + An ；

此处的k为n的二进制形式末尾0的个数，可以利用Lowbit()函数找出k

注意：下标不能从0开始：

当i = 0，0 + Lowbit(0) = 0，会造成死循环!

主要三个函数：

{% highlight c %}
int Lowbit(int x) {
    return x & (-x);
}

void Update(int pos, int val) {//更新之后所有包含a[pos]的tree[]
    while (pos <= n) {
        tree[pos] += val;
        pos += Lowbit(pos);
    }
}

int Query(int x) {// 求一般数组a[1]到a[x]的和  
  int sum = 0;
    while (x > 0) {
        sum += tree[x];
        x -= Lowbit(x);
    }
    return sum;
}
{% endhighlight %}


树状数组功能：

1. 单点更新，区间求和（第一类）//HDU 1166 敌兵布阵
2. 区间更新，单点求值（第二类）//HDU 1556 Color the ball
3. 逆序数的应用，可以求左边小于它的数，右边大于它的数，反过来可以求

题目：HDU 1166 敌兵布阵 ，树状数组模板题，单点更新，区间求和
{% highlight c %}
include<iostream>
include<cstdio>
include<cstring>
include<string>
include<cmath>
include<algorithm>
using namespace std;
const int maxn = 50001;

int kp[maxn];
int tree[maxn];
int n;

inline int Lowbit(int x) {
    return x & (-x);
}

inline void Update(int pos, int val) {
    while (pos <= n) {
        tree[pos] += val;
        pos += Lowbit(pos);
    }
}

inline int Query(int x) {
    int sum = 0;
    while (x > 0) {
        sum += tree[x];
        x -= Lowbit(x);
    }
    return sum;
}

int main() {
    int cas, cases = 1;
    scanf("%d", &cas);
    while(cas--) {
        memset(tree, 0, sizeof(tree));
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            scanf("%d", &kp[i]);
            Update(i, kp[i]);
        }
        char str[15];
        printf("Case %d:\n", cases++);
        while (scanf("%s", str), strcmp(str, "End")) {
            int a, b;
            scanf("%d%d", &a, &b);
            switch(str[0]) {
            case 'Q' : 
                printf("%d\n", Query(b) - Query(a - 1));
                break;
            case 'A' :
                Update(a, b);
                break;
            case 'S' :
                Update(a, -b);
                break;
            }
		}//while
	}//while(cas)
	return 0;
}
{% endhighlight %}

HDU 1556 Color the ball 区间更新，单点求值
{% highlight c %}
//区间更新，单点求值
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 100001;
 
int tree[maxn];
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int pos, int val) {
    while (pos <= n) {
        tree[pos] += val;
        pos += Lowbit(pos);
    }
}
 
inline int Query(int x) {
    int sum = 0;
    while (x > 0) {
        sum += tree[x];
        x -= Lowbit(x);
    }
    return sum;
}
 
int main() {
    while (~scanf("%d", &n), n) {
        memset(tree, 0, sizeof(tree));
 
        for (int i = 0; i < n; i++) {
            int a, b;
            scanf("%d%d", &a, &b);
            Update(b + 1, -1);
            Update(a, 1);
        }
 
        for (int i = 1; i < n; i++) {
            printf("%d ", Query(i));
        }
        printf("%d\n", Query(n));
    }
    return 0;
}
{% endhighlight %}

HDU 1541 Stars  树状数组应用
{% highlight c %} 
/*
题目说明是先按x，从小到大，y从小到，左下往右上推进
要求每个等级的星星数量，每输入一个星星，只要看它前边有几个
星星就可以知道此星星等级，记录之，然后把这个等级的星星加一
树状数组：
更新可用于，增加数量，如+1
然后查询时，查到的就是它前边的数量
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 35000;
 
int tree[maxn];
int level[maxn];
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int pos) {
    while (pos <= maxn) {
        tree[pos] ++;
        pos += Lowbit(pos);
    }
}
 
inline int Query(int x) {
    int sum = 0;
    while (x > 0) {
        sum += tree[x];
        x -= Lowbit(x);
    }
    return sum;
}
 
 
int main() {
    while (~scanf("%d", &n)) {
        memset(tree, 0, sizeof(tree));
        memset(level, 0, sizeof(level));
        for (int i = 0; i < n; i++) {
            int x, y;
            scanf("%d%d", &x, &y);
            level[Query(++x)]++;
            Update(x);
        }
        for (int i = 0; i < n; i++) {
            printf("%d\n", level[i]);
        }
 
    }
    return 0;
}
{% endhighlight %}

HDU 1892 See you~   简单二维树状数组
{% highlight c %} 
/*
简单二维树状数组
1、如果删除或移动时，如果（x1,y1）点书少于要求的数量，
则有多少操作多少，不能出现负数
2、计算输出时，x1,x2,y1,y2大小未给出，需要判断 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 1002;
int tree[maxn + 2][maxn + 2];
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int y, int v) {
    for (int i = x; i <= maxn; i += Lowbit(i)) {
        for (int j = y; j <= maxn; j += Lowbit(j)) {
                tree[i][j] += v;
        }
    }
}
 
inline int Query(int x, int y) {
    int sum = 0;
    for (int i = x; i > 0; i -= Lowbit(i)) {
        for (int j = y; j > 0; j -= Lowbit(j)) {
            sum += tree[i][j];
        }
    }
    return sum;
}
 
inline int getv(int x, int y) {
    return Query(x, y) - Query(x - 1, y) - Query(x, y - 1) + Query(x - 1, y - 1);
}
 
int main() {
    int cas, cases = 1;
    scanf("%d", &cas);
    while (cas--) {
        printf("Case %d:\n", cases++);
        memset(tree, 0, sizeof(tree));
        for (int i = 1; i <= maxn; i++) {
            for (int j = 1; j <= maxn; j++) {
                Update(i, j, 1);
            }
        }
        scanf("%d", &n);
        while (n--) {
            char str[2];
            int x1, x2, y1, y2, v;
            scanf("%s", str);
            if(str[0]=='A'){
                scanf("%d %d %d", &x1, &y1, &v);
                Update(x1 + 1, y1 + 1, v);
            } else if (str[0] == 'S'){
                scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
                if(x1 > x2) swap(x1, x2);
                if(y1 > y2) swap(y1, y2);
                int res = Query(x2 + 1, y2 + 1) - Query(x1, y2 + 1)
                    - Query(x2 + 1, y1) + Query(x1, y1);
                printf("%d\n",res);
            } else if (str[0] == 'D'){
                scanf("%d%d%d", &x1, &y1, &v);
                int v1 = getv(x1 + 1, y1 + 1);
                Update(x1 + 1, y1 + 1, -min(v,v1));
            } else if (str[0] == 'M'){
                scanf("%d%d%d%d%d", &x1, &y1, &x2, &y2, &v);
                int v1 = getv(x1 + 1, y1 + 1);
                v1 = min(v1, v);
                Update(x1 + 1, y1 + 1, -v1);
                Update(x2 + 1, y2 + 1, v1);
            }
        }//while (n)
    }//whle(cas)
    return 0;
}
{% endhighlight %}

HDU 2642 Stars 二维树状数组
{% highlight c %} 
/*
可能重复跟新，所以需要设一个mark数组
二维的树状数组：
B星星亮
D星星灭
Q查询亮星星数量
tree[x][y], x, y分别为坐标
二维，Update， Query,见代码
Update(int x, int y, int num)
当num > 0时， 相当于加
当num < 0时， 相当于减
这样写方便
另外坐标是从0开始的，要+1, 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 1010;
int tree[maxn][maxn];
bool mark[maxn][maxn];
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int y, int num) {
    for(int i = x; i <= maxn; i+= Lowbit(i)) {
        for(int j = y; j <= maxn; j += Lowbit(j)) {
            tree[i][j] += num;
        }
    }
}
 
 
inline int Query(int x, int y) {
    int sum = 0;
    for(int i = x; i > 0; i -= Lowbit(i)) {
        for(int j = y; j > 0; j -= Lowbit(j))
            sum += tree[i][j];
    }
    return sum;
}
 
int  main() {
    char str[3];
    int n, x, y;
    scanf("%d", &n);
    cin.ignore();
    while(n--) {
        scanf("%s", str);
        if(str[0] == 'B')  {
            scanf("%d%d", &x, &y);
            x++; y++;
            if(mark[x][y] == false) {
                mark[x][y] = true;
                Update(x, y, 1);
            }
        } else if(str[0] == 'D') {
            scanf("%d%d", &x, &y);
            x++;
            y++;
            if(mark[x][y] == true) {
                mark[x][y] = false;
                Update(x, y, -1);
            }
        } else if(str[0] == 'Q')  {
            int x1, y1, x2, y2;
            scanf("%d%d%d%d", &x1, &x2, &y1, &y2);
            x1++, x2++, y1++, y2++;
            if (x1 < x2) swap(x1, x2);
            if (y1 < y2) swap(y1, y2);
            int res = Query(x1, y1) - Query(x1, y2 - 1) - Query(x2 - 1, y1) + Query(x2 - 1, y2 - 1);
            printf("%d\n", res);
        }
    }//while
    return 0;
} 
{% endhighlight %}

HDU 3584 Cube 三维树状数组，只有0，1的区间更新，单点求值
{% highlight c %} 
/*
三维树状数组，只有0，1的区间更新，单点求值
具体讲解查看:
国家集训队2009论文集 武森《浅谈信息学竞赛中的“0”和“1”》
里边这题为例题，有讲解
一个区间(x,y)更新，只需在x和y+1出加1，然后Query(x)%2就是答案
前一个1是增加，后一个1，抵消掉，即变成0，所以查询出来就是单点的结果 
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 105;
int tree[maxn][maxn][maxn];
int n, m;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int y, int z) {
    for (int i = x; i <= n; i += Lowbit(i)) {
        for (int j = y; j <= n; j += Lowbit(j)) {
            for (int k = z; k <= n; k += Lowbit(k)) {
                tree[i][j][k] ++;
            }
        }
    }//for(i)
}
 
inline int Query(int x, int y, int z) {
    int sum = 0;
    for (int i = x; i > 0; i -= Lowbit(i)) {
        for (int j = y; j > 0; j -= Lowbit(j)) {
            for (int k = z; k > 0; k -= Lowbit(k)) {
                    sum += tree[i][j][k];
            }
        }
    }//for(i)
    return sum;
}
 
int main() {
    while (~scanf("%d%d", &n, &m)) {
        int sign, x1, y1, z1, x2, y2, z2;
        memset(tree, 0, sizeof(tree));
        while (m--) {
            scanf("%d", &sign);
            if (sign == 0) {
                scanf("%d%d%d", &x1, &y1, &z1);
                printf("%d\n", Query(x1, y1, z1) % 2);
            } else {
                scanf("%d%d%d%d%d%d", &x1, &y1, &z1, &x2, &y2, &z2);
                Update(x1 ,y1, z1);
                Update(x1, y1, z2 + 1);
                Update(x1, y2 + 1, z1);
                Update(x1, y2 + 1, z2 + 1);
                Update(x2 + 1, y1, z1);
                Update(x2 + 1, y1, z2 + 1);
                Update(x2 + 1, y2 + 1, z1);
                Update(x2 + 1, y2 + 1, z2 + 1);
            }
        }
    }
    return 0;
} 
{% endhighlight %}

HDU  2689 Sort it 求逆序对
{% highlight c %} 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
#define MAXN 1001
 
int tree[MAXN];
 
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int v) {
    while (x <= n) {
        tree[x] += v;
        x += Lowbit(x);
    }
}
 
inline int Query(int x) {
    int ans = 0;
    while (x > 0) {
        ans += tree[x];
        x -= Lowbit(x);
    }
    return ans;
}
 
 
int main() {
    while (~scanf("%d", &n)) {
        memset(tree, 0, sizeof(tree));
        int res = 0;
        int a;
        for (int i = 1; i <= n; i++) {
            scanf("%d", &a);
            Update(a, 1);
            res += i - Query(a);
        }
        printf("%d\n", res);
    }
    return 0;
}
{% endhighlight %}

HDU 2838 Cow Sorting 逆序数的应用
{% highlight c %} 
/*
对应之前那道PKU的Cow Sorting
HDU这题要求相邻的才能交换 （necessarily adjacent）
PKU那道不必相邻（not necessarily adjacent）
解题步骤：
1：找出第i个数之前比它大的个数
2：计算第i个数之前比它大的数的和
用树状数组可以很巧妙的求出逆序对
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 100001;
 
struct node {
    int cnt;
    long long sum;
}tree[maxn];
 
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int v, int cnt) {
    while (x <= n) {
        tree[x].sum += v;
        tree[x].cnt += cnt;
        x += Lowbit(x);
    }
}
 
inline int Query_cnt(int x) {//返回的是a前边比a小的个数
    int ans = 0;
    while (x > 0) {
        ans += tree[x].cnt;
        x -= Lowbit(x);
    }
    return ans;
}
 
inline long long Query_sum(int x) {//返回x前所有数的和
    long long ans = 0;
    while (x > 0) {
        ans += tree[x].sum;
        x -= Lowbit(x);
    }
    return ans;
}
 
int main() {
    while (~scanf("%d", &n)) {
        long long ans = 0;
        memset(tree, 0, sizeof(tree));
 
        for (int i = 1; i <= n; i++) {
            int a;
            scanf("%d", &a);
            Update(a, a, 1);
            long long k1 = i - Query_cnt(a);//k1是逆序对数
            if (k1 != 0) {
                //所有前n个数的和 - a前边数的和，即i前边所有比a大的和
                long long k2 = Query_sum(n) - Query_sum(a);
                ans += k1 * a + k2;
            }
        }//for(i)
        printf("%I64d\n", ans);
    }//while 
    return 0;
}
{% endhighlight %}

HDU 1394 Minimum Inversion Number 逆序对应用
{% highlight c %} 
//树状数组求逆序对
//同时，循环移位时，增加一定逆序对数，减少一定逆序对数
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 5001;
 
int tree[maxn];
int kp[maxn];
 
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int v) {
    while (x <= n) {
        tree[x] += v;
        x += Lowbit(x);
    }
}
 
inline int Query(int x) {//返回的是a前边比a小的个数
    int ans = 0;
    while (x > 0) {
        ans += tree[x];
        x -= Lowbit(x);
    }
    return ans;
}
 
int main() {
    while (~scanf("%d", &n)) {
        memset(tree, 0, sizeof(tree));
        int minn = 0;
        for (int i = 1; i <= n; i++) {
            scanf("%d", &kp[i]);
            kp[i]++;
            Update(kp[i], 1);
            minn += i - Query(kp[i]);
        }
 
        int res = minn;
        for (int i = n; i >= 2; i--) {
            //res += kp[i] - 1 - (n - kp[i]);
            //如果以n开头，会增加kp[i] - 1个逆序数，减少n - kp[i]个逆序数
            res += 2 * kp[i] - n - 1;
            //printf("i : %d, mm : %d\n", i, res);
            if (res < minn) {
                minn = res;
            }
        }
        printf("%d\n", minn);
    }
    return 0;
}
{% endhighlight %}

HDU 2688 Rotate 求正序数，然后模拟
{% highlight c %} 
/*
先求正序数
然后每次模拟，最后得出结果
注意求逆序数是：
res += I - Query(kp[i]);
而求正序数是：
先Update(kp[i])；然后 
res += Query(kp[i] - 1);
因为Q询问的是小于等于自己的数，必须排除掉和自己相等的数 
 
注意不是：
1、Query(kp[i]) - 1
因为可能kp[i] 有多个数，不能-1
 2、也不能是Query(kp[i - 1]);
 因为kp[i - 1]不确定，如果是比kp[i]大的数，不可以  
 3、也不可以先
 Query(kp[i])
 再Update(kp[i]);
 这种写法如果没有重复数是可以的，但是如果有重复数据，就不可以了 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 3000005;
const int maxm = 10001;
int tree[maxm];
int kp[maxn];
 
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int v) {
    while (x <= maxm - 1) {
        tree[x] += v;
        x += Lowbit(x);
    }
}
 
inline int Query(int x) {
    int ans = 0;
    while (x > 0) {
        ans += tree[x];
        x -= Lowbit(x);
    }
    return ans;
}
 
int main() {
    int m;
    while (~scanf("%d", &n)) {
        memset(tree, 0, sizeof(tree));
        long long res = 0;
        for (int i = 1; i <= n; i++) {
            scanf("%d", &kp[i]);
            Update(kp[i], 1);
            res += Query(kp[i] - 1);
        }
        scanf("%d", &m);
        char str[2];
        while (m--) {
            scanf("%s", str);
            if (str[0] == 'Q') {
                printf("%I64d\n", res);
            } else {
                int a, b;
                scanf("%d%d", &a, &b);
                a++, b++;
                int tmp = kp[a];
                for (int i = a; i < b; i++) {
                    kp[i] = kp[i + 1];
                    if (kp[i] > tmp) {
                        res--;
                    } else if (kp[i] < tmp) {
                        res++;
                    }
                }//for(i)
                kp[b] = tmp;
            }
        }//while (m)
    }//while(read);
    return 0;
}
{% endhighlight %}

HDU 2852 KiKi’s K-Number 求大于a的第k大元素，可以用二分枚举
{% highlight c %} 
/*
插入，删除即，Update(a, 1), Update(a, -1)
求大于a的第k大元素，可以用二分枚举
设答案在树状数组中的位置为m, 然后kp[a +１］……kp[m]求和
就是小于等于m的个数，即大于a的第m个数 
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 100001;
 
int tree[maxn];
int kp[maxn];
 
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int val) {
    while (x <= maxn - 1) {
        tree[x] += val;
        x += Lowbit(x);
    }
}
 
inline int Query(int x) {
    int ans = 0;
    while (x > 0) {
        ans += tree[x];
        x -= Lowbit(x);
    }
    return ans;
}
 
 
inline int Bin(int a, int k) {
    int l = a + 1;
    int r = maxn - 1;
    int ans = maxn;
    int aS = Query(a);
     
    while (l <= r) {
        int mid = (l + r) >> 1;
        int nS = Query(mid);
        if (nS - aS >= k) {
            r = mid - 1;
            if (mid < ans) {
                ans = mid;
            }
        } else {
            l = mid + 1;
        }
    }
    return ans;
}
 
int main() {
    int n;
    while (~scanf("%d", &n)) {
        memset(tree, 0, sizeof(tree));
        while (n--) {
            int sign, e, a, k;
            scanf("%d", &sign);
            if (sign == 0) {
                scanf("%d", &e);
                Update(e, 1);
            } else if (sign == 1) {
                scanf("%d", &e);
                if (Query(e) - Query(e - 1) == 0) {
                    puts("No Elment!");
                } else {
                    Update(e, -1);
                }
            } else {
                scanf("%d%d", &a, &k);
                int num = Bin(a, k);
                if (num == maxn) {
                    puts("Not Find!");
                } else {
                    printf("%d\n", num);
                }
            }
        }//while(n--) 
    }//while(scanf(n)
    return 0;
}  
{% endhighlight %}

HDU 2227 Find the nondecreasing subsequences 利用树状数组求递增序列个数
{% highlight c %} 
/*
dp[i] = sum{dp[j], j < i,a[j] <= a[i],}
 dp[i]表示到第i个位置能够找到的非递减序列的解的数量
 因为数字太大，所以需要离散化处理，映射到下标
 初始化都dp[]都为0，而所求的是非递减数列，需要把只有自己的情况加进去 
 所以dp[1] = 0 + 1;
 dp[2] = dp[1] + 1; 
 dp[3] = dp[1] + dp[2] + 1;
 do[4] = dp[1] + dp[2] + dp[3] + 1;
 */
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
#define MOD 1000000007
 
const int maxn = 100005;
int n;
int tree[maxn];
int pos[maxn];
int kp[maxn];
int tot;
 
inline int lowbit (int x) {
    return x & (-x);
}
 
inline void Update (int x , int val ) {
    while(x <= tot) {
        tree[x] += val;
        if (tree[x] >= MOD) {
            tree[x] %= MOD;
        }
        x += lowbit(x);
    }
}
 
inline long long Query (int x ) {
    long long sum = 0;
    while (x > 0) {
        sum += tree[x];
        if (sum >= MOD) {
            sum %= MOD;
        }
        x -= lowbit(x);
    }
    return sum;
}
 
 
inline int Bin (int x ) {
    int l = 1 , r = tot;
    while (l <= r ) {
        int mid = (l + r) >> 1;
        if (pos[mid] == x) return mid;
        if (pos[mid] > x) r = mid - 1;
        else l = mid + 1;
    }
    return -1;
}
 
int main() {
    while (~scanf("%d", &n)) {     
        for (int i = 1 ; i <= n ; i++ ) {
            scanf("%d", &kp[i]);
            pos[i] = kp[i];
        }
 
        tot = 0;
        sort(pos + 1, pos + n + 1);
        for( int i = 1 ; i <= n ; i++ ) {
            if( i == 1 || pos[i] != pos[i - 1] ) {
                pos[++tot] = pos[i];
                tree[tot] = 0;
            }
        }
 
        long long res = 0;
        for( int i = 1 ; i <= n ; i++ ) {
            int id = Bin(kp[i]);
 
            long long tp = Query(id);
            res += tp + 1; 
            if (res >= MOD) {
                 res %= MOD;
            }
            Update(id, tp + 1); 
        }
        printf("%I64d\n", res);
    }
    return 0;
}
{% endhighlight %}

HDU 3450 Counting Sequence 类似2227
{% highlight c %} 
/* 
HDU 2836和这题竟然一模一样，晕死
和HDU2227类似，同样DP + 离散化 + 树状数组
abs(a[j] - a[i]) <= H
H - a[i] <= a[j] <= H + a[i]
找到[H - a[i], H + a[i]]范围的个数用树状数组累加即可
另外，题目要求k >= 2, 意思是序列长度要大于等于2，所以
要减去只包含自己的序列，即n 
 
data:
4 2
1 3 7 5
res:
4
_______
process:
1 3
3 5
5 7
1 3 5
______
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
#define MOD 9901
 
 
const int maxn = 100005;
int kp[maxn], pos[maxn], tree[maxn];
int n, tot;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x, int val) {
    while (x <= tot) {
        tree[x] += val;
        if (tree[x] >= MOD) {
            tree[x] %= MOD;
        }
        x += Lowbit(x);
    }
}
 
inline int Query(int x) {
    int sum = 0;
    while (x > 0) {
        sum += tree[x];
        if (sum >= MOD) {
            sum %= MOD;
        }
        x -= Lowbit(x);
    }
    return sum;
}
 
inline int Bin(int x) {
    int l = 1; 
    int r = tot;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (pos[mid] == x) return mid;
        if (pos[mid] < x) l = mid + 1;
        else r = mid - 1;
    }
    return -1;
}
 
inline int Binleft(int x) {//找到最小的  >= a[i] - H  的数
    int l = 1;
    int r = tot;
    int ans = 0;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (pos[mid] >= x) {
            r = mid - 1;
            ans = mid;
        } else {
            l = mid + 1;
        }
    }
    return ans;
}
 
inline int Binright(int x) {//找到最大的  <= a[i] + H 的数
    int l = 1;
    int r = tot;
    int ans = tot;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (pos[mid] <= x) {
            l = mid + 1;
            ans = mid;
        } else {
            r = mid - 1;
        }
    }
    return ans;
}
 
int main() {
    int H;
    while (~scanf("%d", &n)) {
        scanf("%d", &H);
        for (int i = 1; i <= n; i++) {
            scanf("%d", &kp[i]);
            pos[i] = kp[i];
        }
        sort(pos + 1, pos + n + 1);
 
        tot = 0;
        for (int i =1; i <= n; i++) {
            if (pos[i] != pos[i - 1] || i == 1) {
                pos[++tot] = pos[i];
                tree[tot] = 0;
            }
        }
 
        int res = 0;
        for (int i = 1; i <= n; i++) {
            int id = Bin(kp[i]);
            int l = Binleft(kp[i] - H);
            int r = Binright(kp[i] + H);//left__right
 
            int num = Query(r) - Query(l - 1);
            if (num < 0) num += MOD;
            if (num >= MOD) num %= MOD;//num->[0, MOD]
 
            res += num + 1;
            if (res >= MOD) res %= MOD;
            Update(id, num + 1);
        }
      //题目要求k >= 2，算法是按k >= 1计算所以完了要减去n
            printf("%d\n", ((res - n) % MOD + MOD) % MOD);
    }
    return 0;
}

{% endhighlight %}

HDU 2492 Ping Pong 求长度为3的顺序序列有多少个
{% highlight c %} 
/*
求长度为3的顺序序列有多少个，
每次取出一个数，以这个数为第二个数的序列个数就是：
左边比他小的个数乘以右边比他大的个数   ，加上，
左边比他大的个数乘以右边比他小的个数
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int maxn = 100001;
 
int tree[maxn];
int lmin[maxn], lmax[maxn], rmin[maxn], rmax[maxn]; 
int kp[maxn];
 
int n;
 
inline int Lowbit(int x) {
    return x & (-x);
}
 
inline void Update(int x) {
    while (x <= maxn - 1) {
        tree[x] ++;
        x += Lowbit(x);
    }
}
 
inline int Query(int x) {//返回的是a前边比a小的个数
    int ans = 0;
    while (x > 0) {
        ans += tree[x];
        x -= Lowbit(x);
    }
    return ans;
}
 
 
int main() {
    int cas;
    scanf("%d", &cas);
    while (cas--) {
        scanf("%d", &n);
        for (int i = 1; i<= n; i++) {
            scanf("%d", &kp[i]);
        }
        memset(tree, 0, sizeof(tree));
 
        for (int i = 1; i <= n; i++) {
            Update(kp[i]);
            lmin[i] = Query(kp[i] - 1);
            lmax[i] = i - lmin[i] - 1;
        }
 
        memset(tree, 0, sizeof(tree));
         
        for (int i = n, j = 1; i >= 1; i--, j++) {
            Update(kp[i]);
            rmin[i] = Query(kp[i] - 1);
            rmax[i] = j - rmin[i] - 1;
        }
       /*
       这题下边这种写法也是可以的，因为题目给出，每个等级都是独一无二的 
        memset(tree, 0, sizeof(tree));
        for (int i = 1; i <= n; i++) {
            lmin[i] = Query(kp[i]);
            lmax[i] = i - lmin[i] - 1;
            Update(kp[i]);
        }
        memset(tree, 0, sizeof(tree));
        for (int i = n, j = 1; i >= 1; i--, j++) {
            rmin[i] = Query(kp[i]);
            rmax[i] = j - rmin[i] - 1;
            Update(kp[i]);
        }
        */
        long long res = 0;
        for (int i = 1; i <= n; i++) {
            res += lmin[i] * rmax[i] + lmax[i] * rmin[i];
        }
        printf("%I64d\n", res);
    }
    return 0;
}
{% endhighlight %}



