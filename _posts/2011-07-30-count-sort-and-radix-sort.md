---
title: 计数排序和基数排序
author: Wei Li
excerpt: MSD：先按k0排序，得到四堆牌，每堆包含属于同一花色的所有纸牌然后分别对每一堆按照面值排序，然后四堆牌叠到一起就是最后结果。
layout: post
permalink: /2011/07/30/count-sort-and-radix-sort/
views:
  - 3147
categories:
  - 算法学习
tags:
  - 排序
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
排序稳定性的一点知识：

1. 稳定的：如果存在多个具有相同排序码的记录，经过排序后，这些记录的相对次序仍然保持不变，则这种排序算法称为稳定的。

	插入排序、冒泡排序、归并排序、分配排序（桶式、基数）都是稳定的排序算法。

2. 不稳定的：否则称为不稳定的。

	直接选择排序、堆排序、shell排序、快速排序都是不稳定的排序算法。

一个各种排序算法演示网站:http://www.sorting-algorithms.com/ (看网址就知道)

###计数排序

元素必须是大于等于0，同时不能太大，否则数组res会暴栈

特点是：非常稳定

作用：经常用作基数排序算法的一个子过程

数组kp存放输入数据

数组Hash开始存放等于i的元素的个数 ，之后存放小于等于i的个数

数组res存放结果

原理就是，利用C中存放的小于等于i的个数，直接就可以确定i的位置

比如，如果有17个元素小于x，则x就属于第18个位置

{% highlight c %}
#include<iostream>
using namespace std;
//kp[1……n] 
int kp[10] = {0, 2, 5, 3, 0, 2, 3, 0, 3};
int res[10], Hash[10];
int length = 8;
  
void print(int kp[], int size) {
    for (int i = 1; i <= size; i++) {
        printf("%d ", kp[i]);
    }
    puts("");
}
 
void CountingSort(int kp[], int res[], int size) {
    memset(Hash, 0, sizeof(*Hash) * size);
    for (int i = 1; i <= length; i++) {
        Hash[kp[i]]++;
    }
    //此时Hash[]包含等于i的元素个数 
    for (int i = 1; i <= size; i++) {
        Hash[i] += Hash[i - 1];
    }
    //此时Hash[]包含小于等于i的元素个数 
    for (int i = length; i >= 1; i--) {
        res[Hash[kp[i]]] = kp[i];
        Hash[kp[i]]--;
         
        //下边5句，是测试输出当前res[],Hash[]情况 
        printf("i = %d\nres[] : ", i);
        print(res, length);
        printf("Hash[] : ");
        printf("%d ", Hash[0]);
        print(Hash, size);
    }
}
 
int main() {
    puts("原始序列:");
    print(kp, length);
     
    CountingSort(kp, res, 5);
     
    puts("排序后:");
    print(res, length);
     
    return 0;
}    
/*  代码运行输出： 
    原始序列:
    2 5 3 0 2 3 0 3
    i = 8
    res[] : 0 0 0 0 0 0 3 0
    Hash[] : 2 2 4 6 7 8
    i = 7
    res[] : 0 0 0 0 0 0 3 0
    Hash[] : 1 2 4 6 7 8
    i = 6
    res[] : 0 0 0 0 0 3 3 0
    Hash[] : 1 2 4 5 7 8
    i = 5
    res[] : 0 0 0 2 0 3 3 0
    Hash[] : 1 2 3 5 7 8
    i = 4
    res[] : 0 0 0 2 0 3 3 0
    Hash[] : 0 2 3 5 7 8
    i = 3
    res[] : 0 0 0 2 3 3 3 0
    Hash[] : 0 2 3 4 7 8
    i = 2
    res[] : 0 0 0 2 3 3 3 5
    Hash[] : 0 2 3 4 7 7
    i = 1
    res[] : 0 0 2 2 3 3 3 5
    Hash[] : 0 2 2 4 7 7
    排序后:
    0 0 2 2 3 3 3 5
 
*/
{% endhighlight %}
###基数排序

首先要搞懂MSD(Most significant digital， 最高位优先级)和LSD(Least significant digital， 最低位优先级)。 例如: 纸牌排序问题，要求结果符合面值，花色从小到大排序

K0：花色 （高优先级)；
K1：面值 （低优先级)；
MSD：先按k0排序，得到四堆牌，每堆包含属于同一花色的所有纸牌然后分别对每一堆按照面值排序，然后四堆牌叠到一起就是最后结果

LSD：先按k1排序，分成13堆，每堆4种花色，面值相同,然后叠为一堆，然后按k0排序，分为4堆，然后叠为一堆，就是结果。

这期间的排序使用的是计数排序……

可以发现MSD每次都是单独子堆分别排序，而LSD每次都是整堆排序，所以LSD比MSD简单，花费开销要小，所以基数排序通常使用LSD，此处的基数，代表用基数r可以分解为多少个个位数。

r = 10, 基数为10, 按10进制分解结果
r = 2， 基数为2, 按2进制分解结果
排序过程即：先按个位数排序，然后按十位数排序，然后百位数排序，类推……

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
 
//关键字项数的最大值，这个测试代码都是用的3位数[0~999] 
const int MaxNumOfKey = 3;
int kp[15] = {278, 109, 63, 930, 589, 184, 505, 269, 8, 83};
int Hash[15], res[15];
int size = 10;//kp数组长度
  
void print(int a[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", a[i]);
    }
    puts("\n");
}
 
void RadixSort(int kp[], int Hash[], int res[], int size) {
    int radix = 1;//基数
    memset(res, 0, sizeof(*res) * size);
    for (int j = 1; j <= MaxNumOfKey; j++) {
        memset(Hash, 0, sizeof(*Hash) * size);
        for (int i = 0; i < size; i++) {
            int tmp = (kp[i] / radix) % 10; 
            Hash[tmp]++;
        }
        for (int i = 1; i < 10; i++) {
            Hash[i] += Hash[i - 1];
        }
         
        for (int i = size - 1; i >= 0; i--) {
            int tmp = (kp[i] / radix) % 10; 
            Hash[tmp]--;
            res[Hash[tmp]] = kp[i];
        }
        //下一次的稳定排序，是建立在前一次的基础上 
        for (int i = 0; i < size; i++) {
            kp[i] = res[i];
        }
         
        radix *= 10;
         
        printf("res[]第%d次排序后: ", j);
        print(res, size);
    }//for( MaxNumOfKey)
}
 
int main() {
    puts("原始序列:");
    print(kp, size);
     
    RadixSort(kp, Hash, res, size);
     
    puts("排序后:");
    print(res, size);
     
    return 0;
}
/*
代码运行输出： 
    原始序列:
    278 109 63 930 589 184 505 269 8 83
     
    res[]第1次排序后: 930 63 83 184 505 278 8 109 589 269
     
    res[]第2次排序后: 505 8 109 930 63 269 278 83 184 589
     
    res[]第3次排序后: 8 63 83 109 184 269 278 505 589 930
     
    排序后:
    8 63 83 109 184 269 278 505 589 930
*/
{% endhighlight %}
严蔚敏版《数据结构》中基数排序演示
![Image](/uploads/2011/07/数据结构中基数排序演示.jpg)

