---
title: 四种快速排序
author: Wei Li
excerpt: 这种快排的额外好处： kp[low], kp[center], kp[high]排好序后， kp[low], kp[high]本身就处在了分割阶段应该在的位置因此，可以将枢纽元放到kp[high - 1]并在分割阶段将 i和j初始化到low + 1, high - 1，可以减少一些操作。
meta_description: 这种快排的额外好处： kp[low], kp[center], kp[high]排好序后， kp[low], kp[high]本身就处在了分割阶段应该在的位置因此，可以将枢纽元放到kp[high - 1]并在分割阶段将 i和j初始化到low + 1, high - 1，可以减少一些操作。
layout: post
permalink: /2011/07/28/four-quick-sort/
views:
  - 5727
categories:
  - 算法学习
tags:
  - 排序
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
引用《数据结构与算法分析》中“快排中 选取枢纽元：”的描述：

选择枢纽元：

1. 一种错误的方法

	通常的、没有经过充分考虑的选择是将第一个元素用作枢纽元。如果输入是随机的，那么这是可以接受的，但是如果输入是预排序的或者是反序的，那么这样的枢纽元就产生一个劣质的分割，因为所有的元素不是被划入S1就是被划入S2。更有甚者，这种情况发生在所有的递归调用中。实际上，如果第一个元素用作枢纽元而且输入是预先排序的，那么快速排序所花费的时间将是二次的。然而，预排序的输入（或者有一大段预排序数据的输入）是相当常见的，因此，使用第一个元素作为枢纽元是非常糟糕的，应该立即放弃这种想法。另一种想法是选取前两个互异的键中的较大者作为枢纽元，但这和只选取第一个元素作为枢纽元具有相同的害处。不要使用这两种选取枢纽元的策略。

2. 一种安全的作法
  
	一种安全的方针是随机选取枢纽元。一般来说这种策略非常安全，除非随机数生成器有问题（这不像你所想象的那么罕见），因为随机的枢纽元不可能总在接连不断地产生劣质的分割。另一方面，随机数的生成一般是昂贵的，根本减少不了算法其余部分的平均运行时间。

3. 三数中分割法

	一组N个数的中值是第[N/2]个最大的数。枢纽元的最好的选择是数组的中值。可是，这很难算出来，并且会明显减慢快速排序的速度。这样的中值的估计可以通过随机选取三个元素并用它们的中值作为枢纽元而得到。事实上，随机性并没有多大的帮助，因此一般的做法是使用左端、右端和中心位置上的三个元素的中值作为枢纽元。显然使用三数中值分割法消除了预排序输入的不好情形。

下面最初始的"Hoare快排”和"算法导论中所用到的优化版本”属于第一种，即枢纽元是第一个元素。

第三个"随机化版本”是第二种

第四个"三数取中”版本是第三种
<hr/>
下面是霍尔排序

{% highlight c %}
/*第一种是最初始的快排：Hoare快排*/
 
#include<iostream>
#include<cstdio>
using namespace std;
 
int kp[10] = {2, 8, 7, 1, 3, 5, 6, 4};
const int size = 8;
 
void print() {
    for (int i = 0; i < size; i++) {
        printf("%d ", kp[i]);
    }
    puts("");
}
 
//一趟快排之后，以枢纽为分隔，左边的<=枢纽, 右边的>=枢纽 
int Partition(int kp[], int low, int high) {
    int pivotkey = kp[low];//选定枢轴 
    while (low < high) {
        //比枢纽小的交换到低端 
        while (low < high && kp[high] >= pivotkey) high--;
        swap(kp[low], kp[high]);
        //比枢纽大的交换到高端 
        while (low < high && kp[low] <= pivotkey) low++;
        swap(kp[low], kp[high]);
    }
    return low;
}//Partition
 
void Qsort(int kp[], int low, int high) {
    if (low < high) {
        int pivotloc = Partition(kp, low, high);
        //分治思想，递归排序 
        Qsort(kp, low, pivotloc - 1);
        Qsort(kp, pivotloc + 1, high);
    }
}//Qsort
 
int main() {
    puts("原始序列:");
    print();
     
    Qsort(kp, 0, size - 1);
     
    puts("排序后:");
    print();
     
    return 0;
}
{% endhighlight %}

下面是算法导论里的快速排序
{% highlight c %}
/*算法导论里优化后的快排（N.Lomuto优化) 
假如进行一次快排Partition，运行时间为n + X
这里的X为额外的比较或一个时间单元的累加
下边的算法相对于Hoare快排, 减少了比较所耗的时间
，但是比较次数实际上也是视情况而比较
但总体上，这个优化版本效率上有提升*/
 
#include<iostream>
#include<cstdio>
using namespace std;
 
int kp[10] = {2, 8, 7, 1, 3, 5, 6, 4};
const int size = 8;
 
void print() {
    for (int i = 0; i < size; i++) {
        printf("%d ", kp[i]);
    }
    puts("");
}
     
int Partition(int kp[], int low, int high) {
    int pivotkey = kp[high];
    int i =  low - 1;
    for (int j = low; j < high; j++) {
        if (kp[j] <= pivotkey) {
            i++;
            swap(kp[j], kp[i]);
        }
    }
    swap(kp[i + 1], kp[high]);
    return i + 1;
}//Partition
 
void Qsort(int kp[], int low, int high) {
    if (low < high) {
        int pivotloc = Partition(kp, low, high);
        //分治思想，递归排序 
        Qsort(kp, low, pivotloc - 1);
        Qsort(kp, pivotloc + 1, high);
    }
}//Qsort
 
int main() {
    puts("原始序列:");
    print();
     
    Qsort(kp, 0, size - 1);
     
    puts("排序后:");
    print();
     
    return 0;
}
{% endhighlight %}

算法导论中的快排图示
![Image](/uploads/2011/07/算法导论中的快排图示.png)

下面是随机化版本的快速排序

{% highlight c %}
/*快排的随机化版本
随机化版本由于是随机取枢纽，可以减少遇到最坏情况的可能
但是，产生随机数同时会产生大的开销
算法导论中原话："
很多人认为，快速排序的随机化版本是对足够大的输入的理想选择"
所以输入不多时，随机化快排没有优势 
*/
 
#include<iostream>
#include<cstdio>
using namespace std;
 
int kp[10] = {2, 8, 7, 1, 3, 5, 6, 4};
const int size = 8;
 
void print() {
    for (int i = 0; i < size; i++) {
        printf("%d ", kp[i]);
    }
    puts("");
}
 
int Partition(int kp[], int low, int high) {
    int pivotkey = kp[high];
    int i =  low - 1;
    for (int j = low; j < high; j++) {
        if (kp[j] <= pivotkey) {
            i++;
            swap(kp[j], kp[i]);
        }
    }
    swap(kp[i + 1], kp[high]);
    return i + 1;
}//Partition
 
inline int Random(int low, int high) {
    return (rand() % (high - low + 1)) + low;
}
 
int Randomized_Partition(int kp[], int low, int high) {
     int i = Random(low, high);
     swap(kp[low], kp[i]);
     return Partition(kp, low, high);
}
 
void Randomized_Quicksort(int kp[], int low, int high) {
    if (low < high) {
        int pivotloc = Randomized_Partition(kp, low, high);
        //分治思想，递归排序 
        Randomized_Quicksort(kp, low, pivotloc - 1);
        Randomized_Quicksort(kp, pivotloc + 1, high);
    }
}//Qsort
 
int main() {
    puts("原始序列:");
    print();
     
    Randomized_Quicksort(kp, 0, size - 1);
     
    puts("排序后:");
    print();
     
    return 0;
}
/*
补充一点随机数的知识
rand()产生的是0~RAND_MAX之间的随机数
产生一定范围随机数的通用表示公式
要取得[a,b)的随机整数，使用(rand() % (b-a))+ a;
要取得[a,b]的随机整数，使用(rand() % (b-a+1))+ a;
要取得(a,b]的随机整数，使用(rand() % (b-a))+ a + 1;
通用公式:a + rand() % n；其中的a是起始值，n是整数的范围。
要取得a到b之间的随机整数，另一种表示：a + (int)b * rand() / (RAND_MAX + 1)。
要取得0～1之间的浮点数，可以使用rand() / double(RAND_MAX)。
*/ 
{% endhighlight %}
三数取中版本的快排，效率相当高

{% highlight c %}
/*三数中值分割法版本的快排，
下边是《数据结构与算法分析》中的快排版本 
这种快排的额外好处：
kp[low], kp[center], kp[high]排好序后，
kp[low], kp[high]本身就处在了分割阶段应该在的位置
因此，可以将枢纽元放到kp[high - 1]并在分割阶段将
i和j初始化到low + 1, high - 1，可以减少一些操作
*/
 
#include<iostream>
#include<cstdio>
using namespace std;
 
int kp[15] = {2, 8, 7, 1, 3, 5, 6, 4 };
const int size = 8;
 
void print() {
    for (int i = 0; i < size; i++) {
        printf("%d ", kp[i]);
    }
    puts("");
}
 
int Median(int kp[], int low, int high) {
    int center = (low + high) >> 1;
    if (kp[low] > kp[center]) swap(kp[low], kp[center]);
    if (kp[low] > kp[high]) swap(kp[low], kp[high]);
    if (kp[center] > kp[high]) swap(kp[center], kp[high]);
    swap(kp[center], kp[high - 1]);//枢纽放到了kp[high - 1] 
    return kp[high - 1];
}
 
//<数据结构与算法分许与设计》中讲对于小数组(N <= 20) 
//快速排序速度不及插入排序
//所以下面Qsort里当数组很小时，调用的是插入排序 
void InsertionSort(int kp[], int n) {
    for (int j, i = 1; i < n; i++) {
        int tmp = kp[i];
        for (j = i; j > 0 && kp[j - 1] > tmp; j--) {
            kp[j] = kp[j - 1];
        }
        kp[j] = tmp;
    }
}
         
void Qsort(int kp[], int low, int high) {
    if (low + 3 <= high) {
        int pivotloc = Median(kp, low, high);
        int i = low, j = high - 1;
        for (; ;) {
            while (kp[++i] < pivotloc) {}
            while (kp[--j] > pivotloc) {}
            if (i < j)  swap(kp[i], kp[j]);
            else break;
        }
        swap(kp[i], kp[high - 1]);
        Qsort(kp, low, i - 1);
        Qsort(kp, i + 1, high);
    } else {
        InsertionSort(kp + low, high - low + 1);
    }   
}//Qsort
 
int main() {
    puts("原始序列:");
    print();
     
    Qsort(kp, 0, size - 1);
     
    puts("排序后:");
    print();
     
    return 0;
}    
{% endhighlight %}
测试代码，直接看结果即可

{% highlight c %}
/*下面是测试四种快排效率的代码，可以略过
在下边这个例子中，结果输出是：
此测试结果,编译环境为VC++2010
测试结果第一组： 
t2 - t1 - t: 1050  （霍尔排序)
t3 - t2 - t: 1286  （改进版本)
t4 - t3 - t: 1458  (随机数版本）
t5 - t4 - t: 626    (三数中值优化版本）
t6 - t5 - t: 1575   (自带sort版本)
第二组： 
t2 - t1 - t: 1078 
t3 - t2 - t: 1316  
t4 - t3 - t: 1430  
t5 - t4 - t: 644  
t6 - t5 - t: 1581
    
在C-FREE 5中，结果为
第一组: 
t2 - t1 - t = 187
t3 - t2 - t = 203
t4 - t3 - t = 239
t5 - t4 - t = 148
t6 - t5 - t = 124
第二组：
t2 - t1 - t = 221
t3 - t2 - t = 224
t4 - t3 - t = 231
t5 - t4 - t = 133
t6 - t5 - t = 123
 
两种编译器，结果相差很大，可能是不同编译器使用了不同的字节对齐方式
 
测试几组数据，霍尔排序改进版本效率竟然不及霍尔排序 
这个两个应该视数据而定，可能效率平均下来差不多 
 
sort两种编译环境，一个最慢，一个最快，很诡异 
*/
 
#include<iostream>
#include<cstdio>
#include<ctime>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int N = 1000;
const int size = 1000;
 
void InsertionSort(int kp[], int n) {
    for (int j, i = 1; i < n; i++) {
        int tmp = kp[i];
        for (j = i; j > 0 && kp[j - 1] > tmp; j--) {
            kp[j] = kp[j - 1];
        }
        kp[j] = tmp;
    }
}
 
int Partition1(int kp[], int low, int high) {
    int pivotkey = kp[low];//选定枢轴 
    while (low < high) {
        //比枢纽小的交换到低端 
        while (low < high && kp[high] >= pivotkey) high--;
        swap(kp[low], kp[high]);
        //比枢纽大的交换到高端 
        while (low < high && kp[low] <= pivotkey) low++;
        swap(kp[low], kp[high]);
    }
    return low;
}//Partition1
 
int Partition(int kp[], int low, int high) {
    int pivotkey = kp[high];
    int i =  low - 1;
    for (int j = low; j < high; j++) {
        if (kp[j] <= pivotkey) {
            i++;
            swap(kp[j], kp[i]);
        }
    }
    swap(kp[i + 1], kp[high]);
    return i + 1;
}//Partition
 
//霍尔排序1————————————————————————————————————————
void Qsort1(int kp[], int low, int high) {
    if (low < high) {
        int pivotloc = Partition1(kp, low, high);
        //分治思想，递归排序 
        Qsort1(kp, low, pivotloc - 1);
        Qsort1(kp, pivotloc + 1, high);
    }
}//Qsort
 
//算法导论优化版本2————————————————————————————————————
void Qsort2(int kp[], int low, int high) {
    if (low < high) {
        int pivotloc = Partition(kp, low, high);
        //分治思想，递归排序 
        Qsort2(kp, low, pivotloc - 1);
        Qsort2(kp, pivotloc + 1, high);
    }
}//Qsort
 
//随机化版本3———————————————————————————————————————
inline int Random(int low, int high) {
    return (rand() % (high - low + 1)) + low;
}
 
int Randomized_Partition(int kp[], int low, int high) {
     int i = Random(low, high);
     swap(kp[low], kp[i]);
     return Partition(kp, low, high);
}
 
void Randomized_Quicksort(int kp[], int low, int high) {
    if (low < high) {
        int pivotloc = Randomized_Partition(kp, low, high);
        //分治思想，递归排序 
        Randomized_Quicksort(kp, low, pivotloc - 1);
        Randomized_Quicksort(kp, pivotloc + 1, high);
    }
}//Qsort
 
//《数据结构与算法分析》中原版三数中值——————————————————————————————
 
int Median(int kp[], int low, int high) {
    int center = (low + high) >> 1;
    if (kp[low] > kp[center]) swap(kp[low], kp[center]);
    if (kp[low] > kp[high]) swap(kp[low], kp[high]);
    if (kp[center] > kp[high]) swap(kp[center], kp[high]);
    swap(kp[center], kp[high - 1]);//枢纽放到了kp[high - 1] 
    return kp[high - 1];
}
     
void Qsort4(int kp[], int low, int high) {
    if (low + 3 <= high) {
        int pivotloc = Median(kp, low, high);
        int i = low, j = high - 1;
        for (; ;) {
            while (kp[++i] < pivotloc) {}
            while (kp[--j] > pivotloc) {}
            if (i < j)  swap(kp[i], kp[j]);
            else break;
        }
        swap(kp[i], kp[high - 1]);
        Qsort4(kp, low, i - 1);
        Qsort4(kp, i + 1, high);
    } else {
        InsertionSort(kp + low, high - low + 1);
    }    
}//Qsort
 
 
int main() {
    int kp[1010];
    double t0 = clock();
    for (int i = 0; i < N; i++) {
        for (int i = 0; i < size; i++) {
            kp[i] = rand();
        }
    }
    double t1 = clock();
   //霍尔排序 
    for (int i = 0; i < N; i++) {
        for (int i = 0; i < size; i++) {
            kp[i] = rand();
        }
        Qsort1(kp, 0, size - 1);
    }
 
    double t2 = clock();
    //改进版本 
    for (int i = 0; i < N; i++){
        for (int i = 0; i < size; i++) {
            kp[i] = rand();
        }
        Qsort2(kp, 0, size - 1);
    }
 
    double t3 = clock();
    //随机数快排 
    for (int i = 0; i < N; i++) {
        for (int i = 0; i < size; i++) {
            kp[i] = rand();
        }	
        Randomized_Quicksort(kp, 0, size - 1);
    }
    double t4 = clock();
    //三数中值优化版本
    for (int i = 0; i < N; i++){
        for (int i = 0; i < size; i++) {
            kp[i] = rand();
        }
        Qsort4(kp, 0, size - 1);
    }
    double t5 = clock();
    //自带sort 
    for (int i = 0; i < N; i++) {
        for (int i = 0; i < size; i++) {
            kp[i] = rand();
        }
        sort(kp, kp + size);
    }
    double t6 = clock();
    double t = t1 - t0;
    cout << "t2 - t1 - t = " << t2 - t1 - t << endl;
    cout << "t3 - t2 - t = " << t3 - t2 - t << endl;
    cout << "t4 - t3 - t = " << t4 - t3 - t << endl;
    cout << "t5 - t4 - t = " << t5 - t4 - t << endl;
    cout << "t6 - t5 - t = " << t6 - t5 - t << endl; 
}  
{% endhighlight %}
如果还想更加深入，还可以去这里看看这篇《[快速排序优化](/2011/07/29/quick-sort-optimization/)》
