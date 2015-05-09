---
title: 中位数和顺序统计学
author: Wei Li
excerpt: 第三种算法以最坏情况线性时间做选择，最坏运行时间为O(n)，这种算法基本思想是保证每个数组的划分都是一个好的划分，以5为基，五数取分，这个算法，算法导论没有提供伪代码，额，利用它的思想，可以快速返回和最终中位数相差不超过2的数，这样的划分接近最优，基本每次都二分了（算法导论中步骤见图)
meta_description: 第三种算法以最坏情况线性时间做选择，最坏运行时间为O(n)，这种算法基本思想是保证每个数组的划分都是一个好的划分，以5为基，五数取分，这个算法，算法导论没有提供伪代码，额，利用它的思想，可以快速返回和最终中位数相差不超过2的数，这样的划分接近最优，基本每次都二分了（算法导论中步骤见图)
layout: post
permalink: /2011/08/03/median-and-order-statistics/
views:
  - 2812
categories:
  - 算法学习
tags:
  - 中位数
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
首先要了解几个概念：

顺序统计量：在一个由n个元素组成的集合中，第i个顺序统计量是该集合中第i小的元素。例如一组元素中，最小值是第1顺序统计量，最大值是第n顺序统计量。

中位数：一个中位数是它所在集合的“中点元素”， 当n为奇数时，中位数是唯一的，出现在(I + 1) / 2。当n为偶数是，有两个中位数，分别出现在n / 2 和 n / 2 + 1处，不考虑奇偶性，中位数总是出现在(n + 1) / 2(下中位数)和(n + 1) / 2 + 1(上中位数, Google到都是这样写，《算法导论》这里应该写错了，看了下英文版，同错，额)，算法导论提到的中位数特指下中位数。

问题：

输入：一个包含n个(不同的)数的集合A和一个数i， 1 <= I <= n。

输出：元素x∈A， 它恰大于A中其他的I – 1个元素（即求第k小数）。

三种算法：

1. 直接排序，输出数组第i个元素即可, 时间复杂度为O(nlgn)
2. 这种算法，利用“快排的或者类似二分”的思想，每次以枢纽为界，分两边，每次只需处理一边即可（抛弃另一边），平均情况下的运行时间界为O(n)，这种算法以期望时间做选择。《算法都论》里是，在分治时用随机数来选取枢纽（算法导论中伪代码见图），好吧，这是理论上的算法，它没有考虑实际产生随机数的开销，事实上，效率一点也不高，已经测试过，产生随机数花费的开销真的很大，后边我用更快的三数中值又实现了一遍，思想是一样的，只是效率提高了。
![Image](/uploads/2011/08/第k小随机.jpg)
3. 第三种算法以最坏情况线性时间做选择，最坏运行时间为O(n)，这种算法基本思想是保证每个数组的划分都是一个好的划分，以5为基，五数取分，这个算法，算法导论没有提供伪代码，额，利用它的思想，可以快速返回和最终中位数相差不超过2的数，这样的划分接近最优，基本每次都二分了（算法导论中步骤见图)
![Image](/uploads/2011/08/求第k小数五分作法.jpg)

第一种：以期望线性时间做选择，随机数选取枢纽元
{% highlight c %}
/*这个是随机数版本（算法导论中伪代码实现版本）
之前文章已经测试过，产生随机数本身会有很大开销，所以总体效率不高
  
类似二分的解法，平均运行时间是O(n)
利用 Randomized_Partition找出枢纽kp[pivotloc]
kp[low, pivotloc - 1]为小于它的元素
kp[poivolot + 1, high]为大于它的元素
如果kp[pivotloc]是第i小的元素，则返回
如果kp[pivotloc]大于第i小的元素，则在右边(大于它)的区域查找 
否则，在左边(小于它)的区域查找 
*/
#include<iostream>
#include<cstdio>
using namespace std;
 
int kp[10] = {2, 8, 7, 1, 3, 5, 6, 4};
const int size = 8;
 
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
  
int  Randomized_Select(int kp[], int low, int high, int k) {
    if (low == high) {
        return kp[low];
    }
    int pivotloc =  Randomized_Partition(kp, low, high) ;
    int num = pivotloc - low + 1;//小于等于枢纽，数的个数 
    if (num == k) return kp[pivotloc];
    if (k < num) {
        return Randomized_Select(kp, low, pivotloc - 1, k);
    } else {
        return Randomized_Select(kp, pivotloc + 1, high, k - num);
    }
}
 
int main() {
    for (int i = 0; i < size; i++) {
        printf("第%d小的数是:  ", i + 1); 
        cout <<  Randomized_Select(kp, 0, size - 1, i + 1) << endl;
    }
    return 0;
}
{% endhighlight %}
第二种：以期望线性时间做选择，三数中值选取枢纽元

{% highlight c %}
/*三数取中版本的求第k小数，下边有两个版本 
两个版本分别是我写的和《数据结构与算法分析》里的源码*/
#include<iostream>
#include<cstdio>
#include<cmath> 
#include<algorithm>
using namespace std;
 
int kp[15] = {2, 8, 7, 1, 3, 5, 6, 4 };
const int size = 8;
 
int Median(int kp[], int low, int high) {
    int center = (low + high) >> 1;
    if (kp[low] > kp[center]) swap(kp[low], kp[center]);
    if (kp[low] > kp[high]) swap(kp[low], kp[high]);
    if (kp[center] > kp[high]) swap(kp[center], kp[high]);
    swap(kp[center], kp[high - 1]);//枢纽放到了kp[high - 1] 
    return kp[high - 1];
}
 
//下边是我的解法：三数取中求第k小数 
int Qselect1(int kp[], int low, int high, int k) {
    if (low + 2 <= high) {
        int pivotloc = Median(kp, low, high);
        int i = low, j = high - 1;
        for (; ;) {
            while (kp[++i] < pivotloc) {}
            while (kp[--j] > pivotloc) {}
            if (i < j)  swap(kp[i], kp[j]);
            else break;
        }
        swap(kp[i], kp[high - 1]);
         
        int num = i - low + 1;
        if (k == num) return kp[i];
        if (k < num) {
            return Qselect1(kp, low, i - 1, k);
        } else {
            return Qselect1(kp, i + 1, high, k - num);
        }
    } else {//上边的三数取中不能处理两个元素的情况，下边单独处理 
        if (high == low) {//一个元素情况 
            return kp[low];
        }
        if (high - low == 1) {//两个元素情况 
            if (kp[low] > kp[high]) swap(kp[low], kp[high]);
            if (k == 1) return kp[low];
            return kp[high];
        }
    }
}//Qselect1
   
//插入排序 
void InsertionSort(int kp[], int n) {
    for (int j, i = 1; i < n; i++) {
        int tmp = kp[i];
        for (j = i; j > 0 && kp[j - 1] > tmp; j--) {
            kp[j] = kp[j - 1];
        }
        kp[j] = tmp;
    }
}
 
//后来才发现《数据结构与算法分析》理有三数取中求第k小的代码
//不过我写的和它还是有些不一样…… 下边是它的版本 （后来加） 
void Qselect2(int kp[], int low, int high, int k) {
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
         
        if (k <= i) {
            return Qselect2(kp, low, i - 1, k);
        } else if (k > i + 1) {
            return Qselect2(kp, i + 1, high, k);
        }
    } else {
         InsertionSort(kp + low, high - low + 1);
    }
}//Qselect2
 
int main() {
    for (int i = 0; i < size; i++) {
        printf("第%d小的数是:  ", i + 1); 
        cout <<  Qselect1(kp, 0, size - 1, i + 1) << endl;
    }
/*    //已经测试过，两种都是正确的 
    for (int i = 0; i < size; i++) {
        Qselect2(kp, 0, size - 1, i + 1);
        printf("第%d小的数是:  ", i + 1); 
        cout <<  kp[i] << endl;
    }*/
    return 0;
}
{% endhighlight %}

第三种：最坏情况线性时间做选择，五数划分中位数做枢纽元

{% highlight c %}
/*利用中位数来选取枢纽元,这种方法最坏情况下运行时间是O(n) 
 这里求的中位数是下中位数 
算法导论里没有伪代码，写起来很麻烦
注意这里的查找到的中位数，并不是真正意义上的中位数
而是和真正中位数相差不超过2的一个数
开始以为我写错了，又看了算法导论，应该就是这个意思 
返回的是[x - 1, x + 2]的一个数，中位数是x
从下边的输出中也可以看出： 
*/
#include<iostream>
#include<cstdio>
using namespace std;
 
const int maxn = 14;//kp -> size 
const int maxm = maxn / 5 + 1;//mid -> size 
int kp[maxn];
int mid[maxm];
 
//插入排序 
void InsertionSort(int kp[], int n) {
    for (int j, i = 1; i < n; i++) {
        int tmp = kp[i];
        for (j = i; j > 0 && kp[j - 1] > tmp; j--) {
            kp[j] = kp[j - 1];
        }
        kp[j] = tmp;
    }
}
 
//查找中位数, 保证每一个划分都是好的划分 
 int FindMedian(int kp[], int low, int high) {
     if (low == high) {
         return kp[low];
     }
     int index = low;//index初始化为low 
      
     //如果本身小于5个元素，这一步就跳过 
     if (high - low + 1 >= 5) {
         //储存中位数到mid[] 
         for (index = low; index <= high - 4; index += 5) {
             InsertionSort(kp + index, 5);
             int num = index - low;
             mid[num / 5] = kp[index + 2];
         }
     }
     //处理剩下不足5个的元素
     int remain = high - index + 1;
     if (remain > 0) {
         InsertionSort(kp + index, remain);
         int num = index - low;
         mid[num / 5] = kp[index + (remain >> 1)];//下中位数 
     }
      
     int cnt = (high - low + 1) / 5;
     if ((high - low + 1) % 5 == 0) {
         cnt--;//下标是从0开始，所以需要-1 
     }//存放在[0…tmp] 
      
     if (cnt == 0) {
         return mid[0];
     } else {
         return FindMedian(mid, 0, cnt);
     }
 }
   
int Qselect(int kp[], int low, int high, int k) {
    int pivotloc = FindMedian(kp, low, high);
    //这里有点不一样，因为不知道pivotloc下标，所以全部都要比较  
    int i = low - 1, j = high + 1;
    for (; ;) {
        while (kp[++i] < pivotloc) {}
        while (kp[--j] > pivotloc) {}
        if (i < j)  swap(kp[i], kp[j]);
        else break;
    }
 
    int num = i - low + 1;
    if (k == num) return kp[i];
    if (k < num) {
        return Qselect(kp, low, i - 1, k);
    } else {
        return Qselect(kp, i + 1, high, k - num);
    }
}
int main() {
    int kp[maxn] = {10, 14, 8, 11, 7, 1, 2, 13, 3, 12, 4, 9, 6, 5};
    for (int i = 0; i < maxn; i++) {
        printf("中位数是:  %d\n", FindMedian(kp, 0, maxn - 1));
        printf("第%d小的数是:  ", i + 1); 
        cout <<  Qselect(kp, 0, maxn - 1, i + 1) << endl << endl;
    }
    return 0;
}
/* 
输出：
_______________ 
中位数是:  6
第1小的数是:  1
 
中位数是:  9
第2小的数是:  2
 
中位数是:  8
第3小的数是:  3
 
中位数是:  8
第4小的数是:  4
 
中位数是:  8
第5小的数是:  5
 
中位数是:  8
第6小的数是:  6
 
中位数是:  8
第7小的数是:  7
 
中位数是:  8
第8小的数是:  8
 
中位数是:  8
第9小的数是:  9
 
中位数是:  8
第10小的数是:  10
 
中位数是:  8
第11小的数是:  11
 
中位数是:  8
第12小的数是:  12
 
中位数是:  8
第13小的数是:  13
 
中位数是:  8
第14小的数是:  14
*/
{% endhighlight %}