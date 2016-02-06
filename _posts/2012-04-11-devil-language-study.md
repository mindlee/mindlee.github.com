---
title: 魔王语言小窥
author: Wei Li
excerpt: 魔王，就我所知，是世界上最苦逼的职业之一。魔王大都童年悲惨，魔王大都相貌奇葩，魔王大都自以为是，魔王大都死不悔改，魔王大都死于非命。
meta_description: 魔王，就我所知，是世界上最苦逼的职业之一。魔王大都童年悲惨，魔王大都相貌奇葩，魔王大都自以为是，魔王大都死不悔改，魔王大都死于非命。
layout: post
permalink: /2012/04/11/devil-language-study/
views:
  - 5794
categories:
  - 技术启蒙
  - 算法学习
tags:
  - C/C++
  - 数据结构
  - 算法
---
魔王，就我所知，是世界上最苦逼的职业之一。魔王大都童年悲惨，魔王大都相貌奇葩，魔王大都自以为是，魔王大都死不悔改，魔王大都死于非命。比如[比克大魔王](http://www.ahww.or.jp/ani/picco/picco.htm)，刚来地球就撞坏了脑子。数码宝贝里七大魔王，被几个小学生就给收拾了。魔王多悲剧，魔王多可怜，小朋友们长大了一定不要当魔王。在严蔚敏《数据结构题集》里封印着一个苦逼魔王，脑子还被门夹过了，每天只知道“天上一只鹅，地上一只鹅“。所以，魔王通常脑子也不大好使。
![Image][1]

事情是这样子的，学妹不会数据结构课程设计，当学长的怎么能不帮学妹呢，所以就帮忙写了第一个版本，可惜老师太骚，一上来就多层括号，我怎么能接受这样的挑衅，于是写了第二个版本，第三个版本……希望此文能够一劳永逸地解决后辈们关于魔王语言的问题。另外，下边的代码是C和C++的混合体，不要用纯C的编译器，可以试试 [C-Free](http://www.programarts.com/cfree_ch/download.htm) 或者VC++2010。

<hr/>

魔王语言解释

[问题描述]

有一个魔王总是使用自己的一种非常精练而又抽象的语言讲话，没有人能听得懂，但他的语言是可以逐步解释成人能听懂的语言，因为他的语言是由以下两种形式的规则由人的语言逐步抽象上去的：
1.  α 转换为 β1β2…βm
2. （θδ1δ2…δn） 转换为 θδnθδn－1… θδ1θ

在这两种形式重，从左到右均表示解释。试写一个魔王语言的解释兄，把他的话解释成人能听得懂的话。
 
[基本要求]

用下述两条具体规则和上述规则形式（2）实现。设大写字母表示魔王语言的词汇；小写字母表示人的语言词汇；希腊字母表示可以用大写字母或小写字母代换的变量。魔王语言可含人的词汇。
1. B 转换为 tAdA
2. A 转换为 sae
 

[测试数据]

B（exnxgz）B解释成tsaedsaeezegexenehetsaedsae
若将小写字母与汉字建立下表所示的对应关系，则魔王说的话是：“天上一只鹅地上一只鹅鹅追鹅赶鹅下鹅蛋鹅恨鹅天上一直鹅地上一只鹅”。 

t d s a e z g x n h

天 地 上 一只 鹅 追 赶 下 蛋 恨
 
[实现提示]

将魔王地语言自右至左进栈，总是处理栈顶字符。若是开括号，则逐一出栈，讲字母顺序入队列，直至 闭括号出栈，并按规则要求逐一出队列再处理后入栈。其他情形较简单，请读者思考应如何处理。应首先实现栈和队列地基本操作。
 
[选作内容]

1. 由于问题地特殊性，可以实现栈和队列的顺序存储空间共享。
2. 代换变量的数目不限，则在程序开始运行时首先读入一组第一种形式的规则，而不是把规则固定在程序中（第二种规则只能固定在程序中）。
<hr/>

此题目实际上有两种解释，以“a(B)e”为例：

1. 先规则1，后规则2，也就是：a(B)e → a(tsaedsae)e → atetatstdtetatste

2. 先规则2，后规则1，也就是：a(B)e → aBe → atsaedsaee

这两种解释的实现，技术上一样样的，我的代码都是按照第2种来的，下边就是我可怜的小程序了：

<hr/>

### 1）单层括号处理（简单魔王语言）

第一个版本只支持单层括号，比如：(zgx)an(ane)，我画了张图，应该很好理解：
![Image][2]

{% highlight cpp %}
/*
*   Title:      魔王语言 
*   Created on：2012-4-9
*   @author：    酷~行天下 
*   @mail:      mindlee@me.com
*   @homepage   http://mindlee.net
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cctype> 
#include<stack>
#include<queue>
using namespace std;
 
stack<char> S;
queue<char> Q;
char mo[1000];
 
//初始化数据 
void init() {
    memset(mo, 0, sizeof(mo));
    while (!S.empty()) {
        S.pop();
    }
    while (!Q.empty()) {
        Q.pop(); 
    }
}
 
int main() {
    printf("请输入魔王语言：");
    init();
     
    while (gets(mo)) {
        //第一步，入栈 
        for (int i = strlen(mo) - 1; i >= 0; i--) {
            S.push(mo[i]);
        }
         
        printf("魔王语言解释是："); 
        while (!S.empty()) {
            char ch = S.top();
            S.pop();
 
            if (ch == 'A') {
                printf("sae");
            } else if (ch == 'B') {
                printf("tsaedsae");
            } else if (islower(ch)) {
                printf("%c", ch);
            } else if (ch == '(') {
                ch = S.top();//第一个字符
                S.pop(); 
                char tmp = ch;
                if (tmp == ')') continue;//排除掉只有（）时的情况 
 
                while (true) {//括号内所有字符入队列 
                    ch = S.top();
                    S.pop();
                    if (ch == ')') {
                        break;
                    }
                    Q.push(ch);
                }
 
                //交替重新入栈 
                S.push(tmp); 
                while (!Q.empty()) {
                    S.push(Q.front());
                    S.push(tmp);
                    Q.pop();
                }
            } else  {
                puts("输入不合法啊，亲！！");
            }
        }
        printf("\n\n请输入魔王语言："); 
        init();
    }
    return 0;
}
/*
请输入魔王语言：()
魔王语言解释是：
 
请输入魔王语言：(e)
魔王语言解释是：e
 
请输入魔王语言：(e)(e)
魔王语言解释是：ee
 
请输入魔王语言：(aen)B(xn)
魔王语言解释是：anaeatsaedsaexnx
 
请输入魔王语言：a(B)a
魔王语言解释是：atsaedsaea
*/
{% endhighlight %}

### 2）多层括号处理（标准魔王语言）

第二个版本是第一版的加强版，支持嵌套输入。Joshua Bloch大叔说“空格是靠不住的，而括号是从来不说谎的。”括号吗？放马过来吧。从图示可以看出，过程上比第一个版本稍微复杂了些，于是：
![Image][3]

{% highlight c %}
/*
*   Title:      魔王归来_Stack 
*   Created on：2012-4-10
*   @author：    酷~行天下 
*   @mail:      mindlee@me.com
*   @homepage   http://mindlee.net
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cctype> 
#include<climits> 
#include<stack>
#include<queue>
using namespace std;
 
stack<char> S, S1, S2;
string str;
char devil[10000];
char person[10000];
bool contain_bracket = true;
 
//初始化数据 
void init() {
    memset(devil , 0, sizeof(devil ));
    memset(person , 0, sizeof(person ));
    str.clear();
    while (!S.empty()) S.pop();
    while (!S1.empty()) S1.pop();
    while (!S2.empty()) S2.pop();
}
 
//打印字符结果 
void print_letter () {
    printf("魔王语言解释是：");
    int len = strlen(person );
    for (int i = 0; i < len; i++) {
        char ch = person [i];
        if (ch == 'A') {
            printf("sae");
        } else if (ch == 'B') {
            printf("tsaedsae");
        } else if (islower(ch)) {
            printf("%c", ch);
        } else  {
            printf("%c", ch);
        }
    }
}
 
//打印汉字结果 
void print_person () {
    printf("\n魔王表达意思是: ");
    int len = strlen(person );
    for (int i = 0; i < len; i++) {
        char ch = person [i];
        switch(ch) {
            case 't' : printf("天"); break;
            case 'd' : printf("地"); break;
            case 's' : printf("上"); break;
            case 'a' : printf("一只"); break;
            case 'e' : printf("鹅"); break;
            case 'z' : printf("追"); break;
            case 'g' : printf("赶"); break;
            case 'x' : printf("下"); break;
            case 'n' : printf("蛋"); break;
            case 'h' : printf("恨"); break;
            case 'A' : printf("上一只鹅"); break;
            case 'B' : printf("天上一只鹅地上一只鹅"); break; 
            default : printf("%c", ch);; break;
        }
    }
    puts("");
}
     
int main() {
    puts("-----------------------------------------------");
    puts("提示：输入中只能包含 tdsaezgxnh、A、B字符。"); 
    puts("-----------------------------------------------\n");
    printf("请输入魔王语言：");
    init();
     
    while (cin >> str) {
        string::iterator it = str.begin();
        while (it != str.end()) {//过滤字符串中特例 
            if (*it == '(' && *(it + 1) == ')') {//过滤挨着的() 
                str.erase(it, it + 2);
                it = str.begin(); 
            } else if (*it == '(' && isalpha(*(it + 1)) && *(it + 2) == ')'){
                str.erase(it + 2);//过滤括号内只有一个字符,比如(e) 
                str.erase(it);
                it = str.begin();
            } else {
                it++;
            }
        }   
         
        if (str.find('(') == ULONG_MAX) {//压根没括号了 
            strcpy(person, str.c_str());
            print_letter();
            print_person();
            printf("\n请输入魔王语言："); 
            init();
            continue;
        }
         
        strcpy(devil, str.c_str());
        //第一步，入栈 
        for (int i = strlen(devil ) - 1; i >= 0; i--) {
            S.push(devil [i]);
        }
         
        while (!S.empty()) {
            while (true) {//字符进入S1 
                char ch = S.top();
                S.pop();
                if (ch == ')') break;
                S1.push(ch);
            }
              
            while (!S1.empty()) {//从S1进入S2 
                char ch = S1.top();
                S1.pop();
                if (ch == '(') break;
                S2.push(ch);
            }
             
            char tmp = S2.top();
            S2.pop();
            S.push(tmp);
            while (!S2.empty()) {//S2放回初始栈S 
                char ch = S2.top();
                S2.pop();
                S.push(ch);
                S.push(tmp);
            }
         
            while (!S1.empty()) {//S1放回初始栈S 
                char ch = S1.top();
                S1.pop();
                S.push(ch);
            }
             
            contain_bracket = false;
            memset(person, 0, sizeof(person));
            for (int i = 0; i < 10000; i++) {//存到数组&检测是否完结 
                if (!S.empty()) {
                    char ch = S.top();
                    S.pop();
                    if (ch == ')') {
                        contain_bracket = true;
                    }   
                    person [i] = ch;
                } else {
                    break;
                }
            }
             
            for (int i = strlen(person) - 1; i >= 0; i--) {
                S.push(person [i]);
            }
             
            if (!contain_bracket) {//如果没括号，跳出 
                break;
            }
        }//while 
         
        print_letter(); 
        print_person();
         
        printf("\n请输入魔王语言："); 
        init();
    }
    return 0;
}
/*
-----------------------------------------------
提示：输入中只能包含 tdsaezgxnh、A、B字符。
-----------------------------------------------
 
请输入魔王语言：()
魔王语言解释是：
魔王表达意思是:
 
请输入魔王语言：(())
魔王语言解释是：
魔王表达意思是:
 
请输入魔王语言：((()))
魔王语言解释是：
魔王表达意思是:
 
请输入魔王语言：(()())
魔王语言解释是：
魔王表达意思是:
 
请输入魔王语言：(e)
魔王语言解释是：e
魔王表达意思是: 鹅
 
请输入魔王语言：((e))
魔王语言解释是：e
魔王表达意思是: 鹅
 
请输入魔王语言：((e)(e))
魔王语言解释是：eee
魔王表达意思是: 鹅鹅鹅
 
请输入魔王语言：(aen)
魔王语言解释是：anaea
魔王表达意思是: 一只蛋一只鹅一只
 
请输入魔王语言：(aen)B(xn)
魔王语言解释是：anaeatsaedsaexnx
魔王表达意思是: 一只蛋一只鹅一只天上一只鹅地上一只鹅下蛋下
 
请输入魔王语言：((aen)B(xn))
魔王语言解释是：axanaxatsaedsaeaaaeaaana
魔王表达意思是: 一只下一只蛋一只下一只天上一只鹅地上一只鹅一只一只一只鹅一只一只
一只蛋一只
 
请输入魔王语言：a(B)a
魔王语言解释是：atsaedsaea
魔王表达意思是: 一只天上一只鹅地上一只鹅一只
 
请输入魔王语言：a(AB)a
魔王语言解释是：asaetsaedsaesaea
魔王表达意思是: 一只上一只鹅天上一只鹅地上一只鹅上一只鹅一只
*/
{% endhighlight %}
### 3）STL处理

题目要求必须用栈（stack）和队列（queue）这两个好基友，事实上，如果不用它们，只用 STL 的 string 更简单，纯粹的字符串处理而已，因为 cin 比 scanf 慢很多，所以下边的代码，能用 printf 和 scanf 的时候不用 cout 和 cin 。

{% highlight c %}
/*
*   Title:      魔王归来_STL
*   Created on：2012-4-10
*   @author：    酷~行天下 
*   @mail:      mindlee@me.com
*   @homepage   http://mindlee.net
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<cctype>
#include<climits> 
#include<stack>
#include<queue>
using namespace std;
 
string str;
 
//打印结果  
void print_result(string &str) {
    printf("魔王语言解释是：");
    for (string::iterator it = str.begin(); it != str.end(); it++) {
        if (islower(*it)) {
            cout << *it;
        } else if (*it == 'A') {
            printf("sae");
        } else if (*it == 'B') {
            printf("tsaedsae");
        } else {
            cout << *it;
        }
    }
     
    printf("\n魔王表达意思是: ");
    for (string::iterator it = str.begin(); it != str.end(); it++) {
        switch(*it) {
            case 't' : printf("天"); break;
            case 'd' : printf("地"); break;
            case 's' : printf("上"); break;
            case 'a' : printf("一只"); break;
            case 'e' : printf("鹅"); break;
            case 'z' : printf("追"); break;
            case 'g' : printf("赶"); break;
            case 'x' : printf("下"); break;
            case 'n' : printf("蛋"); break;
            case 'h' : printf("恨"); break;
            case 'A' : printf("上一只鹅"); break;
            case 'B' : printf("天上一只鹅地上一只鹅"); break; 
            default : cout << *it; break;
        }
    }
}
 
//括号内语言转换 
void translate_to_human_anguage(string &str, int left, int right) {
    string source_str = str.substr(left + 1, right - left - 1);
    string target_str  = source_str.substr(0, 1);//首字符
    for (string::iterator it = source_str.end() - 1; 
        it > source_str.begin(); it--) {
        target_str += *it + source_str.substr(0, 1);
    }
    str.replace(left, right - left + 1,  target_str);
}
 
//将所有括号展开 
void remove_bracket(string &str) {
    if (str.find('(') == ULONG_MAX) {
        return ;
    }
     
    int right = str.find(')');//从前到后找第一个')'
    int left = str.rfind('(', right);//找到和')'匹配的'('
    if (right == left + 1) {//剔除单纯的() 
        str.erase(left, 2);
    } else if (right == left + 2) {//括号内只有一个字符
        str.erase(right, 1); //先右后左，不然right就失效了 
        str.erase(left, 1);
    } else {//一般情况 
        translate_to_human_anguage(str, left, right);
    }
    remove_bracket(str);
}
 
int main(void) {
    puts("-----------------------------------------------");
    puts("提示：输入中只能包含 tdsaezgxnh、A、B字符。"); 
    puts("-----------------------------------------------\n");
    printf("请输入魔王语言：");
    str.clear();
     
    while (cin >> str) {
        remove_bracket(str);
        print_result(str);  
        printf("\n\n请输入魔王语言："); 
        str.clear();
    }
    return 0;
}
/*
-----------------------------------------------
提示：输入中只能包含 tdsaezgxnh、A、B字符。
-----------------------------------------------
 
请输入魔王语言：()
魔王语言解释是：
魔王表达意思是:
 
请输入魔王语言：(())
魔王语言解释是：
魔王表达意思是:
 
请输入魔王语言：((()))
魔王语言解释是：
魔王表达意思是:
 
请输入魔王语言：(()())
魔王语言解释是：
魔王表达意思是:
 
请输入魔王语言：(e)
魔王语言解释是：e
魔王表达意思是: 鹅
 
请输入魔王语言：((e))
魔王语言解释是：e
魔王表达意思是: 鹅
 
请输入魔王语言：((e)(e))
魔王语言解释是：eee
魔王表达意思是: 鹅鹅鹅
 
请输入魔王语言：(aen)
魔王语言解释是：anaea
魔王表达意思是: 一只蛋一只鹅一只
 
请输入魔王语言：(aen)B(xn)
魔王语言解释是：anaeatsaedsaexnx
魔王表达意思是: 一只蛋一只鹅一只天上一只鹅地上一只鹅下蛋下
 
请输入魔王语言：((aen)B(xn))
魔王语言解释是：axanaxatsaedsaeaaaeaaana
魔王表达意思是: 一只下一只蛋一只下一只天上一只鹅地上一只鹅一只一只一只鹅一只一只
一只蛋一只
 
请输入魔王语言：a(B)a
魔王语言解释是：atsaedsaea
魔王表达意思是: 一只天上一只鹅地上一只鹅一只
 
请输入魔王语言：a(AB)a
魔王语言解释是：asaetsaedsaesaea
魔王表达意思是: 一只上一只鹅天上一只鹅地上一只鹅上一只鹅一只
*/
{% endhighlight %}

[1]: /uploads/2012/04/比克大魔王.jpg
[2]: /uploads/2012/04/魔王语言.jpg
[3]: /uploads/2012/04/魔王归来_Stack.jpg
