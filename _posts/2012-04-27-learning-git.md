---
title: Git学习笔记
author: Wei Li
excerpt: 感叹我生不逢时，错过了版本控制的“史前黑暗”时代，刚刚接触版本控制，Linus就站在前边，笑嘻嘻的说：“版本控制？ 什么是版本控制 。。 ”。真希望Linus身退时，这样说过：“I'll be back.” ——施瓦辛格，《终结者》，1984。
meta_description: 感叹我生不逢时，错过了版本控制的“史前黑暗”时代，刚刚接触版本控制，Linus就站在前边，笑嘻嘻的说：“版本控制？ 什么是版本控制 。。 ”。真希望Linus身退时，这样说过：“I'll be back.” ——施瓦辛格，《终结者》，1984。
layout: post
permalink: /2012/04/27/learning-git/
views:
  - 9335
categories:
  - 技术启蒙
tags:
  - git
  - 软件
---
[Git](http://git-scm.com/) /’ɡɪt/,  Linus的第二个伟大作品。Git这个词有点儿意思，Linus自嘲说 “I’m an egotistical bastard,  and I name all my projects after myself. First Linux, now Git. “  感叹我生不逢时，错过了版本控制的“史前黑暗”时代，刚刚接触版本控制，Linus就站在前边，笑嘻嘻的说：“版本控制？ 什么是版本控制 。。 ”。真希望Linus身退时，这样说过：“I’ll be back.” ——施瓦辛格，《终结者》，1984。
![Image][1]

Git这个词本身很喜感，学Git更欢乐，因为书里没啥难懂的知识，看的时候，让人心花怒放，心想，啥时候看书这么快了。。写iMiss时，和东东用Github操作代码，非常方便，感叹遇上了好时代，一上手就有Git用，还不用给地主家交租，连看书都可以直接跳过像《CVS版本库到Git的迁徙》这样的章节。。Git实在是个好东西，它让你笑，它让你叫，它让你跳。

刚开始用时，遇到过些小麻烦，比如，commit内容写错了，merge到了错误的分支，最关键的是，不知道怎么用Git画出优雅的下水道等等。。。后来慢慢知道怎么解决了，于是，留点笔记吧，既是给自己看的，也是给别人参考的。。

如果你能看懂下边这堆命令（纯属瞎折腾），赶紧关掉这个页面，遛马去吧。。
{% highlight c %}
touch hello.txt
echo "first line in hello.txt" >> hello.txt
cat hello.txt
git add hello.txt
git commit -m "add heo.txt"
git log –oneline
git commit –amend -m "add hello.txt"   //修改最近一次的commit内容
git log –graph –oneline                //图示查看提交
rm hello.txt                           //从工作区删除hello.txt
git rm hello.txt                       //从暂存区删除hello.txt
git commit -m "delete hello.txt"
git reset –mixed HEAD^                 //版本库和暂存区恢复，工作区删除
ls                                     //无hello.txt
git ls-files                           //有hello.txt
git ls-tree HEAD                       //有hello.txt
git checkout hello.txt                 //在工作区恢复hello.txt
git reflog
git reset –hard HEAD@{1}               //取消恢复，又删了
{% endhighlight %}

后边提到的，工作区是指（working directory），暂存区是指（staging area，index 在这里），版本库是指（git directory，respository,  HEAD在这里），三者的关系如下图。标签是指（tag，也有书里译为“里程碑”）
![Image][2]

### Git基础配置：

配置名字和邮箱：

	git config –global user.name "your name"
	git config –global user.email "your email"

设置高亮颜色：

	git config –global color.ui "auto"      (Linux下)
	git config –global color.ui "always"    (Windows下)
 
设置文本编辑器：

	git config –global core.editor gedit.exe(常用哪个，设置哪个)

设置差异分析工具：

	git config –global merge.tool kdiff3    (合并工具有很多种)
 
设置别名：（commit → ci）（可以移植习惯，有人会把Git命令别名为CVS中的相应命令）
	
	git config –global alias.ci "commit"
 
GitHub配置名字和API接口：（故事要从[这里](http://help.github.com/win-set-up-git/)说起）
	
	git config –global github.user username_in_github
	git config –global github.token 972b0f310c66a61a0b59f949 (类似的哈希吗，具体去GitHub看看)
 
检查上述设置是否成功：

	git config –global –list

### 查看信息相关：

#### git log：

	git log –pretty=oneline                (可以显示完整哈希码)
	git log –-oneline                      (显示哈希吗前7位，上边的简化版)
	git log –oneline –graph –stat –decorate

这几个是我常用的参数，更多查看文档吧:    (git help log)

	–oneline：                             单行显示;
	–graph：                               图示显示提交历史;
	–stat：                                显示文件修改历史;
	–decorate：                            显示标签(Tag);

示例指令：
	
	git log –oneline –graph -3

#### git status -s

	示例运行结果： MM hello.txt （第一个 M 绿色，第二个 M 红色）

第一列字母M，版本库文件与暂存区文件差异

第二列字母M，工作区文件与暂存区文件差异
 
#### git reflog
	git reflog -5
	查看最近五次的操作记录，默认是HEAD

	git reflog show master –5
	查看master最近五次的操作记录
 

#### 其他查看指令：
	ls                                     查看本地文件
	cat hello.txt                          查看文件内容
	git ls-files                           查看暂存区文件
	git ls-tree HEAD                       查看版本库(HEAD)文件
	git ls-remote origin                   查看上游文件(远程版本库)
	gitk –all                              图形界面查看提交(图形化界面)
	du -sh .git                            查看版版本库的大小

### 添加和删除：

	git add -u     
	u → update, 以暂存区中有的文件为准, 更新暂存区文件; 如果是工作区新建的文件,则此文件不能添加到暂存区(already tracked files in the index rather than the working tree)

	git add .
	.代表所有文件，以工作区中有的文件为准，更新暂存区中相应文件; 包括工作区刚刚新建的文件。(update all tracked files in the current directory and its subdirectories.)

	git add -A
	A → all, 范围覆盖(暂存区 + 工作区)中所有文件，所有文件变化都提交到暂存区(包括新建，修改，删除文件)(files in the working tree in addition to the index.)

	rm hello.txt                           从工作区删除
	git rm hello.txt                       从暂存区删除
	git rm –cached <file>                  同上

### 反悔了：（悔棋，穿越）

#### git reset
	
	git reset HEAD — <paths>               
	用版本库中文件替换掉暂存区中文件夹;
	
	git reset –hard <commit>               
	版本库、暂存区、工作区全部一致，撤回到某个commit点;

示例：

	git reflog –5                          显示最近的5条操作记录
	git reset –hard HEAD@{2}               回到HEAD@{2}前的状态,即取消最近两步操作;
   
来个比较狠的例子，下面的两个操作(git reset –hard <commit> 和 git reset –hard HEAD@{<num>})，可以让你在任意两个点之间玩穿越，历史穿梭就是这么回事儿。。。

{% highlight java %}
$ git log –oneline –3                  //显示最近的三次commit记录
10ce10c second line in abc.txt
1fed225 first line in abc.txt
4916146 delete hello.txt
$ git reset –hard 4916146              //从现在(10ce10c)退回到4916146
HEAD is now at 4916146 delete hello.txt
$ git reflog –3                        //查看最近的操作记录
4916146 HEAD@{0}: reset: moving to 4916146
10ce10c HEAD@{1}: commit：second line in abc.txt
1fed225 HEAD@{2}: commit: first line in abc.txt
$ git reset –hard HEAD@{1}             //撤销最近的一次操作，即从4916146返回到10ce10c;
HEAD is now at 10ce10c second line in abc.txt
{% endhighlight %}
 
git reset –soft \<commit\> 只改变版本库，不改变暂存区和工作区;
示例：

	git reset –soft HEAD^        
	撤销刚才的commit，相当于git commit的反操作
	
	git reset –soft HEAD^^                 
	可以用来合并提交，此时git commit –m “……”可以覆盖原来最近两次的commit内容;

git reset –mixed \<commit\> 改变版本库和暂存区，不改变工作区;
实例：

	git reset –mixed HEAD^                 
	如果紧跟在git add + git commit后面执行这条命令,相当于git add + git commit 的反操作;

	git reset .                            
	将git add 命令更新到暂存区的内容撤出暂存区, 即用版本库的HEAD重置暂存区;

	git reset HEAD                         
	同上;

	git reset filename                    
	将暂存区中filename撤出暂存区, 相当于git add filename的反操作;
    
	git reset HEAD filename                
	同上
 
#### git checkout
git checkout <file> 用暂存区文件替换工作区文件，但并不是git add 的反向操作，因为如果新建一个文件，git add 到暂存区，利用这条命令是无法将将此文件移出暂存区的，试试下面的代码：

	echo "first line" > abc.txt
	git add abc.txt
	echo "second line" >> abc.txt
	cat abc.txt
	git checkout abc.txt
	cat abc.txt
<hr/>
	git checkout .
	会用暂存区内容刷新工作区，相当于取消本地所有修改;

	git checkout branch
	检出到分支，版本库，暂存区、工作区全部更新;

	git checkout branch – filename
	用branch中的文件替换暂存区 + 工作区中相应的文件;

	git checkout tagname
	HEAD头指针指向tagname;

	git checkout HEAD~1 — welcome.txt
	从历史中恢复文件;

	git revert HEAD
	反转上一条提交，但是会有两条记录，上一条记录 + 新的revert记录

### 改变历史：

	git cherry-pick master^
	从众多的提交中挑选出一个提交应用到当前的工作分支中

	git rebase –onto <newbase> since till
	将(since, till]嫁接到newbase上

示例：

	git rebase –onto C E^ F
	将（E^, F]，即[E，F]嫁接到C上
	
	git cherry-pick <commit>
	拣选操作， 将<commit>重新放到当前HEAD上； git rebase可以将连续的几个<commit>重放到<newbase>上， git rebase 作用上相当于git cherry-pick的加强版，都是为了改变历史；

	git rebase -i C
	交互式变基，手动编辑，可以试试，最方便 + 最强大的方法，不过要谨慎）
 
### 标签：（里程碑）

#### git tag

	git tag mytag                          创建轻量级标签
	git tag -m "First annoated tag." mytag2创建带说明的标签。
	git tag -d <tagname>                   删除本地tag
	git push origin mytag                  将mytag共享到上游版本库（默认是不共享的）
	git push origin:mytag2                 删除远程版本库标签mytag2
 

### 比较差异：

#### git diff

	git diff                               比较, 暂存区 Vs. 工作区;
	git diff HEAD                          比较, 版本库 Vs. 工作区;
	git diff –cached                       比较, 版本库 Vs. 暂存区;


### 忽略文件：

	cat > .gitignore << EOF
	> *.o
	> *.h
	> EOF
	git add .gitignore
	git status –ignored –s                 查看已忽略文件
	git add -f hello.h                     强制添加已忽略文件
 

### 其他一些指令：

	git clean –nd                          看看那些文件和目录会被删除
	git clean –fd                          清除工作区中未加入版本库的文件&目录
	git stash                              保存当前进度
	git blame README                       可以看到每次改变的来源
	git mergetool                          利用工具解决冲突
	git branch –r                          查看远程分支
	git commit –amend -m "modify last commit content"         修改上次的提交内容
<hr/>
好像没怎么关注分支（branch），还没写过啥大软件，分支不起来。。
结尾总得留点什么，上个故事吧。。
 
>“你能告诉我，我从这儿该走哪条路吗？”爱丽丝问

>“那多半儿要看你想去哪里。”猫说。

>“我不在乎去哪儿— —”爱丽丝说。

>“那么你走哪条路都没关系，”猫说。

>“— —只要能到这个地方就行，”爱丽丝解释。

>“奥，当然，你总能到个地方”，猫说，“只要你走得够远。”

>-- Lewis Carroll：《爱丽丝漫游奇境》

[1]: /uploads/2012/04/git_octocats.png
[2]: /uploads/2012/04/git_operation.png