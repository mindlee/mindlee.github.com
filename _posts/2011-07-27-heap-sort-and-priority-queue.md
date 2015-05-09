---
title: 堆排序和优先级队列
author: Wei Li
excerpt: 堆排序实现效率通常不及快速排序，但堆数据结构还是有很大好处，比如堆的一个常见应用：高效的的优先级队列优先队列是种用来维护由一组元素构成的集合S的数据结构，优先级队列分最大优先级队列和最小优先级队列，下边是最大堆实现的最大优先级队列。
meta_description: 堆排序实现效率通常不及快速排序，但堆数据结构还是有很大好处，比如堆的一个常见应用：高效的的优先级队列优先队列是种用来维护由一组元素构成的集合S的数据结构，优先级队列分最大优先级队列和最小优先级队列，下边是最大堆实现的最大优先级队列。
layout: post
permalink: /2011/07/27/heap-sort-and-priority-queue/
views:
  - 5172
categories:
  - 算法学习
tags:
  - 堆
  - 排序
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
堆排序算法, 数组实现，具体模拟过程，算法导论P79很详细，下边是把伪代码用C/C++实现；堆排序运行时间主要消耗在建初始堆和调整建新堆时进行的反复筛选上，所以适用于n比较大的情况下
{% highlight c %}
#include<iostream>
#include<cstdio>
using namespace std;
#define lson (idx << 1)//左孩子 
#define rson (lson | 1)//右孩子 
 
int kp [15] = {0, 4, 1, 3, 2, 16, 9, 10, 14, 8, 7};
int size = 10;
 
// 输出当前序列 
inline void  print() {
    for (int i = 1; i <= size; i++) {
        printf("%d ", kp[i]);
    } puts("");
}
//维护最大堆 
//从节点id开始，维护以id为根节点的子树 
//使以id为根节点的子树成为最大堆 
inline void MaxHeaplfy(int kp[], int idx, int size) {
    int largest = idx;
    /*下边两个if，是找出（当前节点，左孩子，右孩子）中
    最大值的下标*/
    if (lson <= size && kp[lson] > kp[idx]) {
        largest = lson;
    } 
    if (rson <= size && kp[rson] > kp[largest]) {
        largest = rson;
    }
    //如果最大值不是根节点，那么交换kp[id], kp[largest]
    //然后递归，维护以largest为根节点的子树 
    if (largest != idx) {
        swap(kp[idx], kp[largest]);
        MaxHeaplfy(kp, largest, size);
    }
}
 
/*
建堆，下边从size/2 ~ 1
i = size / 2时，使 [size / 2, size]为最大堆
 i = size / 2 - 1时, 使[size / 2 - 1, size]为最大兑
建堆就是通过每次迭代来跟新最大堆，最后当
i = 1时，[1, size]为最大堆 
*/
inline void BuildMaxHeap(int kp[]) {
    for (int i = size >> 1; i >= 1; i--) {
        MaxHeaplfy(kp, i, size);
    }
}
 
/*堆排序，最大堆中，最顶端的元素
是所有元素中最大的， 利用这个性质
每次交换kp[size],  kp[1], 即kp[size]中存放当前最大元素
然后调用MaxHeapify使(kp[size - 1], kp[1])成为最大堆 
继续交换kp[size - 1], kp[1], 使kp[size - 1]存放当前最大元素
然后调用 MaxHeapify使(kp[size - 2], kp[1])成为最大堆 
当循环结束时，kp[]中元素就从小到大排好序了*/
inline void HeadSort(int kp[]) {
    BuildMaxHeap(kp);
         
    puts("排序过程: ");
    print();
     
    for (int i = size; i >= 2; i--) {
        swap(kp[1], kp[i]);
        //kp[i]，存放最大值，然后使(kp[1], kp[i - 1])为最大堆 
        MaxHeaplfy(kp, 1, i - 1); 
        print();
    }
}
     
int main() {
    puts("初始状态:");
    print(); 
     
    HeadSort(kp);
     
    puts("堆排序后:"); 
    print();
}
{% endhighlight %}

堆排序实现效率通常不及快速排序，但堆数据结构还是有很大好处，比如堆的一个常见应用：高效的的优先级队列，优先队列是种用来维护由一组元素构成的集合S的数据结构，优先级队列分最大优先级队列和最小优先级队列，下边是最大堆实现的最大优先级队列 ，C/C++基于《算法导论》伪代码实现 ；优先级队列应用：基于优先级的作业调度，当一个作业做完或被中断时，用 HeapExtractMax操作从等待的作业中选择出具有最高优先级的作业，任何时候，一个新作业都可以用 HeapInsert加入到队列中中去

{% highlight c %}
#include<iostream>
#include<cstdio>
using namespace std;
#define inf 0x7fffffff
#define lson (idx << 1)//左孩子 
#define rson (lson | 1)//右孩子 
#define parent (idx >> 1)//父母节点 
 
 
int kp [20] = {0, 4, 1, 3, 2, 16, 9, 10, 14, 8, 7};
int size = 10;
 
// 输出当前序列 
inline void  print() {
    for (int i = 1; i <= size; i++) {
        printf("%d ", kp[i]);
    } puts("");
}
  
inline void MaxHeaplfy(int kp[], int idx, int size) {
    int largest = idx;
    if (lson <= size && kp[lson] > kp[idx]) {
        largest = lson;
    } 
    if (rson <= size && kp[rson] > kp[largest]) {
        largest = rson;
    }
    if (largest != idx) {
        swap(kp[idx], kp[largest]);
        MaxHeaplfy(kp, largest, size);
    }
}
inline void BuildMaxHeap(int kp[]) {
    for (int i = size >> 1; i >= 1; i--) {
        MaxHeaplfy(kp, i, size);
    }
}
 
inline void HeadSort(int kp[]) {
    BuildMaxHeap(kp);    
    for (int i = size; i >= 2; i--) {
        swap(kp[1], kp[i]);
        MaxHeaplfy(kp, 1, i - 1); 
    }
}
//返回S中最大关键字的元素 
inline int HeapMaximum(int kp[]) {
    return kp[1];
}
//去掉并返回S中具有最大关键字的元素,即返kp[1],并删除之 
inline int HeapExtractMax(int kp[], int size) {
    if (size < 1) {
        return -1;//溢出 
    }
    int maxx = kp[1];
    kp[1] = kp[size];
    MaxHeaplfy(kp, 1, size - 1);
    return maxx;
}
/* 将元素idx（下标）的关键字的值增加到key，这里key必须大于等于kp[idx]；
由于新的值key肯跟会破坏最大堆，随意需要从idx节点开始，往根节点移动，
不断与其父母节点相比，如果次关键字大于父母节点关键字，交换，并继续比较
；否则，最大堆条件成立，程序终止*/
inline void HeapIncreaseKey(int kp[], int idx, int key) {
    if (key < kp[idx]) {//不合法 
        return ;
    }
    kp[idx] = key;
    while (idx > 1 && kp[parent] < kp[idx]) {
        swap(kp[parent], kp[idx]);
        idx = parent;
    }
}
/*将元素key插入到集合S， 先加入一个关键字值-inf的叶节点，
然后调用 HeapIncreaseKey将节点关键字增大道key*/
inline void HeapInsert(int kp[], int key, int size) {
    kp[size + 1] = -inf;
    HeapIncreaseKey(kp, size + 1, key);
}
     
int main() {
    puts("初始状态:");
    print(); 
     
    BuildMaxHeap(kp);
     
    puts("初始化堆后(建堆BuildMaxHeap):"); 
    print();
      
     puts("获取并删除最大元素后：(HeapExtractMax)");
     size--;
     HeapExtractMax(kp, size);
    print(); 
             
    puts("当前堆最大元素是:(HeapMaximum)");
     printf("%d\n", HeapMaximum(kp));
      
     puts("增加新元素11后：(HeapInsert)");
     HeapInsert(kp, 11, size);
     print();
      
     puts("将11增大到15后:(11是第二个元素，HeapIncreaseKey)");
     HeapIncreaseKey(kp, 2, 15);
     print();      
}
{% endhighlight %} 
