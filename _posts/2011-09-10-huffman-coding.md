---
title: 哈夫曼编码(Huffman Coding)
author: Wei Li
excerpt: 动态编码是指：编码和解码使用相同的初始化哈夫曼树，每处理完一个字符，编码和解码使用相同的方法修改哈夫曼树，编码和解码一个字符所需的时间与该字符的编码长度成正比，所以动态哈夫曼编码可实时进行。明显的，动态编码更具优势，但是丫的应该会非常复杂。
meta_description: 哈夫蛮编码是为了压缩数据，一般可压缩掉20%~90%，这里的数据通常是指字符串序列。
layout: post
permalink: /2011/09/10/huffman-coding/
views:
  - 3433
categories:
  - 算法学习
tags:
  - 数据结构
  - 树
  - 笔记
  - 算法
  - 算法导论
---
哈夫蛮编码是为了压缩数据，一般可压缩掉20%~90%，这里的数据通常是指字符串序列。

核心思想是：摈弃固定长度编码，而是使用可变长编码，根据频度高低来动态分配编码，也就是，对频度高的字符赋以短编码，对频度低的字符则赋以较长的编码。

《算法导论》里把哈夫蛮编码放到贪心算法这一章来讲解，因为整个哈夫蛮编码算法都在贪心，都在不断找最优值进行编码（就是key最小的值）。

哈夫蛮编码代码实现有两种方式：静态编码方式，动态编码方式。

静态编码是指：先把需编码数据扫一遍，预存数据，第二遍根据预存的数据编码；

动态编码是指：编码和解码使用相同的初始化哈夫曼树，每处理完一个字符，编码和解码使用相同的方法修改哈夫曼树，编码和解码一个字符所需的时间与该字符的编码长度成正比，所以动态哈夫曼编码可实时进行。明显的，动态编码更具优势，但是丫的应该会非常复杂。

我参考的是严蔚敏《数据结构》中的静态编码方式实现，利用C++里的模板实现这个应该会更方便，比如

bitset之类，可惜我C++模板不太熟，先用C吧。

下边第一幅图是是算法导论中的illustrate过程，诡异的是，它给每幅图都排过序了，可能是为了便于理解，
事实上，代码不是这样运行的，模拟建树过程，建议看后边的第二幅图（每次新生成的节点，放最后边）

图1：
![Image][1]

图2：
![Image][2]

代码注释很详细了，直接看代码实现吧：
{% highlight c %}
/*
运行环境：VC++2010
result:
_____________________________
编码结果为：
 5 : 1110
 9 : 1111
12 : 100
13 : 101
16 : 110
45 : 0
*/
#include<iostream>
#include<cstdio>
#include<cstdlib>
#include<climits>
using namespace std;
  
//HuffmanTree --> HT;
typedef struct HuffmanTreeNode HT;
typedef struct HuffmanTreeNode * SearchTree;//哈夫曼树
typedef char * *HTCode;//动态分配数组储存哈夫蛮编码 
  
//哈夫曼树ATD
struct HuffmanTreeNode {
    int key;
    int parent, left, right;
    HuffmanTreeNode(int a, int b, int c, int d) {
        key = a, parent = b, left = c, right = d;
    }
};
  
void CreatHuffmanTree(SearchTree &tree, int *w, int n);
void Select(SearchTree tree, int t, int &s1, int &s2);
void HuffmanCoding (SearchTree tree, HTCode &HC, int n);
void PrintHTCode(HTCode HC, int *c, int n);
  
//构造哈夫曼树
void CreatHuffmanTree(SearchTree &tree, int *w, int n) {
    int i, s1, s2;
    SearchTree p;
    /*哈夫蛮树中没有度为1的结点，是满二叉树，因为根节点不保存信息，
    所以至少应该有2个结点，才能构成满二叉树  */
    if (n <= 1) {
        return ;
    }
    int m = 2 * n - 1;//n个叶子结点的哈夫曼树共有2n - 1个结点
    tree = (SearchTree) malloc((m + 1) * sizeof(HT));//0位置木有结点
  
    for (p = tree + 1, i = 1; i <= n; i++, p++, w++) {
        *p = HT(*w, 0, 0, 0);//前n个结点初始化为已知结点key 
  
    }
    for (; i <= m; i++, p++) {//后n + 1 ~ 2n - 1初始化为0
        *p = HT(0, 0, 0, 0);
    }
  
    //找到最小的两个结点，给新结点tree[i]赋值
    for (i = n + 1; i <= m; i++) {
        Select(tree, i - 1, s1, s2);
        tree[s1].parent = tree[s2].parent = i;
        tree[i] = HT(tree[s1].key + tree[s2].key, tree[i].parent, s1, s2);
    }
}
  
//完成编码任务
void HuffmanCoding (SearchTree tree, HTCode &HC, int n) {
    int i, c, f, start;
    char *cd;
    HC = (HTCode) malloc ((n + 1) * sizeof(char *)); //分配n个字符编码的头指针向量
    cd = (char *) malloc(n * sizeof(char));//分配编码的工作空间
    //n个结点的哈夫曼树高度最多为n, 所以路径最高为n - 1，稍微画下图就可以了解，
    //也就是说编码空间有n - 1就足够了
    cd[n - 1] = '\0';
    for (i = 1; i <= n; i++) {
        start = n - 1;
        //从叶子到根逆向求编码
        for (c = i, f = tree[i].parent; f != 0; c = f, f = tree[f].parent) {
            if (tree[f].left == c) {
                cd[--start] = '0';
            } else {
                cd[--start] = '1';
            }
        }
        HC[i] = (char *)malloc((n - start) * sizeof(char));
        strcpy(HC[i], &cd[start]);
    }
    free(cd);
}
  
//找出最小值，次小值下标
void Select (SearchTree tree, int t, int &s1, int &s2) {
    int i, min1, min2;
    min1 = min2 = INT_MAX;
    /*  当t为偶数时      min1-->最小
                        min2-->次小
        当t为奇数时， min1->次小
                        min2->最小
        因为有两种情况，所以for循环结束之后，还要判断一次来确定最小值，次小值
    */
    for (i = 1; i <= t; i++) {
        if (tree[i].parent == 0 &&
             (tree[i].key < min1 || tree[i].key < min2)) {
                if (min1 < min2) {//先找到次小
                    min2 = tree[i].key;
                    s2 = i;
                } else {//再找到最小
                    min1 = tree[i].key;
                    s1 = i;
                }//if
             }//if (……)
    }//for(i)
    if (s1 > s2) {
        swap(s1, s2);
    }
}
  
//输出编码结果
void PrintHTCode(HTCode HC, int *w, int n) {
    puts("编码结果为：");
    for (int i = 1; i <= n; i++) {
        printf("%2d : %s\n", w[i - 1], HC[i]);
    }
}
  
int w[6] = {5, 9, 12, 13, 16, 45};
const int N = 6;
int main() {
    SearchTree tree;
    HTCode HC;
    CreatHuffmanTree(tree, w, N);
    HuffmanCoding (tree, HC, N);
    PrintHTCode(HC, w, N);
    return 0;
}
{% endhighlight %}
[1]: /uploads/2011/09/HuffmanCoding.jpg
[2]: /uploads/2011/09/HuffmanCode_real.jpg