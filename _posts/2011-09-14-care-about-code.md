---
title: 要写整洁代码
author: Wei Li
excerpt: 好的代码优雅高效，整洁的代码如同散文，读起来酣畅淋漓；而读坏的代码犹如陷入沼泽，目光所及，无比绝望。整洁的代码，堪称人见人爱的艺术品；就像武术有不同的流派，整洁代码同样有不同的流派，这里要分享的是一些适用于面向对象的武林秘籍。
meta_description: 好的代码优雅高效，整洁的代码如同散文，读起来酣畅淋漓；而读坏的代码犹如陷入沼泽，目光所及，无比绝望。整洁的代码，堪称人见人爱的艺术品；就像武术有不同的流派，整洁代码同样有不同的流派，这里要分享的是一些适用于面向对象的武林秘籍。
layout: post
permalink: /2011/09/14/care-about-code/
views:
  - 5445
categories:
  - 技术启蒙
tags:
  - C/C++
  - JAVA
  - 笔记
  - 读书
---
>程序写出来是为了让人看懂它的算法，附带告诉计算机如何执行。
>———Abelson & Sussman

好的代码优雅高效，整洁的代码如同散文，读起来酣畅淋漓；而读坏的代码犹如陷入沼泽，目光所及，无比绝望。整洁的代码，堪称人见人爱的艺术品；就像武术有不同的流派，整洁代码同样有不同的流派，这里要分享的是一些适用于面向对象的武林秘籍：

###一、清晰而有意义的命名。

####1）名副其实， 比如：

{% highlight c %}
int d; // 消逝的时间，以日计算
int elapsedTimeInDays;
{% endhighlight %}
第一种写法，看似简单，但是，不要忽视掉后面的注释，更多的时候，糟糕的命名需要更多的注释来说明，而能体现本意的名称能让人更容易理解和修改。

####2) 命名区分要有意义，比如：
{% highlight c %}
getActiveAccount();
getActiveAccounts();
getActiveAccountInfo();
{% endhighlight %}
这三个函数，除了作者本人，估计没人知道有什么区别。

####3) 使用能读得出来的名称，对比下面两段代码：

{% highlight c %}
//1)
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
    //…………
};
 
//2)
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;;
    private final String recordId = "102";
    //…………
};
{% endhighlight %}
如果不说genymdhms是代表生成日期，年、月、日、时、分、秒，你能猜到它什么意思吗，如果不知道这个函数名代表什么意思，那你又如何在后续代码中记住这个函数名，想想都痛苦，相反，第二种写法显而易见了：是用来生成时间戳的。

   这里还包含一层意思是：不要嫌麻烦就自造词，看看上边第一种代码的类名，妈的，又得猜了！

####4) 使用能搜索的词，具体例子：
{% highlight c %}
//1)
for (int j = 0; j < 34; j++) {
    s += (t[j] * 4) / 5;
}
 
//2)
int realDaysPerIdealDay = 4;
const int WORK_DAYS_PER_WEEK = 5;
int sum = 0;
for (int j=0; j < NUMBER_OF_TASKS; j++) {
    int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
    int realTaskWeeks = (realdays / WORK_DAYS_PER_WEEK);
    sum += realTaskWeeks;
}
{% endhighlight %}
如果从代码中间突然冒出一段第一种的代码，你知道它什么意思吗，34, 4, 5 各代表什么意思，天知道，虽然第二种代码，type多了点，但是，回去看看文章的第一句话；不仅如此，维护代码时（维护代码当然要搜索），类似纯数字这样的代码难以维护，如果维护出现差错，就不是多打几个字母的损失了。

一条可以参考经验是：

>“单字母名称仅用于短方法中的本地变量，名称长短应与其作用域大小相对应。”

要是在三四行的for循环里，比如for (i = 0; i < n; i++) ，将i换成一个长名字，那你太欠扁了.

####5) 一些约定俗称的用法

类名和对象应该是名词或名词短语，比如：Customer, WikiPage, Account, 和 AddressParser. 避免使用Manager, Processor, Data, or Info 这样的类名。类名不该是动词。

方法名应当是动词或动词短语，比如：postPayment, deletePage.

####6) 添加有意义的语境

将变量名：firstName, lastName, street, houseNumber, city, state, and zipcode等放到一起，你知道这是一组地址，但是，如果你在某个方法里看到一个孤零零的state变量呢？你还会认为这个是某个地址的一部分吗？

对比下面两段代码，感觉一下，哪个更舒服：

{% highlight c %}
//1)
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    } else if (count == 1) {
        number = "1";
        verb = "is";
        pluralModifier = "";
    } else {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
    String guessMessage = String.format(
        "There %s %s %s%s", verb, number, candidate, pluralModifier);
    print(guessMessage);
}
 
//2)
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;
    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format(
            "There %s %s %s%s", 
            verb, number, candidate, pluralModifier );
    }
 
    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }
 
    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
 
    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }
    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
{% endhighlight %}
###二、函数应该遵守的原则

####1) 函数要短小，要更短小；

好的函数应该20行封顶，一般多于十几行就该考虑分割函数了。譬如这个例子：

{% highlight c %}
//1)
public static String renderPageWithSetupsAndTeardowns(
    PageData pageData, boolean isSuite
    ) throws Exception {
    boolean isTestPage = pageData.hasAttribute("Test");
    if (isTestPage) {
        WikiPage testPage = pageData.getWikiPage();
        StringBuffer newPageContent = new StringBuffer();
        includeSetupPages(testPage, newPageContent, isSuite);
        newPageContent.append(pageData.getContent());
        includeTeardownPages(testPage, newPageContent, isSuite);
        pageData.setContent(newPageContent.toString());
    }
    return pageData.getHtml();
}
 
//2)
public static String renderPageWithSetupsAndTeardowns(
    PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageData))
        includeSetupAndTeardownPages(pageData, isSuite);
    return pageData.getHtml();
}
{% endhighlight %}
虽然第一种已经够短，为什么不更短些写呢，譬如缩短成第二种；

####2) 一个函数只做一件事，譬如下边这个例子：
{% highlight c %}
import java.util.*;
public class GeneratePrimes {
    public static int[] generatePrimes(int maxValue)
    {
        if (maxValue >= 2) // the only valid case
        {
            // declarations
            int s = maxValue + 1; // size of array
            boolean[]
            // initialize array to true.
            for (i = 0; i < s; i++)
                f[i] = true;
            // get rid of known non-primes
            f[0] = f[1] = false;
            // sieve
            int j;
            for (i = 2; i < Math.sqrt(s) + 1; i++)
            {
                if (f[i]) // if i is uncrossed, cross its multiples.
                {
                    for (j = 2 * i; j < s; j += i)
                        f[j] = false; // multiple is not prime
                }
            }
            // how many primes are there?
            int count = 0;
            for (i = 0; i < s; i++)
            {
                if (f[i])
                    count++; // bump count.
            }
            int[] primes = new int[count];
            // move the primes into the result
            for (i = 0, j = 0; i < s; i++)
            {
                if (f[i]) // if prime
                    primes[j++] = i;
            }
            return primes; // return the primes
        }
        else // maxValue < 2
            return new int[0]; // return null array if bad input.
    }
}
 
//2)
public class PrimeGenerator
{
    private static boolean[] crossedOut;
    private static int[] result;
    public static int[] generatePrimes(int maxValue)
    {
        if (maxValue < 2)
            return new int[0];
        else
        {
            uncrossIntegersUpTo(maxValue);
            crossOutMultiples();
            putUncrossedIntegersIntoResult();
            return result;
        }
    }
 
    private static void uncrossIntegersUpTo(int maxValue)
    {
        crossedOut = new boolean[maxValue + 1];
        for (int i = 2; i < crossedOut.length; i++)
            crossedOut[i] = false;
    }
 
    private static void crossOutMultiples()
    {
        int limit = determineIterationLimit();
        for (int i = 2; i <= limit; i++)
            if (notCrossed(i))
                crossOutMultiplesOf(i);
    }
 
    private static int determineIterationLimit()
    {
        double iterationLimit = Math.sqrt(crossedOut.length);
        return (int) iterationLimit;
    }
 
    private static void crossOutMultiplesOf(int i)
    {
        for (int multiple = 2*i;
            multiple < crossedOut.length;
            multiple += i)
            crossedOut[multiple] = true;
    }
 
    private static boolean notCrossed(int i)
    {
        return crossedOut[i] == false;
    }
 
    private static void putUncrossedIntegersIntoResult()
    {
        result = new int[numberOfUncrossedIntegers()];
        for (int j = 0, i = 2; i < crossedOut.length; i++)
            if (notCrossed(i))
                result[j++] = i;
    }
 
    private static int numberOfUncrossedIntegers()
    {
        int count = 0;
        for (int i = 2; i < crossedOut.length; i++)
            if (notCrossed(i))
                count++;
        return count;
    }
}
{% endhighlight %}
例子1可以改写为例子2，generatePrimes函数被切分为declarations, initializations和sieve等区段，这就是函数做事太多的明显征兆，只做一件事的函数无法被合理地切分为多个区段。当然，上边第一段还有其他糟糕的缺陷，譬如注释过多，命名太差等。

####3) 每个函数一个抽象层级

简单说，就是，让代码拥有自顶向下的阅读顺序，这样，每个函数后面都跟着位于下一个抽象层级的函数，这样在查看函数列表时，就能循抽象层级向下阅读了，这个叫做向下规则。

举个例子，大概就是：

程序要求：要实现A，就先实现B，然后再实现C，再实现D。

就可以，函数1：要实现A就实现B；      函数2：要实现B就实现C；       函数3：要实现C就实现D

####4) 函数参数，没有最佳，其次是一，再次是二，避免三参数

includeSetupPage()要比includeSetupPageInfo(newPage-Content)易于理解，参数与函数名处于不同的抽象层级，它要求你了解目前并不特别重要的细节，这个很烦人。从测试的角度，编写能确保各种参数的各种组合运行正常的测试用例非常困难，相反，如果没有参数，那就小菜一碟了。然而，并不是说让所有函数都没有参数，事实上这是不可能的，很多转换函数都必须要有参数，这里要说的是，尽量压缩函数参数，以防后患。

####5) 抽离Try/Catch代码块

Try/Catch非常实用，但是却搞乱了代码结构，把错误处理与正常流程混为一谈，最好把try和catch代码块的主体部分抽离出来，另外形成函数。看看下边这种写法，是不是好多了。

{% highlight c %}
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    }
    catch (Exception e) {
        logError(e);
    }
}
 
private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}
 
private void logError(Exception e) {
    logger.log(e.getMessage());
}
{% endhighlight %}
来看看一个非常漂亮的，遵循上边所有原则的代码：（实在太漂亮了）

{% highlight c %}
package fitnesse.html;
 
import fitnesse.responders.run.SuiteResponder;
import fitnesse.wiki.*;
 
public class SetupTeardownIncluder {
    private PageData pageData;
    private boolean isSuite;
    private WikiPage testPage;
    private StringBuffer newPageContent;
    private PageCrawler pageCrawler;
 
    public static String render(PageData pageData) throws Exception {
        return render(pageData, false);
    }
 
    public static String render(PageData pageData, boolean isSuite)
        throws Exception {
            return new SetupTeardownIncluder(pageData).render(isSuite);
    }
 
    private SetupTeardownIncluder(PageData pageData) {
        this.pageData = pageData;
        testPage = pageData.getWikiPage();
        pageCrawler = testPage.getPageCrawler();
        newPageContent = new StringBuffer();
    }
 
    private String render(boolean isSuite) throws Exception {
        this.isSuite = isSuite;
        if (isTestPage())
            includeSetupAndTeardownPages();
        return pageData.getHtml();
    }
 
    private boolean isTestPage() throws Exception {
        return pageData.hasAttribute("Test");
    }
 
    private void includeSetupAndTeardownPages() throws Exception {
        includeSetupPages();
        includePageContent();
        includeTeardownPages();
        updatePageContent();
    }
 
    private void includeSetupPages() throws Exception {
        if (isSuite)
            includeSuiteSetupPage();
        includeSetupPage();
    }
 
    private void includeSuiteSetupPage() throws Exception {
        include(SuiteResponder.SUITE_SETUP_NAME, "-setup");
    }
 
    private void includeSetupPage() throws Exception {
        include("SetUp", "-setup");
    }
 
    private void includePageContent() throws Exception {
        newPageContent.append(pageData.getContent());
    }
 
    private void includeTeardownPages() throws Exception {
        includeTeardownPage();
        if (isSuite)
            includeSuiteTeardownPage();
    }
 
    private void includeTeardownPage() throws Exception {
        include("TearDown", "-teardown");
    }
 
    private void includeSuiteTeardownPage() throws Exception {
        include(SuiteResponder.SUITE_TEARDOWN_NAME, "-teardown");
    }
 
    private void updatePageContent() throws Exception {
        pageData.setContent(newPageContent.toString());
    }
 
    private void include(String pageName, String arg) throws Exception {
        WikiPage inheritedPage = findInheritedPage(pageName);
        if (inheritedPage != null) {
            String pagePathName = getPathNameForPage(inheritedPage);
            buildIncludeDirective(pagePathName, arg);
        }
    }
 
    private WikiPage findInheritedPage(String pageName) throws Exception {
        return PageCrawlerImpl.getInheritedPage(pageName, testPage);
    }
 
    private String getPathNameForPage(WikiPage page) throws Exception {
        WikiPagePath pagePath = pageCrawler.getFullPath(page);
        return PathParser.render(pagePath);
    }
 
    private void buildIncludeDirective(String pagePathName, String arg) {
        newPageContent
            .append("\n!include ")
            .append(arg)
            .append(" .")
            .append(pagePathName)
            .append("\n");
    }
}
{% endhighlight %}
###三、代码格式

####1）垂直格式

首先是垂直尺寸，源代码文件该有多大，多数Java源文件有多大，看看下边这张图：
![Image][1]
图中涉及7个不同项目，贯穿方块的直线两端显示这些项目中最小和最大的文件长度，方块表示在平均值以上或以下三分之一文件的长度。方块中间就是平均数。可以看到FitNesse项目文件平均尺寸是65行，最大是400行，最小是6行。Junit，FitNesses,Time and Money由相对较小文件组成，没有一个超过500行，多数小于200行。Tormat和Ant则有些达到数千行，将近一半文件长于200行。

这意味着：我们可以用大多数为200行、最长为500行的单个文件构造出出色的系统。尽管这不是什么不可以违背的原则，但是也应该乐于接受，短文件总是比长文件易于理解。

####2) 横向格式

一行代码该有多宽，大多数人都有自己的标准，看看典型的程序中代码行的宽度，上图：
![Image][2]
明显，大多数programmer更喜欢用短代码，图中可以看到，20~80个字符长度的代码分布非常平稳，更长到100或120也可以，这不是什么规则，也许有些人显示器一行可以显示200个字符，但是，培养一个好习惯，不要超过120个字符，想想有谁愿意看代码的时候左右拉滚动条呢。

####3）团队规则

每个程序员都有自己喜欢的格式规则，但如果在一个团队工作，就是团队说了算。一组开发者应当认同一种格式风格，每个成员都应该采用那种风格。软件应该拥有一以贯之的风格，而不是，一看就是一大票意见相左的个人所写。好的软件系统是由一系列读起来不错的代码文件组成。它们需要拥有一致和顺畅的风格。绝对不要用各种不同的风格来编写源代码，这样会增加复杂度。

最后附一个Robert C.Martin（代码整洁之道作者）的格式规则，当初作者和其团队制定的规则，可以参考作为范例：

{% highlight c %}
public class CodeAnalyzer implements JavaFileAnalysis {
    private int lineCount;
    private int maxLineWidth;
    private int widestLineNumber;
    private LineWidthHistogram lineWidthHistogram;
    private int totalChars;
 
    public CodeAnalyzer() {
        lineWidthHistogram = new LineWidthHistogram();
    }
 
    public static List<File> findJavaFiles(File parentDirectory) {
        List<File> files = new ArrayList<File>();
        findJavaFiles(parentDirectory, files);
        return files;
    }
 
    private static void findJavaFiles(File parentDirectory, List<File> files) {
        for (File file : parentDirectory.listFiles()) {
            if (file.getName().endsWith(".java"))
                files.add(file);
            else if (file.isDirectory())
                findJavaFiles(file, files);
        }
    }
 
    public void analyzeFile(File javaFile) throws Exception {
        BufferedReader br = new BufferedReader(new FileReader(javaFile));
        String line;
        while ((line = br.readLine()) != null)
            measureLine(line);
    }
 
    private void measureLine(String line) {
        lineCount++;
        int lineSize = line.length();
        totalChars += lineSize;
        lineWidthHistogram.addLine(lineSize, lineCount);
        recordWidestLine(lineSize);
    }
 
    private void recordWidestLine(int lineSize) {
        if (lineSize > maxLineWidth) {
            maxLineWidth = lineSize;
            widestLineNumber = lineCount;
        }
    }
 
    public int getLineCount() {
        return lineCount;
    }
 
    public int getMaxLineWidth() {
        return maxLineWidth;
    }
 
    public int getWidestLineNumber() {
        return widestLineNumber;
    }
 
    public LineWidthHistogram getLineWidthHistogram() {
        return lineWidthHistogram;
    }
 
    public double getMeanLineWidth() {
        return (double)totalChars/lineCount;
    }
 
    public int getMedianLineWidth() {
        Integer[] sortedWidths = getSortedWidths();
        int cumulativeLineCount = 0;
        for (int width : sortedWidths) {
            cumulativeLineCount += lineCountForWidth(width);
            if (cumulativeLineCount > lineCount/2)
                return width;
        }
        throw new Error("Cannot get here");
    }
 
    private int lineCountForWidth(int width) {
        return lineWidthHistogram.getLinesforWidth(width).size();
    }
 
    private Integer[] getSortedWidths() {
        Set<Integer> widths = lineWidthHistogram.getWidths();
        Integer[] sortedWidths = (widths.toArray(new Integer[0]));
        Arrays.sort(sortedWidths);
        return sortedWidths;
    }
}
{% endhighlight %}
参考书籍：Rotert C.Martin 著 韩磊 译.《代码整洁之道》[M].人民邮电出版社. 2010

[1]: /uploads/2011/09/clean-code1.jpg
[2]: /uploads/2011/09/clean-code-2.jpg