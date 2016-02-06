---
title: 字符串匹配(String Matching)
author: Wei Li
excerpt: 相比较Rabin-Karp算法和KMP算法主要解决，少量长字符之间匹配问题。字典树主要用于解决大量短字符之间匹配问题。
meta_description: 相比较Rabin-Karp算法和KMP算法主要解决，少量长字符之间匹配问题。字典树主要用于解决大量短字符之间匹配问题。
layout: post
permalink: /2011/11/25/string-matching/
views:
  - 14906
categories:
  - 算法学习
tags:
  - 字符串匹配
  - 数据结构
  - 笔记
  - 算法
  - 算法导论
---
字符串 T = abcabaabcabac，字符串 P = abaa，判断P是否是T的子串，就是字符串匹配问题了，T 叫做文本（Text） ，P 叫做模式（Pattern），所以正确描述是，找出所有在文本 T = abcabaabcabac 中模式 P = abaa 的所有出现。字符串匹配的用处应该很明显，经常使用的全文查找功能，Ctrl + F，用的应该就是字符串匹配算法，更高级的还有DNA序列中搜寻特定模式等。
![Image][1]
模式 P 在文本 T 中出现一次，在位移 s = 3 处。如果用最朴素（Naive）的匹配算法，也可以解决，两个 for 循环搞定，代码倒是巨短，但是效率很低，因为有很多不必要的比较，朴素匹配算法，最坏情况下，运行时间为：O((n – m + 1)m)。

朴素算法代码实现：
{% highlight c %}
/*
运行结果：
_________________________
朴素算法，匹配位置是：7
*/
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
 
//朴素匹配算法
void NativeStringMatcher(const char *T, const char *P) {
    int n = strlen(T);
    int m = strlen(P);
    for (int j, i = 0; i < n - m; i++) {
        for (j = 0; j < m; j++) {
            if (T[i + j] != P[j]) {
                break;
            }
        }
        if (j == m) {
            printf("朴素算法，匹配位置是：%d\n", i + 1);
        }
    }
}
 
int main() {
    const char *T = "2359023141526739921";
    const char *P = "31415";
    NativeStringMatcher(T, P);
    return 0;
}
{% endhighlight %}

下面是四个高级算法，Rabin-Karp算法，Knuth-Morris-Pratt算法，字典树，AC自动机。

先验知识，记号与术语：

1. 用 Σ* 表示用字母表Σ中的所有有限长度的字符串的集合 ；

2. 字符串 x 的长度用 \|x\| 表示。

3. x 和 y 的连接表示为 xy，长度为\|x\| + \|y\|

4. x = yw，y 是 x 的前缀，w 是 x 的后缀

### 一、Rabin-Karp算法

Rabin-Karp算法由 Rabin 和 Karp 提出，预处理时间为 O（m），最坏情况下运行时间为O((n – m + 1)m)，似乎和朴素算法差不多，但是它最坏情况出现的几率太小，所以平均情况很好。Rabin-Karp算法的核心思想是通过对字符串进行哈稀运算（散列运算），即给文本中 模式长度 的字符串哈希出一个数值，开始只需比较这个数值即可，之后在数值的基础上再用朴素算法比较字符串，利用散列函数可以很容易的吧字母转化为数字，这里假定字符串就是数字字符。比如字符串 31415 对应于十进制的31415。

已知模式 P[1.……m]，设 p 表示其相应十进制数地值，类似地， 对于给定的文本T[1.……n]. 用 t$_s$ 表示长度为 m 的子字符串 T[s + 1 ‥ s + m]（ s = 0, 1, . . . , n – m）， t$_s$ = p 当且仅当 [s + 1 ‥ s + m] = P[1 ‥m]；因此s是有效位移当且仅当 t$_s$ = p，可以通过把 p 与每一个 t$_s$ 值进行比较。

可以用霍纳规则(Horner’s rule) 在Θ(m) 的时间内计算p的值 ：

$$
p = P[m] + 10 (P[m – 1] + 10(P[m – 2] + · · · + 10(P[2] + 10P[1]) ))
$$

类似地，可以在Θ(m)时间内，根据T[1..m]计算出$t_0$ 的值。为了在Θ(n – m) 时间内计算出剩余的值$t_1$, $t_2$, . . . , tn-m可以在常数的时间内根据$t_s$计算出$t_{s+1}$，总结出公式：

$$
t_{s + 1} = 10 （t_s – 10^{m-1} T[s + 1]） + T[s + m + 1]………………公式1
$$

单看公式很难理解，来个例子：如果m = 5，$t_s$ = 31415, 我们去掉高位数字T [s + 1] = 3，然后在加入一个低位数字T [s + 5 + 1]（假设为2)，得到：$t_{s+1}$ = 10(31415 – 10000 • 3) + 2 = 14152。

当然还有一个问题是，计算中 p 和 $t_s$ 的值可能太大，超出计算机字长，不能方便地进行处理。如果 p 包含m 个字符，那么， 关于在 p 上地每次算术运算需要“常数”时间这一假设就不合理了，幸运的是，对这一问题存在一个简单的补救方法，对一个合适的模 q 来计算 p 和 $t_s$ 的模，每个字符是一个十进制数，因为 p 和 t$_0$  以及 公式1 计算过程都可以对模 q 进行，所以可以在 Θ(m) 时间内计算出模 q 的 p 值，在 Θ(n – m + 1) 时间内计算出模 q 的所有 $t_s$ 值，通常选模 q 为一个素数，使得 10q 正好为一个计算机字长，单精度算术运算就可以执行所有必要的运算过程。 一般情况下，采用d进制的字母表{0, 1, . . . , d – 1}, 所选的 q 要满足 d * q < 字长，调整 公式1， 使其为：

$$
t_{s + 1} = （d（t_s – T[s + 1] * h） + T[s + m + 1]） mod  q
$$

其中的$h = d ^{m-1}$ (mod q)，但是加入模q后，由$t_s$ ≡ p (mod q)不能说明 $t_s$ = p. 但ts $\not\equiv$ p (mod q), 可以说明 $t_s$ ≠ p，因此当$t_s$ ≡ p (mod q)时， 再用朴素的字符串匹配算法验证$t_s$ = p。. 如果q足够大，可以期望伪命中很少出现。

![Image][2]

如图（a）是一个文本字符串，阴影部分长度为5，模13为7。b）所有长度为5的窗口都计算出了模13的值，当然，出现两个值匹配的地方，第一个是合法匹配，第二个是伪命中点，不管是合法的，还是伪的，都进行朴素匹配，当然不匹配的，直接按照（c）图往后推进即可。伪代码：

	RABIN-KARP-MATCHER(T, P, d, q)
	1 n ← length[T]
	2 m ← length[P]
	3 h ← dm-1 mod q
	4 p ← 0
	5 t0 ← 0
	6 for i ← 1 to m           ▹ Preprocessing.
	7     do p ← (dp + P[i]) mod q
	8        t0 ← (dt0 + T[i]) mod q
	9 for s ← 0 to n – m       ▹ Matching.
	10     do if p = ts
	11           then if P[1 ‥ m] = T [s + 1 ‥ s + m]
	12                   then print "Pattern occurs with shift" s
	13        if s < n – m
	14           then ts+1 ← (d(ts – T[s + 1]h) + T[s + m + 1]) mod q

{% highlight c %}
/*
运行结果：
_________________________
t1 = 8
t2 = 9
t3 = 3
t4 = 11
t5 = 0
t6 = 1
t7 = 7
匹配位置是：7
t8 = 8
t9 = 4
t10 = 5
t11 = 10
t12 = 11
t13 = 7
伪命中点：13
t14 = 9
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cmath>
#include<algorithm>
using namespace std;
 
//朴素匹配算法，用于RabinKarp调用
bool NativeStringMatcher(const char *T, const char *P) {
    int n = strlen(T);
    int m = strlen(P);
    for (int j, i = 0; i < n - m; i++) {
        for (j = 0; j < m; j++) {
            if (T[i + j] != P[j]) {
                break;
            }
        }
        if (j == m) {
            return true;
        }
    }
    return false;
}
 
//RabinKarp算法
void RabinKarp(const char *T, const char *P, int d, int q) {
    int n = strlen(T);
    int m = strlen(P);
 
    int h = 1;
    for (int i = 0; i < m - 1; i++) {
        h *= d;//pow可能会越界，所以用乘法
        if (h >= q) {
            h %= q;
        }
    }
 
    int p = 0;
    int t = 0;
    for (int i = 0; i < m; i++) {
        p = (d * p + (P[i] - '0')) % q;
        t = (d * t + (T[i] - '0')) % q;        
    }
 
    for (int i = 0; i < n - m; i++) {
        printf("t%d = %d\n", i + 1, t);
        if (p == t) {
            if (NativeStringMatcher(T + i, P)) {
                printf("匹配位置是：%d\n", NativeStringMatcher(T + i, P) + i);
            } else {
                printf("伪命中点：%d\n", i + 1);
            }
        } 
 
        if (i < n - m) {
            t = (d * (t - h * (T[i] - '0')) + T[i + m] - '0') % q;
            if (t < 0) {
                t += q;
            }
        }
    }
}
 
int main() {
    const char *T = "2359023141526739921";
    const char *P = "31415";
    RabinKarp(T, P, 10, 13);
    return 0;
}
{% endhighlight %}

### 二、Knuth-Morris-Pratt算法

仨人设计的算法，所以简称KMP算法，KMP算法预处理时间Θ（m），匹配时间Θ（n），KMP算法用到了一个辅助数组π[1，m]，这个数组记录模式与其自身的位移进行匹配的信息，这些信息可以避免在朴素匹配算法中的无用位移测试，KMP算法的精髓和高效之处全在这个辅助数组。
![Image][3]
比如这个例子，模式P和T匹配过程中，（a）中一个特定的位移 s 处，q = 5个字符已经匹配成功，但是第六个字符不匹配了，如果是朴素算法，位移s处无效，则接着到 s + 1处，但是明显的 s + 1 处是明显无效的，而如（b）图，s + 2前三个字符都可以匹配，所以很可能是匹配点。数组π记录的就是这些信息，比如对于P，上边的例子 π[ 5 ] = 3，则下一个可能的位移是s’= s + （q – π[q]），即s’= s + 2，也就是在匹配过程中，同时用π数组记录下一次可能匹配位置的信息。

上边例子，完整π数组的值：
![Image][4]

如果你能看懂上边的例子，那么代码就极好理解了，KMP算法伪代码，其中 COMPUTE-PREFIX-FUNCTION 过程是预处理来计算π数组的：

	KMP-MATCHER(T, P)
	1 n ← length[T]
	2 m ← length[P]
	3 π ← COMPUTE-PREFIX-FUNCTION(P)
	4 q ← 0                          ▹Number of characters matched.
	5 for i ← 1 to n                 ▹Scan the text from left to right.
	6      do while q > 0 and P[q + 1] ≠ T[i]
	7             do q ← π[q]    ▹Next character does not match.
	8         if P[q + 1] = T[i]
	9            then q ← q + 1      ▹Next character matches.
	10         if q = m                    ▹Is all of P matched?
	11            then print "Pattern occurs with shift" i – m
	12                 q ← π[q]    ▹Look for the next match.
 

	COMPUTE-PREFIX-FUNCTION(P)
	1 m ← length[P]
	2 π[1] ← 0
	3 k ← 0
	4 for q ← 2 to m
	5      do while k > 0 and P[k + 1] ≠ P[q]
	6             do k ← π[k]
	7         if P[k + 1] = P[q]
	8            then k ← k + 1
	9         π[q] ← k
	10 return π
	     KMP代码实现：

{% highlight c %}
/*
运行结果：
————————————————
匹配位置: 1
匹配位置: 12
*/
 
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
using namespace std;
 
//伪代码中的fail数组，用fail来表示
int fail[1000];
 
//预处理fail数组
void ComputePrefixFunction(char *P) {
    int m = strlen(P);
    memset(fail, 0, sizeof(fail));
    fail[0] = 0;
    int k = 0;
    for (int i = 2; i <= m; i++) {
        while (k > 0 && P[k] != P[i - 1]) {
            k = fail[k - 1];
        }
        if (P[k] == P[i - 1]) {
            k = k + 1;
        }
        fail[i - 1] = k;
    }
}
 
void KMPMatcher(char *T, char *P) {
    int n = strlen(T);
    int m = strlen(P);
 
    int q = 0;
    for (int i = 1; i <= n; i++) {
        while (q > 0 && P[q] != T[i - 1]) {
            q = fail[q - 1];
        }
 
        if(P[q] == T[i - 1]) {
            q = q + 1;
        }
 
        if(q == m) {
            printf("匹配位置: %d\n", i - m + 1);
            q = fail[q - 1];
        }
    }
}
 
int main() {
    KMPMatcher("123451233211234561234", "12345");
    return 0;
}
{% endhighlight %}

关于KMP算法，Matrix67的这篇文章不能错过[KMP算法详解](http://www.matrix67.com/blog/archives/115)。

### 三、字典树

字典树：又称为 Trie ，是一种用于快速检索的多叉树结构。如英文字母的字典树是一个26叉树。数字的字典树是一个10叉树。字典树的基本功能是用来查询某个单词在所有单词中出现次数的一种数据结构，它的插入和查询复杂度都为O(len)，Len为单词（前缀）长度，但是它的空间复杂度却非常高，如果字符集是26个字母，那每个节点的度就有26个，典型的以空间换时间结构。

相比较Rabin-Karp算法和KMP算法主要解决，少量长字符之间匹配问题。字典树主要用于解决大量短字符之间匹配问题。

特别地：和二叉查找树不同，在Trie树中，每个结点上并非存储一个元素。 在 Trie 树中查找一个关键字的时间和树中包含的结点数无关，而取决于组成关键字的字符数。HH师兄讲字典树时的一个例子，用she，he，his，hers 构成一棵字典树：

![Image][5]

[MatRush](http://hi.baidu.com/matrush/blog/item/52dbcf4940c5f2fd83025c1a.html)博客摘录：

	字典树特点：

	①利用串的公共前缀->节约内存。

	②根结点(root)不包含任何字母。

	③其余结点仅包含一个字母(非元素)。

	④每个结点的子结点包含字母不同。

	字典树查找过程：

	①在Trie树上进行检索总是始于根结点。

	②取得要查找关键词的第一个字母，并根据该字母选择对应的子树并转到该子树继续进行检索。

	③在相应的子树上，取得要查找关键词的第二个字母，并进一步选择对应的子树进行检索。

	④在某个结点处，关键词的所有字母已被取出，则读取附在该结点上的信息，即完成查找。

假如用包含26个字母的字符构造字典树，那么每个结点都最多有26个分支，匹配某个单词时，每个字符在字典树中对应一层，这样可以非常快速的查找，因为根据字符对应分支查找就可以了。字典树主要的时间花在预处理构造字典树上，设node为实际使用的结点数目，建树O(node * 26)，每次查询是O(length)，空间复杂度O(node * 26)。

字典树模板：（初学时，看的是 MatRush 的博文，所以模板源于此）

{% highlight c %}
//HDU 1251 代码，字典树模板
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
 
const int MAXN = 100010, MAXM = 11, KIND = 26;
//小写字母->26 ，大小混写->52，大小写+数字->62
int m;
struct node {
    char* s;
    int prefix;
    bool isword;
    node* next[KIND];
    void init() {
        s = NULL;
        prefix = 0;//前缀
        isword = false;
        memset(next, 0, sizeof(next));
    }
}a[MAXN*MAXM], *root;//根
 
void insert(node *root, char *str) {//插入
    node *p = root;
    for (int i = 0; str[i]; i++) {
        int x = str[i] - 'a';
        p->s = str + i;
        if (p->next[x] == NULL) {
            a[m].init();
            p->next[x] = &a[m++];
        }
        p = p->next[x];
        p->prefix++;
    }
    p->isword = true;
}
 
bool del(node *root, char *str) {//删除
    node *p = root;
    for (int i = 0; str[i]; i++) {
        int x = str[i] - 'a';
        if (p->next[x] == NULL) {
            return false;
        }
        p = p->next[x];
    }//for(i)
    if (p->isword) {
        p->isword = false;
    } else {
        return false;
    }
    return true;
}
 
bool search(node *root, char* str) {//查找
    node* p = root;
    for (int i = 0; str[i]; i++) {
        int x = str[i] - 'a';
        if (p->next[x] == NULL) {
            return false;
        }
        p = p->next[x];
    }//for(i)
    return p->isword;
}
 
int count(node *root, char *str) {//统计后缀
    node *p = root;
    for (int i = 0; str[i]; i++) {
        int x = str[i] - 'a';
        if (p->next[x] == NULL) {
            return 0;
        }
        p = p->next[x];
    }//for(i)
    return p->prefix;
}
 
int main() {
    m = 0;
    a[m].init();
    root = &a[m++];
    char str[MAXM];
 
    while (gets(str), strcmp(str, "")) {
        insert(root, str);
    }
 
    while (gets(str)) {
        printf("%d\n", count(root,str));
    }
}
{% endhighlight %}

###四、AC自动机

首先简要介绍一下AC自动机：Aho-Corasick automation，该算法在1975年产生于贝尔实验室，是著名的多模匹配算法之一。一个常见的例子就是给出n个单词，再给出一段包含 m 个字符的文章，让你找出有多少个单词在文章里出现过。要搞懂AC自动机，先得有模式树（字典树）Trie 和 KMP模式匹配算法 的基础知识。AC自动机算法分为 3 步：构造一棵Trie树，构造失败指针和模式匹配过程。

如果你对 KMP 算法和了解的话，应该知道 KMP算法 中的 next 函数（shift 函数或者 fail 函数，即上文的π）是干什么用的。KMP 中我们用两个指针 i 和 j 分别表示，A[ i – j + 1……i ] 与 B[1…….j ] 完全相等。也就是说，i 是不断增加的，随着 i 的增加 j 相应地变化，且 j 满足以 A[i] 结尾的长度为 j 的字符串正好匹配B串的前 j 个字符，当 A [ i + 1] ≠ B [ j + 1]，KMP 的策略是调整j的位置（减小 j 值）使得A [ i – j + 1……i ]与 B[1……j ] 保持匹配且新的 B [ j + 1 ] 恰好与 A [ i + 1 ]匹配，而 next 函数恰恰记录了这个 j 应该调整到的位置。同样AC自动机的失败指针具有同样的功能，也就是说当我们的模式串在Tire上进行匹配时，如果与当前节点的关键字不能继续匹配的时候，就应该去当前节点的失败指针所指向的节点继续进行匹配。

这里有一个帖子讲的非常详细，AC自动机，此贴足矣：[AC自动机算法详解](http://www.cppblog.com/mythit/archive/2009/04/21/80633.html)，AC自动机的关键概念是fail指针，上边那个例子的fail指针。
![Image][6]
PS：AC自动机常用来，解决少量长字符匹配大量短字符的问题（常常做辅助以解决更难的问题）

[HDU 2222  Keywords Search](http://acm.hdu.edu.cn/showproblem.php?pid=2222)，给你10000个单词(每个单词长度不大于50,由小写字母组成),现在给你一个长句子(长度1000000)，问出现了多少单词表里的单词，AC自动机算法练习题，实现代码：

{% highlight c %}
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cmath>
#include<algorithm>
using namespace std;
#define MAXN 10001
#define MAXM 51
#define KIND 26
 
struct node {
    int prefix;
    node *fail;
    node *next[26];
    void init() {
        prefix = 0;
        fail = NULL;
        memset(next, 0, sizeof(next));
    }
}*que[MAXN * MAXM], trie[MAXN * MAXM],  *root;
 
int cnt;
char keyword[MAXM];
char str[MAXN * 2];
 
void Insert(node *root, char *str) {
    node *ptr = root;
    for (int i = 0; str[i]; i++) {
        int x = str[i] - 'a';
        if (ptr->next[x] == NULL) {
            trie[cnt].init();
            ptr->next[x] = &trie[cnt++];
        }
        ptr = ptr->next[x];
    }
    ptr->prefix++;
}//insert
 
void Build(node *root) {
    int head = 0, tail = 0;
    root->fail = NULL;
    que[head++] = root;
    while (head != tail) {
        node *tmp = que[tail++];
        node *ptr = NULL;
        for (int i = 0; i < KIND; i++) {
            if (tmp->next[i] != NULL) {
                if (tmp == root) {
                    tmp->next[i]->fail = root;
                } else {
                    ptr = tmp->fail;
                    while (ptr != NULL) {
                        if (ptr->next[i] != NULL) {
                            tmp->next[i]->fail = ptr->next[i];
                            break;
                        }
                        ptr = ptr->fail;
                    }
                    if (ptr == NULL) {
                        tmp->next[i]->fail = root;
                    }
                }//if_else
                que[head++] = tmp->next[i];
            }//if
        }//for(i)
    }//while (head != tail) 
}//Build
 
int Query(node *root, char *str) {
    int ret = 0;
    node *ptr = root;
    for (int i = 0; str[i]; i++) {
        int x = str[i] - 'a';
        while (ptr->next[x] == NULL && ptr != root) {
            ptr = ptr->fail;
        }
        ptr = ptr->next[x];
        if (ptr == NULL) {
            ptr = root;
        }
        node *tmp = ptr;
        while (tmp != root && tmp->prefix != -1) {
            ret += tmp->prefix;
            tmp->prefix = -1;
            tmp = tmp->fail;
        }
    }//for(i)
    return ret;
}
 
 
int main() {
    int cas, n;
    scanf("%d", &cas);
    while (cas--) {
        // head = tail = 0;
        cnt = 0;
        trie[cnt].init();
        root = &trie[cnt++];
 
        scanf("%d%*c", &n);
        while (n--) {
            gets(keyword);
            Insert(root, keyword);
        }
        Build(root);//构造自动机
        scanf("%s", str);
        printf("%d\n", Query(root, str));
    }
    return 0;
}
{% endhighlight %}

[1]: /uploads/2011/11/字符串匹配pic.png
[2]: /uploads/2011/11/Rabin-Karp.png
[3]: /uploads/2011/11/KMP算法演示1.png
[4]: /uploads/2011/11/KMP算法演示2.png
[5]: /uploads/2011/11/字典树图示.png
[6]: /uploads/2011/11/AC自动机fail指针.png