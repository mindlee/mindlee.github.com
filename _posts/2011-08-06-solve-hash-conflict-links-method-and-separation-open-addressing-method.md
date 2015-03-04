---
title: 解决散列冲突之分离链接法和开放寻址法
author: 酷~行天下
excerpt: 散列表的缺点就是容易出现冲突（也叫碰撞），两个关键字可能映射到同一个槽中，然后就产生了冲突，解决冲突的方法有很多种，这里只讨论其中最简单的两种：分离链接法和开放寻址法。
layout: post
permalink: /2011/08/06/solve-hash-conflict-links-method-and-separation-open-addressing-method/
views:
  - 5801
categories:
  - 算法学习
tags:
  - 散列
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
如果对散列不了解，可以先去看看这篇《[散列及散列函数](/2011/08/06/hash-and-hash-function/)》

散列表的缺点就是容易出现冲突（也叫碰撞），两个关键字可能映射到同一个槽中，然后就产生了冲突，解决冲突的方法有很多种，这里只集中讨论其中最简单的两种：

第一种是：分离链接法，就把散列到同一个槽中的所有元素都放在一个链表中，如果，槽 j 中有一个指针，它指向所有散列到j的元素构成的链表的头；如果不存在这样的元素，则j中为NIL，如图应该可以很清楚了，只是代码实现很麻烦，指针的指针，尼玛，另外，malloc开辟空间也是笔不小的开销。
![Image](/uploads/2011/08/分离链接法.jpg)
代码测试环境是：VC++2010，可能有些编译器会出错，把malloc前边的强制类型转换去掉应该就可以了

下面是代码，包含查找， 插入，删除操作：
{% highlight c %}
/*最好先手动画下图，或者看上边那张图，这样好理解，否则不太好理解二重指针
输出：
————————————————
表的大小是；23
 
89插入表中
18插入表中
49插入表中
58插入表中
69插入表中
 
小伙子，69查找成功了
25不存在表中
69删除成功
25不存在, 无法删除
*/
 
#include<iostream>
#include<cstdio>
#include<cstdlib>
using namespace std;
 
struct HashTbl;
struct ListNode;
 
typedef struct HashTbl * HashTable;//不用这几个typedef程序很难看懂
typedef struct ListNode * Position;
//下边这个typedef也是为了更好的理解
typedef Position List;
 
const int maxn = 400000; // maxn : size 
bool prime[maxn]; // true : prime number
 
//Sieve Prime素数筛法, 先预处理好，否则很费时间
//单独判定素数是非常耗时间的
void Seive_Prime() {//判素,素数筛选 
     memset(prime, true, sizeof(prime));
     prime[0] = prime[1] = false;
     for (int i = 2; i * i < maxn; i++) {
          if (prime[i]) {
               for (int j = i * i; j < maxn ; j += i) {
                    prime[j] = false;
               }
          }
     }//for
}
 
//一个点代表一个槽
struct ListNode {//代表每行的节点
    int element;
    Position next;
};
 
//一个点代表槽中的一个链表上的一个点
struct HashTbl {
    int TableSize;
    Position *TheList;//指针的指针，指向由于冲突形成的链表
};
 
 
//简单起见，这里用int整型，然后是除法散列
int Hash(int key, int TableSize) {
    return key % TableSize;//TableSize得是素数
}
 
//素数的分布很均匀，所以这里耗时平均应该差不多，找到第一个比它大的素数
int NextPrime(int x) {
    if (prime[x]) {
        return x;
    } else {
        NextPrime(x + 1);
    }
}
 
//初始化, 返回的是一个指针
HashTable InitalizeTable(int TableSize) {
    if (TableSize <= 0) {
        puts("Table size is too small");
        return NULL;
    }
 
    HashTable H = (HashTable)malloc(sizeof(struct HashTbl));
    if (H == NULL) {
        puts("Out of space");
    }
     
    //指定List的一个数组，由于List是一个指针，因此结果为指针的数组
    H->TableSize = NextPrime(TableSize);
    H->TheList = (List *)malloc(sizeof(List) * H->TableSize);
    if (H->TheList == NULL) {
        puts("Out of space");
    }
 
    //List是带头结点的
    H->TheList[0] = (List)malloc(H->TableSize * sizeof(struct ListNode));
    if (H->TheList[0] == NULL) {
        printf("Out of space");
    }
 
    for (int i = 0; i < H->TableSize; i++) {
        H->TheList[i] = H->TheList[0] + i;
        H->TheList[i]->next = NULL;
    }
    return H;
}
 
//返回的是一个指针，指向包含Key的那个单元
Position Find(int key, HashTable H){
    Position p;
    //先找到所在行
    List L = H->TheList[Hash(key, H->TableSize)];
    p = L->next;
    while (p != NULL && p->element != key) {
        p = p->next;
    }
    if (p == NULL) {
        return L;
    } else {
        return p;
    }
}
 
//如果没有找到，就插入，如果找到了，就什么也不做
void Insert(int key, HashTable H) {
    Position pos,  NewCell;
    pos = Find(key, H);
    if (pos->element != key) {//Key没找到，所以要插入
        NewCell = (List)malloc(sizeof(struct ListNode));
        if (NewCell == NULL) {
            puts("Out of space");
        } else {
            List L = H->TheList[Hash(key, H->TableSize)];
            NewCell->next = L->next;//新节点放在表的最前面
            NewCell->element = key;
            pos->next = NewCell;//表头指向新加入的节点，新节点成为了第一个节点
        }
        printf("%d插入表中\n", key);
    } else {
        printf("%d已经存在, 无需重复插入.\n", key);
    }
}
 
//如果找到该值，原来的节点用一个空节点替换，销毁原来节点，如果没找到，什么也不做
void Delete(int key, HashTable H) {
    Position pos, NewCell;
    pos = Find(key, H);
    if (pos->element == key) {
        NewCell = H->TheList[Hash(key, H->TableSize)];
        while (NewCell->next != pos) {
            NewCell = NewCell->next;
        }
        NewCell->next = pos->next;
        free(pos);
        printf("%d删除成功\n", key);
    } else {
        printf("%d不存在, 无法删除\n", key);
    }
}
 
inline void Find_Description(Position p, int key) {
    if (p->element == key) {
        printf("小伙子，%d查找成功了\n", key);
    } else {
        printf("%d不存在表中\n", key);
    }
}
 
int main() {
     Seive_Prime() ;
 
     HashTable table = InitalizeTable(20);
     printf("表的大小是；%d\n", table->TableSize );//23
 
     Position pos = NULL;
 
     //先插入5个元素
     Insert(89, table);
     Insert(18, table);
     Insert(49, table);
     Insert(58, table);
     Insert(69, table);
      
     //测试可以找到的
     pos = Find(69, table);
     Find_Description(pos, 69);
 
     //测试找不到的
     pos = Find(25, table);
     Find_Description(pos, 25);
      
     //测试删除有的key
     Delete(69, table);
 
     //测试删除没有的key
     Delete(25, table);
 
     return 0;
}
{% endhighlight %}

第二种是：开放寻址法，所有元素都存放在散列里，查找一个元素时，要检查所有的表项，直到找到所需的元素，或者最终发现元素不在表中。不像在链接法中，这里没有链表，也没有元素存放在散列表外。 当要插入一个元素时，可以连续地检查散列表的个各项，直到找到一个空槽来放置这个元素为止。检查顺序可以是线性的，可以是二次平方的，也可以是双散列的，还有很多如：再散列，可扩散列等。

整个开放寻址法思路都是一样的，变得是散列函数，所以只写了平方探测的代码，其他两个稍微改改就好了。

线性探测法：第一次冲突移动1个单位，再次冲突时，移动2个，再次冲突，移动3个单位，依此类推。

它的散列函数是：H(x) = (Hash(x) + F(i)） mod TableSize, 且F(0) = 0.

假如散列函数是：Hash(x) = x mod 10(表的大小应该是素数，这里是为了简单)

散列函数将关键字{89， 18， 49， 58， 69}插入一个散列表中。

下边模拟线性探测：

89， 无冲突， OK;

18， 无冲突， OK；

49， 和89冲突，然后顺移一个单位，到地址0，可以插入，OK

58， 和18冲突，顺移一个单位，到地址0，还是冲突，顺移2个单位，到1，可以插入，OK;

69，和89冲突，顺移一个单位，到地址0， 还是冲突，顺移2个单位，到1， 冲突，顺移3个单位，到2，可以插入，OK

最后结果如图：

![Image](/uploads/2011/08/线性探测.jpg)
 从图中，可以明显的看到，线性探测存在“一次聚集“的问题，就是在表中存放是全部挨着的。这样在后期的插入过程中，会造成开销很大。


平方探测 可以避免上边提到的一次聚集：第一次冲突时移动1个单位，再次冲突时，移动4（2的平方）个单位，还冲突，移动9个单位，依此类推。F(i) = i * i , 于是可以很快推出：实现函数是：F(i) = F( i – 1) + 2i + 1.（稍微化简一下便知），这样可以避免每次的乘法和除法，而用加法代替，是一种加速手段。

线性探测中提到的例子，在这里应该不用模拟了吧，结果是：
![Image](/uploads/2011/08/平方探测.jpg)

对于线性探测，如果几乎很满，后期插入时表的性能会降低，对于平方探测情况，情况更糟：一旦表被填满超过一半，当表的大小不是素数时甚至在表被填满一半之前，就不能保证一次找到一个空单元了。这是因为最多有表的一半可以用于解决冲突的备选位置（证明过程看书吧）。

另外，表的大小是素数也非常重要。如果表的大小不是素数，则备选单元的个数可能会锐减。例如，如果表的大小是16，那么备选单元只能在距散列值1,4或9距离处。

平方探测排除了一次聚集，但是散列到同一位置上的元素将探测相同的备选单元，这叫做二次聚集，二次聚集理论上是个小缺陷，它一般要引起另外的少于一半的探测。另外，双散列可以排除这个遗憾，不过要花费另外的一些乘法和除法。

啰嗦完了上代码：平方探测的插入，查找，删除操作。

{% highlight c %}
/*平方探测法代码实现
输出：
——————————————
表的大小是；11
89插入成功
18插入成功
49插入成功
58插入成功
69插入成功
位置4上的元素是：69
 
69删除成功
69不存在或者已被删除，删除失败
69插入成功
 
*/
 
#include<iostream>
#include<cstdio>
#include<cstdlib>
using namespace std;
 
struct HashTbl;
typedef struct HashTbl * HashTable;
typedef unsigned int Position;
 
typedef struct HashEntry Cell;
//用来表示这行是合法的，空的，还是删掉了
enum KindOfEntry {Legitimate, Empty, Deleted};
 
struct HashEntry {
    int element;
    enum KindOfEntry Info;
};
 
struct HashTbl {
    int TableSize;
    Cell *TheCells;
};
 
const int maxn = 400000; // maxn : size 
bool prime[maxn]; // true : prime number
 
//Sieve Prime素数筛法, 先预处理好，否则很费时间
//单独判定素数是非常耗时间的
void Seive_Prime() {//判素,素数筛选 
     memset(prime, true, sizeof(prime));
     prime[0] = prime[1] = false;
     for (int i = 2; i * i < maxn; i++) {
          if (prime[i]) {
               for (int j = i * i; j < maxn ; j += i) {
                    prime[j] = false;
               }
          }
     }//for
}
 
 
//是除法散列
Position Hash(int key, int TableSize) {
    return key % TableSize;//TableSize得是素数
}
 
//素数的分布很均匀，所以这里耗时平均应该差不多，找到第一个比它大的素数
int NextPrime(int x) {
    if (prime[x]) {
        return x;
    } else {
        NextPrime(x + 1);
    }
}
 
//初始化, 返回的是一个指针
HashTable InitalizeTable(int TableSize) {
    if (TableSize <= 0) {
        puts("Table size is too small");
        return NULL;
    }
 
    HashTable H = (HashTable)malloc(sizeof(struct HashTbl));
    if (H == NULL) {
        puts("Out of space");
        return NULL;
    }
    //例子中，10不是素数，如果想测试，可以在这里改一改，下边这句改为：
    //H->TableSize = TableSize;
    H->TableSize = NextPrime(TableSize);
    H->TheCells = (HashEntry *)malloc(sizeof(Cell) * H->TableSize);
    if (H->TheCells == NULL) {
        puts("Out of space!");
        return NULL;
    }
 
    //每个单元的Info域都设置为Empty
    for (int i = 0; i < H->TableSize; i++) {
        H->TheCells[i].Info = Empty;
    }
    return H;
}
 
/*返回Key在散列表中的位置，如果Key不出现，那么Find返回最后的单元 ,上边讨论过
所以表的大小至少要是表中元素个数的两倍，这样平方探测才能实现
*/
Position Find(int key, HashTable H){
    Position pos;
    int num = 0;
    pos = Hash(key, H->TableSize);
    while (H->TheCells[pos].Info != Empty && H->TheCells[pos].element != key) {
        pos += 2 * (++num) - 1;
        if (pos >= (Position)(H->TableSize)) {//不要取模，取模真的很慢
            pos -= H->TableSize;
        }
    }
    return pos;
}
 
/*没找到就插入，修改状态，写入值*/
void Insert(int key, HashTable H) {
    Position pos;
    pos = Find(key, H);
    if (H->TheCells[pos].Info != Legitimate) {
        H->TheCells[pos].Info = Legitimate;
        H->TheCells[pos].element = key;
        printf("%d插入成功\n", key);
    } else {
        printf("%d已存在，无需插入", key);
    }
}
 
/*这里的删除并不是真正的删除，只是状态标记为删除而已
标准的删除这里不能实行，因为相应的单元可能已经引起过冲突，
元素绕过它存在了别处。例如，如果删除 89，那么实际上所有的
Find例程都不能正常运行，所以这里的删除是懒惰删除*/
void Delete(int key, HashTable H) {
    Position pos;
    pos = Find(key, H);
    if (H->TheCells[pos].Info == Legitimate) {
        H->TheCells[pos].Info = Deleted;
        printf("%d删除成功\n", key);
    } else {
        printf("%d不存在或者已被删除，删除失败\n", key);
    }
}
 
int main() {
     Seive_Prime() ;
 
     HashTable table = InitalizeTable(10);
     printf("表的大小是；%d\n", table->TableSize );//11
 
     //先插入5个元素
     Insert(89, table);
     Insert(18, table);
     Insert(49, table);
     Insert(58, table);
     Insert(69, table);
 
     //测试可以找到的
     Position  pos = Find(69, table);
     printf("位置%u上的元素是：%d\n", pos, table->TheCells[pos].element);
 
     Delete(69, table);
     Delete(69, table);
     Insert(69, table);
 
     return 0;
}
{% endhighlight %}

双散列是：双重散列借用两个散列函数，F(i) = i  * hash2(X)，这个公式意思是：我们将第二个散列函数应用到X并在距离hash2(X)，2 * hash2(X)等处探测。如果双散列正确实现，模拟表明：预期的探测次数几乎和随机冲突解决方法的情形相同，所以理论上双散列很有吸引力。它的散列函数是：h(k, i) = (h1(k) + ih2(k)) mod m，它的结果是不定的，有冲突时，移动随机个单位，但是确实可以很好的解决冲突。

图就不上了，有冲突，就随机找位置。