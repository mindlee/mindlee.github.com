---
title: 《C陷进与缺陷》读书笔记
author: Wei Li
excerpt:  全书总共171页，比起普通的经典C书籍，算是一本小册子了，讲了一些新手容易犯的错误，成书较早，有些写法，很老，现在估计不用了吧（还是有些编译器支持的写法?）. 
meta_description: 全书总共171页，比起普通的经典C书籍，算是一本小册子了，讲了一些新手容易犯的错误，成书较早，有些写法，很老，现在估计不用了吧（还是有些编译器支持的写法?）. 
layout: post
permalink: /2011/08/01/c-traps-and-pitfalls/
views:
  - 7809
categories:
  - 技术启蒙
tags:
  - C/C++
  - 笔记
  - 读书
---
全书总共171页，比起普通的经典C书籍，算是一本小册子了，讲了一些新手容易犯的错误，成书较早，有些写法，很老，现在估计不用了吧（还是有些编译器支持的写法?）.
不过很多在ACM过程中都碰到过，最后都教训深刻的记住了，所以书看得很快，不过，如果没看的话，还是有些新的错误可能以后会犯，收获还是很大。下边是笔记：

### P10
如果一个整型变量的第一个字符是数字0， 那么该常量将被视作八进制数，这个前几天还遇到，会报错，不知道原因，原来还有这样的规定，例如0175的含义是1*8^2 + 7 *8^1 + 5*8^0 = 145， ANSI C标准禁止这种写法，但是这个错误，编译器是不会报错的，所以，数组里为了数字之间对齐，不了解的话，很可能会犯这个错误，除非你写成091这样，才会报错，因为9在八进制里本身就是非法的。

### P24
这个代码：
{% highlight c %}
if (n < 3)
    return
logrec.data = x[0];
logrec.data = x[1];
logerc.data = x[2];
return 后边缺分号，但是编译不会报错，因为实际运行时
logrec.data = x[0] 当作了return语句的返回值
{% endhighlight %}
这种写法可能在某些老得编译器里不报错（函数声明可以省略返回值类型, 如果可以省略，那真的可以出错了），但是现在编译器，我用的C-FREE5和VC++2010报错报的非常厉害：函数声明必须有类型，另外函数本意是返回void，但是实际返回是int，所以类型不一致也会报错。

### P29
可能有时候，有些编辑器缩进能力不是很强，偶尔需要手动缩进下，下面的代码错误，
很可能会出现，本意是, else和第一个if匹配，规则是“else同一对括号内最近的未匹配的if结合”，所以实际运行时，else和第二个if匹配了，不鸟第一个if，所以好的编码习惯是多写括号，这样无论如何不会出现这个错误。

{% highlight c %}
if (x == 0) 
    if (y == 0) z = x - y;
else {
    z = x + y;
}
{% endhighlight %}

###P39
假定要把两个字符串s和t连接成单个字符串r，这时可以借助库函数strcpy和 strcat实现
于是：
{% highlight c %}
char *r
strcpy(r, s);
strcat(r, t);
{% endhighlight %}
答案是这样不行，r是一个指针，不能确定指向何处，而且所指的地址有多少空间也不确定
再试一次：

{% highlight c %}
char r[100];
strcpy(r, s);
strcat(r, t);
{% endhighlight %}

问题又出现了，数组大小必须为常量，但是这里不确定，如果r不够大，还是不行
于是改进：

{% highlight c %}
char *r, *malloc();
r = malloc(strlen(s) + strlen(t));
strcpy(r, s);
strcat(r, t);
{% endhighlight %}
但是还是错的
1. malloc有可能无法提供请求的内存，这种情况下malloc函数会返回一个空指针来作为”内存分配失败”事件的信号。(malloc用的较少，这些机制不太了解)
2. 给r分配的内存在使用完后应该及时释放，修订后的程序显式地给r分配的内存，为此就必须显式地释放内存(这个以前也没注意，额)
3. 这个代码malloc并没有分配足够的内存。 strlen计算的是字符数目，不包含’\0’，所以必须为r多分配一个字符空间
于是，最终代码：

{% highlight c %}
char *r, *malloc();
r = malloc(strlen(s) + strlen(t) + 1);
if (!r) {//分配失败 
    complain();
    exit(1);
}
strcpy(r, s);
strcat(r, t);
//一段时间后:
free(r);
{% endhighlight %}

### P46
计算边界时的一个小技巧：
计算x >= 16 && x <= 37有多少个元素？ 20 ？ 21 ？ 22？
正常人都会停下来，稍微推理一下，而不是非常确定一下子算出来，因为经验告诉我们直接减，很可能出错，所以要确定是+1，还是-1，还是直接减
作者的建议是：使用 左闭右开 的写法：x >= 16 && x < 38，这样，答案就是：38 – 16
如果统一了写法，就可以省去很多麻烦
所以for循环最好写：（话说，下边写法已经很普及了，只是上边的左闭右开，自己没想到）

{% highlight c %}
for (int i = 0; i < 10; i++) {
    a[i] = 10;
}
{% endhighlight %}
而不是下边的，因为上边的写法10 – 0 可以很快看出出循环次数

{% highlight c %}
for (int i = 1; i <= 9; i++) {
    a[i] = 0;
}
{% endhighlight %}

### P61
不小心发现书上写了一个很实用的库函数：#include 里边包含所有类型的
范围，比如INT_MAX代表int类型上限，INT_MIN代表int类型下限，我平常都宏定义:

{% highlight c %}
#define inf 0x7fffffff
{% endhighlight %}
这种写法可读性非常差，不是每个人对16进制非常熟悉
原来库函数直接有，见图，或者去[cplusplus](http://www.cplusplus.com/reference/clibrary/climits/)更详细：
![Image](/uploads/2011/07/climits.jpg)

### P76
简单的输入输出代码：

{% highlight c %}
#include<stdio.h>
int main() {
    int i;
    char c;
    for (i = 0; i < 5; i++) {
        scanf("%d", &c);
        printf("%d ", i);
    }
    printf("\n");
}
{% endhighlight %}
本意输出:0 1 2 3 4
某运行输出：0 0 0 0 1 2 3 4(大端机，小端机输出不同)
这里c被声明为char类型，而不是int类型
本例中，c存放的是int的低位部分，每次读入一个c时，i低端都会覆盖为0，i高端本来就位0,于是i被重新设置0，循环一直进行，直到输入结束。
我画了张图，来说明这里i和c的地址储存，和为什么会覆盖i低端：
![Image](/uploads/2011/07/xianjinyuquexianzhanlizi.jpg)

### P84
标准输入，标准输出，上代码：

{% highlight c %}
#include<stdio.h>
int main() {
    int c;
    while ((c = getchar()) != EOF)
        putchar(c);
}
{% endhighlight %}
变量c被声明为char类型, 所以无法容下所有可能字符,
宏定义里EOF是-1，而char类型里不存在-1，c无法容下EOF，所以这里的 != EOF无法比较
结果有两种可能：一、某些合法字符被”截断”后使得c与EOF相同，中途终止。二、c根本不可能取到EOF, 程序陷入一个死循环
### P90
还是上边的代码，有一个很有意思的题目：
如果去掉上边代码的头文件：#include，为了让程序运行，再加上：#define EOF -1
变为：

{% highlight c %}
#define EOF -1
int main() {
    int c;
    while ((c = getchar()) != EOF)
        putchar(c);
}
{% endhighlight %}
很多系统上，仍能运行，但是在某些系统运行会慢很多，为什么？
解释很牛逼：(不了解机制，怎么可能知道)
getchar()在stdio.h头文件里定义为宏，但是为了预防编程者粗心大意，在C语言很多库文件里都包含了getchar()函数（注意是函数）,所以，程序忘记加#include的后果是：
在应该用宏的地方，都用getchar()函数替换，而函数调用所导致的开销增多，putchar同理。
这个代码在我编译器上无法运行，可能有些可以，注意这个代码没有头文件。此时，编译器对getchar()一无所知，这种情况下，编译器会假定getchar是一个返回类型为整数的函数。(好霸气的编译器)

### P95
定义max(a, b)，我经常会用宏定义写：#define max(a, b) ((a) > (b) ? (a) : (b))
括号必须加，宏不知道优先级，只是赤裸裸的替换
下边这个例子来说明宏定义并不是函数：
{% highlight c %}
#include<stdio.h>
#define max(a, b) ((a) > (b) ? (a) : (b))
 
int main() {
    int x[4] = {2, 3, 1};
    int biggest = x[0];
    int i = 1;
    while (i < n) { 
        biggest = max(biggest, x[i++]);
    }
}
{% endhighlight %}
第一次迭代时, 上边语句被扩充为：

{% highlight c %}
biggest = ((biggest) > (x[i++]) ? (biggest) : (x[i++]));
{% endhighlight %}
首先biggest与x[i++]比较，此时i = 1，x[i] = 3；而biggest = 2
关系运算符结果为false， 因为i++的原因，比较完后I = 2；
此时赋值：x[i++]赋值给biggest，即biggest = x[2]， 然后i++原因，i的值变为3
可以看到，一次循环，i自增两次，所以宏定义和函数是不一样的，使用时应该避免使用这种写法。

### P98
使用宏的另外一个坏处：
宏展开可能会产生非常庞大的表达式，占用的空间远远超过编程者所期望的空间，比如
max(a, max(b, max(c, d)))展开后够写两三行
《[Effective C++](http://www.kuqin.com/effectivec2e/)》里, 好的编码习惯，第一条就是：
“尽量用编译器而不用预处理”
于是，max函数建议的写法是：
template

{% highlight c %}
inline const T& max(const T& a, const T& b)
{ return a > b ? a : b; }
{% endhighlight %}
可以用模板定义各种类型的比较，同时，内联函数可以保证速度。
如果是C++，C++库函数#include里包含了max(a, b)函数，直接用就可以了,上边的例子只是说明：好的编码习惯是多用inline函数，而不是define
PS：
实测可以发现，比大小max(),从效率上：
{% highlight c %}
if.else > 三元运算符 ? : >  #define > #include<cmath>里max() > 
inline int max(a, b)> int max(a, b)  
{% endhighlight %}   
if.else效率是三元运算符一倍左右，如果是ACM要提升效率的话，用if.else吧
如果是项目代码为了整洁，用自带max或者内联吧