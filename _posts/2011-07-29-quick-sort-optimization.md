---
title: 快速排序优化
author: Wei Li
excerpt: 由于N <= 20时，插入排序效率高于快速排序。于是就可以想到在快排到递归时：当n <= 20，采用插入排序；当n > 20时，采用快排，这种方法效率可以提高很多；可以看出第三种优化效果非常显著, 仅仅略逊于algorithm自带sort……
layout: post
permalink: /2011/07/29/quick-sort-optimization/
views:
  - 7647
categories:
  - 算法学习
tags:
  - 排序
  - 笔记
  - 算法
  - 算法导论
---
上一篇《[四种快速排序](/2011/07/28/four-quick-sort/)》罗列了常见的4中快排方法，但是，真正使用时，效率还可以提高

1. 实测发现算法导论里的分割+三数中值效率并不高，甚至可以说很低，由下边测试代码Qsort1可知，

2. 由于N <= 20时，插入排序效率高于快速排序，见图

于是就可以想在快排到递归时，当n <= 20，采用插入排序
当n > 20时，采用快排，这种方法效率可以提高很多

下边是测试结果:
size = 1000的数组，排序1000次

所耗时间:

Qsort1 : 188MS

Qsort2 ：179MS

Qsort3 ：149MS

sort : 143MS

可以看出第三种优化效果非常显著, 略逊algorithm自带sort……


第一种，三数中值 + 算法导论里分割方法代码
{% highlight c %}
/*三数中值 + 算法导论里分割方法*/

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
int Median(int kp[], int low, int high) {
	int center = (low + high) >> 1;
	if (kp[low] > kp[center]) swap(kp[low], kp[center]);
	if (kp[low] > kp[high]) swap(kp[low], kp[high]);
	if (kp[center] > kp[high]) swap(kp[center], kp[high]);
	swap(kp[center], kp[high - 1]);//枢纽放到了kp[high - 1] 
	return kp[high - 1];
}

int Partition(int kp[], int low, int high) {
	int pivotkey = Median(kp, low, high);
	int i =  low;
	for (int j = low + 1; j < high - 1; j++) {
		if (kp[j] <= pivotkey) {
			i++;
			swap(kp[j], kp[i]);
		}
	}
	swap(kp[i + 1], kp[high - 1]);
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

第二种，算法导论里分割方法，两排序优化
{% highlight c %}
/*算法导论里分割方法，两排序优化*/

#include<iostream>
#include<cstdio>
#include<cmath> 
#include<algorithm>
using namespace std;

int kp[15] = {2, 8, 7, 1, 3, 5, 6, 4 };
const int size = 8;

void print() {
    for (int i = 0; i < size; i++) {
        printf("%d ", kp[i]);
    }
    puts("");
} 

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
	if (low + 20 <= high) {
        int pivotloc = Partition(kp, low, high);
        Qsort(kp, low, pivotloc - 1);
        Qsort(kp, pivotloc + 1, high);
	} else {
		InsertionSort(kp + low, high - low + 1);
	}	
}//Qsort2

int main() {
    puts("原始序列:");
    print();
    
    Qsort(kp, 0, size - 1);
    
    puts("排序后:");
    print();
    
    return 0;
} 
{% endhighlight %}

第三种，三数中值 +《数算》中分割 + 两种排序优化
{% highlight c %}
/*三数中值 +《数算》中分割 + 两种排序优化
实测，效率最高，是其他快排效率好几倍*/

#include<iostream>
#include<cstdio>
#include<cmath> 
#include<algorithm>
using namespace std;

int kp[15] = {2, 8, 7, 1, 3, 5, 6, 4 };
const int size = 8;

void print() {
    for (int i = 0; i < size; i++) {
        printf("%d ", kp[i]);
    }
    puts("");
} 

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

int Median(int kp[], int low, int high) {
	int center = (low + high) >> 1;
	if (kp[low] > kp[center]) swap(kp[low], kp[center]);
	if (kp[low] > kp[high]) swap(kp[low], kp[high]);
	if (kp[center] > kp[high]) swap(kp[center], kp[high]);
	swap(kp[center], kp[high - 1]);//枢纽放到了kp[high - 1] 
	return kp[high - 1];
}

void Qsort(int kp[], int low, int high) {
	if (low + 20 <= high) {
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
}//Qsort3

int main() {
    puts("原始序列:");
    print();
    
    Qsort(kp, 0, size - 1);
    
    puts("排序后:");
    print();
    
    return 0;
} 
{% endhighlight %}

下面是测试代码，测试内容为文章开头所说的，三种快排的效率
{% highlight c %}
/*改进 + 优化测试
次测试代码结果：(测试环境：C-FREE5.0专业版） 
第一组（数据）： 
t2 - t1 - t = 188
t3 - t2 - t = 179(算法导论里分割方法，两排序优化，效率有所提高)
t4 - t3 - t = 149(最终优化版本，效果明显，多组测试总体略逊自带sort);
t5 - t4 - t = 143(algorithm自带sort) 
第二组：
t2 - t1 - t = 197
t3 - t2 - t = 172
t4 - t3 - t = 126
t5 - t4 - t = 123
第三组：
t2 - t1 - t = 189
t3 - t2 - t = 165
t4 - t3 - t = 119
t5 - t4 - t = 123
第四组:
t2 - t1 - t = 215
t3 - t2 - t = 178
t4 - t3 - t = 129
t5 - t4 - t = 121

再来一组VC++2010中的测试结果:
t2 - t1 - t = 1268
t3 - t2 - t = 886
t4 - t3 - t = 398
t5 - t4 - t = 1594(为什么这么慢 ?) 
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

//三数中值 + 算法导论里分割方法________1
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

int Median(int kp[], int low, int high) {
	int center = (low + high) >> 1;
	if (kp[low] > kp[center]) swap(kp[low], kp[center]);
	if (kp[low] > kp[high]) swap(kp[low], kp[high]);
	if (kp[center] > kp[high]) swap(kp[center], kp[high]);
	swap(kp[center], kp[high - 1]);//枢纽放到了kp[high - 1]
	return kp[high - 1];
}

void Qsort1(int kp[], int low, int high) {
    if (low < high) {
        int pivotloc = Partition(kp, low, high);
        //分治思想，递归排序
        Qsort1(kp, low, pivotloc - 1);
        Qsort1(kp, pivotloc + 1, high);
    }
}//Qsort1

/*算法导论里分割方法，两排序优化*/
int Partition2(int kp[], int low, int high) {
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

void Qsort2(int kp[], int low, int high) {
	if (low + 20 <= high) {
        int pivotloc = Partition2(kp, low, high);
        Qsort2(kp, low, pivotloc - 1);
        Qsort2(kp, pivotloc + 1, high);
	} else {
		InsertionSort(kp + low, high - low + 1);
	}
}//Qsort2

//最终优化版本_____________3
void Qsort3(int kp[], int low, int high) {
	if (low + 20 <= high) {
		int pivotloc = Median(kp, low, high);
		int i = low, j = high - 1;
		for (; ;) {
			while (kp[++i] < pivotloc) {}
			while (kp[--j] > pivotloc) {}
			if (i < j)  swap(kp[i], kp[j]);
			else break;
		}
		swap(kp[i], kp[high - 1]);
		Qsort3(kp, low, i - 1);
		Qsort3(kp, i + 1, high);
	} else {
		InsertionSort(kp + low, high - low + 1);
	}
}//Qsort3

int main() {
	int kp[1010];

	double t0 = clock();
	for (int i = 0; i < N; i++) {
		for (int i = 0; i < size; i++) {
			kp[i] = rand();
		}	 
	}	
	double t1 = clock();

	for (int i = 0; i < N; i++) {
		for (int i = 0; i < size; i++) {
			kp[i] = rand();
		}	 
		Qsort1(kp, 0, size - 1);
	}

	double t2 = clock();
	for (int i = 0; i < N; i++){
		for (int i = 0; i < size; i++) {
			kp[i] = rand();
		}	 
		Qsort2(kp, 0, size - 1);
	}

	double t3 = clock();
	for (int i = 0; i < N; i++) {
		for (int i = 0; i < size; i++) {
			kp[i] = rand();
		}	 
		Qsort3(kp, 0, size - 1);
	}
	double t4 = clock();
	
	for (int i = 0; i < N; i++) {
		for (int i = 0; i < size; i++) {
			kp[i] = rand();
		}	 
		sort(kp, kp + size);
	}
	double t5 = clock();
	double t = t1 - t0;
	cout << "t2 - t1 - t = " << t2 - t1 - t << endl;
	cout << "t3 - t2 - t = " << t3 - t2 - t << endl;
	cout << "t4 - t3 - t = " << t4 - t3 - t << endl;
	cout << "t5 - t4 - t = " << t5 - t4 - t << endl; 
}
{% endhighlight %}

