---
title: 计算几何学(Computational Geometry)
author: Wei Li
excerpt: 有 n 个点的集合 Q，找出最近的两点，这一问题可以应用于交通控制等系统。在空中或海洋交通控制系统中，需要发现两个距离最近的交通工具，以便检测出可能发生的相撞事故。
layout: post
permalink: /2011/11/27/computational-geometry/
views:
  - 4775
categories:
  - 算法学习
tags:
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
###一、线段算法基础

首先要讨论的是三个基础问题：

1. 已知两条有向线段p0p1，和p0p2，相对于它们的公共端点p0来说，p0p1是否在p0p2的顺时针方向上？

2. 已知两条线段p0p1和p1p2，如果先通过p0p1再通过p1p2，在点p1处是不是要向左旋？

3. 线段p1p2和p3p4是否相交？

####1）叉积(Cross Product)
![Image][1]
叉积是线段算法的中心，图 a) 两个向量 p1，p2，可以把 p1 ×p2看作是由点（0, 0），p1，p2 和 p1 + p2 = (x1 + x2，y1 + y2) 所形成平行四边形的面积，另一种等价定义是把叉积定义为一个矩阵的行列式：

$$
p1\times p2 = \begin{vmatrix} x1 &x2 \\ y1 &y2 \end{vmatrix} = x1y2 - x2y = -p2\times p1
$$

如果 p1×p2 为正数，则相对于原点来说，p1 在 p2 的顺时针方向上；如果 p1×p2 为负数，则 p1 在 p2 的逆时针方向上。如图b) 浅阴影区域包含了 p 的顺时针方向上的向量。深阴影区包含了 p 的逆时针方向上的向量。

对于上边提到的第一个问题，解决方法是，将p0当做原点即可，计算叉积：

$$
(p1 - p0) \times (p2 - p0) = (x1 - x0)(y2-y0) - (x2-x0)(y1-y0)
$$

如果叉积为政，则 p0p1 在 p0p2 的顺时针方向上，如果为负，则 p0p1 在 p0p2 的逆时针方向上。

####2）确定线段旋转方向
![Image][2]

第二个问题，如果知道叉积，那再简单不过了，相对于有向线段 p0p1，检查有向线段 p0p2 是在其顺时针方向还是逆时针方向即可，只需计算叉积（p2 – p0）× （p1 – p0）,如果叉积为负，则 p0p2 在 p0p1 逆时针方向，因此左旋，反之，同理。

####3）判断两个线段是否相交

第三个问题，以下两个条件，至少满足一个，即可断定两线段相交：

1. 每个线段都跨越（straddle）包含了另一线段的直线（即最普通的相交）

2. 一个线段的某一个端点位于另一线段上（边界情况，包含重合）

SEGMENTS-INTERSECT 用于判断是否相交，DIRECTION利用叉积方法，计算线段方位；ON-SEGMENT确定一个与某一线段共线的点是否位于该线段上。伪代码：

	SEGMENTS-INTERSECT(p1, p2, p3, p4)
	1  d1 ← DIRECTION(p3, p4, p1)
	2  d2 ← DIRECTION(p3, p4, p2)
	3  d3 ← DIRECTION(p1, p2, p3)
	4  d4 ← DIRECTION(p1, p2, p4)
	5  if ((d1 > 0 and d2 < 0) or (d1 < 0 and d2 > 0)) and
	             ((d3 > 0 and d4 < 0) or (d3 < 0 and d4 > 0))
	6     then return TRUE
	7  elseif d1 = 0 and ON-SEGMENT(p3, p4, p1)
	8     then return TRUE
	9  elseif d2 = 0 and ON-SEGMENT(p3, p4, p2)
	10     then return TRUE
	11  elseif d3 = 0 and ON-SEGMENT(p1, p2, p3)
	12     then return TRUE
	13  elseif d4 = 0 and ON-SEGMENT(p1, p2, p4)
	14     then return TRUE
	15  else return FALSE

	DIRECTION(pi, pj, pk)
	1  return (pk – pi) × (pj – pi)

	ON-SEGMENT(pi, pj, pk)
	1  if min(xi, xj) ≤ xk ≤ max(xi, xj) and min(yi, yj) ≤ yk ≤ max(yi, yj)
	2     then return TRUE
	3     else return FALSE

伪代码很容易看懂，判断过程，有四种情况需要选择，a）相交，叉积异号；b）不想交，叉积同号；c）p3和 p1，p2 共线，4）p3 和 p1,p2共线，但是不相交。（最容易忽略就是第四种情况）
![Image][3]

代码实现：
{% highlight c %}
//新写一个，这个代码可以顺便AC掉 HDU 1086
#include<iostream>
#include<cstdio>
#include<cmath>
#include<algorithm>
using namespace std;

struct Point {
	double x, y;
};

double Direction(Point p1, Point p2, Point p0 ) {
	return (p1.x - p0.x) * (p2.y - p0.y) - (p2.x - p0.x) * (p1.y - p0.y);
}

bool OnSegment( Point p1, Point p2, Point p0) {
    if (min(p1.x, p2.x) <= p0.x && p0.x <= max(p1.x, p2.x) &&
        min(p1.y, p2.y) <= p0.y && p0.y <= max(p1.y, p2.y)) {
            return true;
    } else {
        return false;
    }
}

bool SegmentsIntersert(Point p1, Point p2, Point p3, Point p4) {
    double d1 = Direction(p3, p4, p1);
    double d2 = Direction(p3, p4, p2);
    double d3 = Direction(p1, p2, p3);
	double d4 = Direction(p1, p2, p4);

	if (d1 * d2 < 0 && d3 * d4 < 0) {
		return true;
    } else if (d1 == 0 && OnSegment (p3, p4, p1)) {
		return true;
    } else if (d2 == 0 && OnSegment (p3, p4, p2)) {
		return true;
    } else if (d3 == 0 && OnSegment (p1, p2, p3)) {
		return true;
    } else if (d4 == 0 && OnSegment (p1, p2, p4)) {
		return false;
    }
	return 0;
}

int main() {
    int n;
    Point a[105], b[105];
    while (~scanf("%d", &n), n) {
        int cnt = 0;
        for (int i = 0; i < n; i++) {
            scanf("%lf %lf %lf %lf", &a[i].x, &a[i].y, &b[i].x, &b[i].y);
        }
  		for (int i = 0; i < n; i++) {
			for (int j = i + 1; j < n; j++) {
					if (SegmentsIntersert (a[i], b[i], a[j], b[j])) {
						cnt++;
                    }
            }
        }
		printf("%d\n", cnt);
	}
	return 0;
}
{% endhighlight %}

###二、寻找凸包(Convex Hull)

点集Q的凸包是一个最小的凸多边形P，可以把Q中的每个点都想象成是露在一块板外的铁钉，那么凸包就是包围了所有这些铁钉的一条拉紧了的橡皮绳所构成的形状。如图点集Q和凸包CH（Q）。
![Image][4]

寻找凸包，可以使用Graham扫描法（Graham’s scan）。借助一个堆栈S，输入集合Q中的每个点都被压入栈一次，非 CH(Q) 中顶点的点最终将被弹出堆栈，算法结束时，堆栈S中仅包含CH（Q）中的顶点，其顺序为各点在边界上出现的逆时针方向排列的顺序。

伪代码：

	GRAHAM-SCAN(Q)
	1  let p0 be the point in Q with the minimum y-coordinate,
	             or the leftmost such point in case of a tie
	2  let 〈p1, p2, …, pm〉 be the remaining points in Q,
	             sorted by polar angle in counterclockwise order around p0
	             (if more than one point has the same angle, remove all but
	             the one that is farthest from p0)
	3  PUSH(p0, S)
	4  PUSH(p1, S)
	5  PUSH(p2, S)
	6  for i ← 3 to m
	7       do while the angle formed by points NEXT-TO-TOP(S), TOP(S),
	                     and pi makes a nonleft turn
	8              do POP(S)
	9          PUSH(pi, S)
	10  return S

第一行找到 y 坐标最小的 p0，第二行根据相对于p0的极角排序（利用叉积），3~5 行堆栈初始化，第 6~9行的迭代对一个点执行一次，意图是：在对点pi进行处理后，在栈S中，按由底到顶得顺序，包含CH（{p0，p1, ……，pi}）中按逆时针方向排列的各个顶点。第 7~8 行由 while 循环把发现不是凸包中的顶点从堆栈中移去。沿逆时针方向通过凸包时，在每个点处应该左转。因此while循环每次发现在一个顶点处没有向左转时，就把该顶点从堆栈中弹出。图示：
![Image][5]

如图 a)→b) 过程中，原本p2在栈S中，到 b) 时，根据p1p3，p1p2的叉积计算，得知 p1p3 在 p1p2 的顺时针方向行，所以 p2 不在CH（Q）中，所以 p2 出栈，p3入栈，整个算法都循环这个过程，后续完整图示：
![Image][6]

代码实现：
{% highlight c %}
//可以顺便AC掉  HDU 1392，去年的代码略加修改
#include<iostream>
#include<cstdio>
#include<cmath>
#include<algorithm>
using namespace std;

typedef struct {
	double x;
	double y;
}Point;
Point p[110], stack[110];

int N, top;

double Direction(Point p1, Point p2, Point p3) {
	return (p2.x - p1.x) * (p3.y - p1.y) - (p2.y - p1.y) * (p3.x - p1.x);
}

double Dis(Point a, Point b) {
	return sqrt((a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y));
}

int cmp(const void *a, const void *b) {
	Point c = *(Point *)a;
	Point d = *(Point *)b;
	double k = Direction(p[0], c, d);
	if(k < 0 || (!k && Dis(c, p[0]) > Dis(d, p[0]))) {
		return 1;
    }
	return -1;
}

void Convex() {
	for(int i = 1; i < N; i++) {
		Point temp;
		if(p[i].y < p[0].y || (p[i].y == p[0].y && p[i].x < p[0].x)) {
			temp = p[i];
			p[i] = p[0];
			p[0] = temp;
		}
	}
	qsort(p + 1, N - 1, sizeof(p[0]), cmp);
	stack[0] = p[0];        
	stack[1] = p[1];
	stack[2] = p[2];
	top =2;
	for(int i = 3; i < N; i++) {
		while(top >= 2 && Direction(stack[top - 1], stack[top], p[i]) <= 0) {
			top--;
        }
		top++;
		stack[top] = p[i];
	}
}

int main() {
	while(~scanf("%d", &N), N){
		for(int i = 0; i < N; i++) {
            scanf("%lf %lf", &p[i].x, &p[i].y);
		}
		cout.setf(ios::fixed);
		cout.precision(2);

		if(N == 1) {
            puts("0.00");
			continue;
		} else if(N == 2) {
            printf("%.2f\n", Dis(p[0], p[1]));
			continue;
		}

		Convex();
		double ans = 0;
		for(int i = 0; i < top; i++) {
			ans += Dis(stack[i],stack[i + 1]);
        }

		ans += Dis(stack[top], stack[0]);
        printf("%.2f\n", ans);
	}
	return 0;
}
{% endhighlight %}	
###三、寻找最近点对

有 n 个点的集合 Q，找出最近的两点，这一问题可以应用于交通控制等系统。在空中或海洋交通控制系统中，需要发现两个距离最近的交通工具，以便检测出可能发生的相撞事故。

暴力必然可以，但是时间复杂度太高，因为要查看 Cn2 = O（n2）个点对，算法导论中介绍的是一种分治策略，输入 P，X 和 Y 的递归调用首先检查是否 \|P\| <= 3。如果是，则用暴力枚举C\|p\|2个点对，并返回最近点对，如果 \|P\| > 3，则分治递归。

找出一条垂直线，近似平分，左边的为 PL，右边的为 PR，类似地X数组划分为两个 XL 和 XR，分别包含 PL和 PR 中的店，按 x 坐标递增排序。类似的，Y 数组划分为连个 YL 和 YR，分别包含 PL 和 PR 中的店，按 y 坐标递增排序。

把 P 分为 PL 和 PR 后，再两次递归调用，一次找出 PL 中的最近点对，另一次找出 PR 中的最近点对。第一次调用输入为 PL，XL，YL，第二次调用输入为 PR，XR，YR。设分别返回的最近距离为 δL 和 δR，则δ = min（δL，δR）

算出左右两边独自的最小距离后，还要计算跨域左右两边的最小距离，即，一个点在PL中，另外一个在PR中。比δ小，且在分布在两边，必定是以下情况：
![Image][7]
即，必定在（-δ，+δ）内，如图a)，另外，最多有 8 个点可能处于 δ × 2δ 矩形区域内，原因是：比如左半边，因为 PL 中的所有点之间的距离至少为 δ 单位，所以至多有 4 个点可能位于该正方形内，PR 同理。这个结论的用处是，在暴力枚举可能范围内的点时，每个点只需考虑紧随其后的 7 个点即可。部分伪代码：

	1  length[YL] ← length[YR] ← 0
	2  for i ← 1 to length[Y]
	3       do if Y[i] ∈ PL
	4             then length[YL] ← length[YL] + 1
	5                  YL[length[YL]] ← Y[i]
	6             else length[YR] ← length[YR] + 1
	7                  YR[length[YR]] ← Y[i]

代码实现：
{% highlight c %}
#include<iostream>
#include<cstdio>
#include<algorithm>
#include<cmath>
#include<cfloat>
using namespace std;
const int maxn = 10000;

struct Point {
    double x, y;
    int index;
}X[maxn], Y[maxn];
double ans;

bool Cmpx(const Point &a, const Point &b) {
    return a.x < b.x;
}

bool Cmpy(const Point &a, const Point &b) {
    return a.y < b.y;
}

double Dis(const Point &a, const Point &b) {
    return sqrt((a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y));
}

//暴力枚举
double ForceMinDis(int a, int b) {
    for (int i = a; i < b; i++) {
        for (int j = i + 1; j <= b; j++) {
            double temp = Dis(X[i], X[j]);
            if (ans > temp) {
                ans = temp;
            }
        }
    }
    return ans;
}

double MinDis(int a, int b, Point *Y) {
    if (b - a <= 3) {
        return ForceMinDis(a, b);
    }

    int mid = (a + b) >> 1;
    Point *Yleft = new Point[mid - a + 1];
    Point *Yright = new Point[b - mid];

    for (int i = 0, j = 0, k = 0; i <= b - a; i++) {
        if (Y[i].index <= mid) {
            Yleft[j++] = Y[i];
        } else {
            Yright[k++] = Y[i];
        }
    }

    double left_min = MinDis(a, mid, Yleft);
    double right_min = MinDis(mid + 1, b, Yright);
    ans = min(left_min, right_min);

    double line = X[mid].x;
    int ind;
    for (int i = 0, ind = 0; i <= b - a; i++) {
        if (fabs(Y[i].x - line) < ans) {
            Y[ind++] = Y[i];
        }
    }
    for (int i = 0; i < ind - 1; i++) {
        for (int j = i + 1; j <= i + 7 && j < ind; j++) {
            double temp = Dis(Y[i], Y[j]);
            if (ans > temp) {
                ans = temp;
            }
        }
    }
    delete Yleft;
    delete Yright;
    return ans;
}

int main() {
    int n, i;
    while (scanf("%d", &n), n) {
        ans = DBL_MAX;
        for (i = 0; i < n; i++) {
            scanf("%lf %lf", &X[i].x, &X[i].y);
        }
        sort(X, X + n, Cmpx);
        for (i = 0; i < n; i++) {
            X[i].index = i;
        }
        memcpy(Y, X, n * sizeof(Point));
        sort(Y, Y + n, Cmpy);
        printf("%lf\n", MinDis(0, n - 1, Y));
    }
    return 0;
}
{% endhighlight %}
这里有个网页《[计算几何算法概览](http://dev.gameres.com/Program/Abstract/Geometry.htm)》，太全面了

[1]: /uploads/2011/11/叉乘.png
[2]: /uploads/2011/11/判断线段旋转方向.png
[3]: /uploads/2011/11/线段相交.png
[4]: /uploads/2011/11/凸包定义.png
[5]: /uploads/2011/11/凸包1.png
[6]: /uploads/2011/11/凸包2.png
[7]: /uploads/2011/11/寻找最近点对.png