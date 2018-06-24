---
title: 字符串模式匹配及KMP算法分析与实现
urlname: string-pattern-match-kmp
date: 2018-06-23 15:14:25
copyright: true
mathjax: true
tags:
- 算法设计
categories:
- 算法设计
---

## 字符串基础知识

根据相关资料，字符串定义如下：字符串是由零个或者多个字符组成的有限序列，其中零个字符的串称为空串。字符串的主要存储方式有定长顺序存储（对应于C/C++中栈空间）、堆分配存储（动态申请内存）以及块链存储（链表节点为一个定长小块，多个小块依次组成一个实际串）。

字符串模式匹配是指给定两个串$S_1,S_2$，要求找出串$S_1$中与串$S_2$相等的子串。目前模式匹配算法思路主要有朴素模式匹配算法、KMP算法等（更多的算法可见参考资料）。本文依次分析实现两种算法。

## 朴素模式匹配算法

朴素模式匹配算法是最简单最容易想到的算法，也是最暴力的算法。其基本思路是对于主串$S_1$与模式串$S_2$分别设置两个下标 $i,j$。假设主串比较的起始位置为$startPos$，则置$i=startPos,j=0$，依次比较从 $i$ 和从 $j$ 下标开始的字符序列是否相等，如果比较到模式串最后一个位置相等则返回找到的结果，否则将 $i$ 置为上次起始位置的下一个下标（$i=startPos+1$），$j$ 置为模式字符串起始位置（$j=0$）。具体实现代码如下：

```cpp
#include <string>
using std::string;

int BrutePatternMatch(string mainString, string patternString, size_t startPos)
{
        size_t i = startPos, j = 0;
        while (i < mainString.size() && j < patternString.size()) {
                if (mainString[i] == patternString[j]) {
                        ++i; ++j;
                }
                else {
                        startPos += 1;
                        i = startPos; j = 0;
                }
        }
        if (j >= patternString.size()) {
                return int(startPos);
        }
        else {
                return -1;
        }
}
```

以上朴素的字符串模式匹配算法可以做进一步优化，比如 $i$ 定位到的位置必须保证剩下的子串长度大于等于模式串的长度。

## KMP算法

### KMP算法基本原理

观察朴素模式匹配算法，每次失配后会导致下标 $i,j$ 均发生回溯，使模式串起始位置（$j=0$）对齐到主串的$startPos+1$位置重新开始下一轮比较。但是我们在 $i,j$ 发生失配时是已知$[0,(j-1)]$与$[i-j,i-1]$之间的字符串是相等，如果我们能找到这部分相等字符子串的某些规律使得我们在对齐主串与模式串的下一轮比较起始位置时能够使 $i$ 尽量多跳几步（即$i=startPos+k,k>1$）甚至使得 $i$ 不发生回溯，那么便可以优化算法的时间复杂度了。根据这种思路得到的算法就是KMP算法了，KMP算法是由D.E.Knuth、J.H.Morris和V.R.Pratt共同发现的，取名字的首字母便得到KMP算法简称。假设主串长度为n模式串长度为m则该算法的时间复杂度为$O(n+m)$。空间复杂度由于增加一个 next 数组为$O(m)$。

如下图为一张KMP算法说明图：

![KMP](/images/kmp-algorithm-pattern.png)

红框内为第一次发生失配的的字符，蓝框为已经匹配相等的子串。如果按照朴素模式匹配算法，则此时对齐位置如下图所示：

![KMP](/images/brute-pattern-algorithm-step.png)

此时显然依旧会发生失配，需要再次将模式串往后移以对齐。但是如果保持 $i$ 不移动，一直移动模式串使得可以再次启动从 $i$ 开始的比较，则会得到如下图所示的对齐位置：

![KMP](/images/kmp-algorithm-step.png)

如上图所示，我们字符串可以直接开始从 $i$ 启动下一轮比较，此时 $i$ 没有发生回溯，只是 $j$ 发生回溯以对齐比较位置。如果我们希望能够一次就可以使得对齐位置移动到如上图所示的位置该如何处理呢？这便是$next$数组需要完成的工作了。比如上述匹配失配发生在$j=7$的位置，如果此时我们令$next[7]=2$，则我们可以当第 7 号（下标从 0 开始）元素发生失配时直接将 $j$ 的值置为$j=next[j]$便可以完成不回溯 $i$ 的情况下一次性对齐主串和模式串使得下一轮比较可以从当前失配 $i$ 开始。以上便是KMP算法的基本思路了，剩下的就是如何得到这个$next$数组了。

### KMP算法Next数组计算

在移动模式串对齐主串的过程中，我们先只观察已经匹配的部分，如下图红框内的字符字串所示：

![KMP](/images/kmp-algorithm-next.png)

我们可以看到在逐步滑动模式串时，会比较主串中已匹配字符子串（图中$ababcab$）的真后缀部分与模式串中已匹配字符子串（和主串中一样）的真前缀部分，我们在滑动过程中发现真前缀和真后缀相等时下标 $i$ 会再次指向上次失配的位置（即图中红框外字符 $c$），如果滑动使得真前缀真后缀不相等会导致 $i$ 回溯至当前失配的前方且必定导致模式串继续向前滑动直到找到一个真前缀与真后缀相等的位置，此时从失配位置（图中红框外字符 $c$）继续比较。如果此时 $i,j$ 对应的字符相等则比较下一个，如果不等则可以重复上述步骤，只是现在已匹配字符子串发生变化（图中在比较主串红框外c再次发生失配，此时已匹配子串为 $ab$）。

通过上述的分析我们可以得知 $next$ 数组的含义是在某个字符上发生失配后已匹配部分字符串的真前缀和真后缀部分最长相等长度（由于下标从0开始，最长是因为在模式串滑动时找到第一个真前缀真后缀相等部分是最长的）。以上是基本的 $next$ 数组含义。接下来就是具体如何计算该数组了。

根据上述分析，$next$ 数组是已匹配字符部分的最大前后缀相等（不包含自身）长度，因此最朴素的想法是针对模式串中的每一段字符分别暴力求解最长前后缀相等长度，但此时时间复杂度过高,可能高达$O(m^3)$。如果我们仔细分析问题会发现求最长前后缀长度也是一个模式匹配问题，只不过此时主串与模式串均为同一个字符串且从主串的第1个字符（下标从0开始）开始匹配。如下图是一个模式串的 $next$ 数组：

![KMP](/images/kmp-next-array.png)

如上图所示，我们依次分析。以 $i,j$ 代表主串和模式串下标。当 $j=0$ 发生失配时，此时已匹配长度为0，因此不应发生回溯而是 $i$ 向前走一步，但在实际编程中为了统一处理需要特殊处理这种情况，因此此处暂时使用问号，后期表明如何设置一个特殊值以方便编程；当 $j=1$ 发生失配时，此时已匹配长度为1，滑动一步即可开始比较，即从头开始比较，此时 $next[1] = 0$；当 $k=j+1$ 时，此时已知 $next[j]$ , 即 $[0, next[j])$ 和 $[j-next[j],j)$之间的字符串相等，如果当模式串的第 $next[j]$ 个字符与第 $j$ 个字符相等时，可推知 $[0, next[j]]$ 和  $[j-next[j],j]$ 之间的字符串相等，可得到 $next[k]=next[j]+1$，如果不等，如下图所示：

![KMP不等](/images/kmp-next-not-equal.png)

此时只能在该区间缩小范围继续查找，从而有缩小区间真前缀串必定是 $[0, next[j])$ 串的真前缀，上图所示第 $next[j]=4$ 个字符不等于第 $j=9$ 个字符，在缩小范围时需要保证  $[0, nextIndex)$ 与  $[j-nextIndex,j)$ 之间的字符串依旧保持相等(其中$nextIndex < next[j]$)，否则此时比较第 $nextIndex$ 个字符与第 $j$ 个字符已经失去意义。而两个区间的字符串又分别是 $[0, next[j])$ 字符串的真前缀真后缀，此时回到了求某个字符串的最长相等真前缀真后缀问题，这在上一步已经解决了，即本问题是一个递推问题，此时 $nextIndex=next[next[j]]$，即反复递推，最后递推到找到满足相等条件或者递推结束。根据上述思路可以得到计算 $next$ 数组的代码。

```cpp
#include <vector>
#include <string>
using std::vector;
using std::string;

vector<int> CalculateNextArray(string patternString)
{
        vector<int> nextArray(patternString.size(), -1);
        if (patternString.size() <= 1) {
                return nextArray;
        }
        nextArray[1] = 0; /*只有一个匹配的字符子串真前缀真后缀均为空*/
        int i = 1, j = 0;
        for ( ; i < int(patternString.size()) - 1; i++) {
                j = nextArray[i]; /*获取真前缀下一个下标*/
                if (patternString[i] == patternString[j]) {
                        nextArray[i+1] = j + 1;
                }
                else {
                        if (j == 0) {
                                nextArray[i+1] = 0;
                        }
                        else {
                                while (1) {
                                        j = nextArray[j];
                                        if (j == 0) {
                                                if (patternString[i] == patternString[j]) {
                                                        nextArray[i+1] = j + 1;
                                                }
                                                else {
                                                        nextArray[i+1] = 0;
                                                }
                                                break;
                                        }
                                        else if (patternString[i] == patternString[j]){
                                                nextArray[i+1] = j + 1; break;
                                        }
                                }
                        }
                }
        }
        return nextArray;
}
```

从上述代码可以看出代码重复判断 $j==0$ 的情况，如果可以统一处理可以使得代码更加简洁，这便需要用到前面提到过的 $next[0]$，如何设置该值呢？观察代码，我们可以发现当 $j==0$是有两种情况，一种是此时 $i,j$ 对应的字符相等，此时 $next[i+1]=j+1=0+1=1$，当不等时 $next[i+1]=0=-1+1$，如果我们让 $j==0$ 和字符不等时将 $j$ 置为 $-1$，结合上述思路修改代码如下：

```cpp
#include <vector>
#include <string>
using std::vector;
using std::string;

vector<int> CalculateNextArrayImprove(string patternString)
{
        vector<int> nextArray(patternString.size(), -1);
        if (patternString.size() <= 1) {
                return nextArray;
        }

        nextArray[0] = -1; /*第0个赋为-1*/
        int i = 0, j = 0;
        for ( ; i < int(patternString.size()) - 1; i++) {
                j = i; /*获取真前缀下一个下标*/
                while (1) {
                        j = nextArray[j];
                        if (j == -1 || patternString[i] == patternString[j]) {
                                nextArray[i+1] = j + 1; break;
                        }
                }
        }
        return nextArray;
}
```

上述代码是否还可以进一步优化呢？观察代码我们可以发现每次 $j$ 都重置为 $j=i$，然后才进行迭代查找，实际在一次计算完毕后 $j$ 已经保存了下一轮计算所需的结果，因此可做进一步优化，代码如下：

```cpp
#include <vector>
#include <string>
using std::vector;
using std::string;

vector<int> CalculateNextArrayUltimate(string patternString)
{
        vector<int> nextArray(patternString.size(), -1);
        if (patternString.size() <= 1) {
                return nextArray;
        }

        nextArray[0] = -1; /*第0个赋为-1*/
        int i = 0, j = -1;
        for ( ; i < int(patternString.size()) - 1; i++) {
                while (1) {
                        if (j == -1 || patternString[i] == patternString[j]) {
                                j += 1;
                                nextArray[i+1] = j; break;
                        }
                        j = nextArray[j];
                }
        }
        return nextArray;
}
```

上述代码只是优化掉了一行赋值代码，实际上上面代码依旧可以进行优化，考虑如下图一种情况：

![KMP不等](/images/kmp-next-array-improve.png)

在第 $2,3,5,6,7,8,10,11$ 个字符一旦失配后，如果 $j$ 回溯到 $next[j]$ 时依旧会失效，因为这两个下标的字符是一样的，因此如果在发现第 $j$ 与第 $next[j]$ 字符相等时，可以接着往前回溯，直到不相等。由于递推是从前往后，因此我们只需把值赋为上一个值即可，优化代码如下：

```cpp
#include <vector>
#include <string>
using std::vector;
using std::string;

vector<int> CalculateNextArrayUltimateImprove(string patternString)
{
        vector<int> nextArray(patternString.size(), -1);
        if (patternString.size() <= 1) {
                return nextArray;
        }

        nextArray[0] = -1; /*第0个赋为-1*/
        int i = 0, j = -1;
        for ( ; i < int(patternString.size()) - 1; i++) {
                while (1) {
                        if (j == -1 || patternString[i] == patternString[j]) {
                                j += 1;
                                if (patternString[j] == patternString[i+1]) {
                                        nextArray[i+1] = nextArray[j];
                                }
                                else {
                                        nextArray[i+1] = j;
                                }
                                break;
                        }
                        j = nextArray[j];
                }
        }
        return nextArray;
}
```

### KMP算法实现

既然KMP算法Next数组已经计算得到，则写出KMP模式匹配算法也是非常容易的，考虑 $next[0]=-1$，有如下模式匹配算法：

```cpp
#include <string>
using std::string;

int KMP(string mainString, string patternString, size_t startPos)
{
        vector<int> nextArray = CalculateNextArrayUltimate(patternString);
        int i = startPos, j = 0;
        for ( ; i < int(mainString.size()) && j < int(patternString.size()); ) {
                /*相等i，j均向前推，j==-1表明不存在真前缀真后缀相等，i，j也往前推*/
                if (j == -1 || mainString[i] == patternString[j]) {
                        ++j; ++i;
                }
                else {
                        j = nextArray[j];
                }
        }

        if (j >= int(patternString.size())) {
                return i - j; /*返回起始位置*/
        }
        else {
                return -1;
        }
}
```

## 算法测试

### Next数组计算代码测试

使用多个案例测试Next数组计算代码，测试代码如下：

```cpp
#include <vector>
#include <string>
#include <iostream>
using std::cout;
using std::endl;
using std::vector;
using std::string;

int main()
{
        /*问号代表值未确定*/
        string testStrings[] = {"ababcabababc", /*?,0,0,1,2,0,1,2,3,4,3,4*/
                                "ababaaababaa", /*?,0,0,1,2,3,1,1,2,3,4,5*/
                                "babababaa",    /*?,0,0,1,2,3,4,5,6*/
                                "aaab",         /*?,0,1,2*/
                               };

        vector<vector<int>> correct = {
                                       {-1,0,0,1,2,0,1,2,3,4,3,4},
                                       {-1,0,0,1,2,3,1,1,2,3,4,5},
                                       {-1,0,0,1,2,3,4,5,6},
                                       {-1,0,1,2}
                                      };

        for (size_t i = 0; i < sizeof(testStrings)/sizeof(testStrings[0]); i++) {
                /*调用具体的函数*/
                vector<int> result = CalculateNextArray(testStrings[i]);

                cout << (result == correct[i] ? "Correct" : "Wrong") << endl;
        }

        return 0;
}
```

### KMP算法测试

KMP算法测试代码如下所示：

```cpp
#include <vector>
#include <string>
#include <iostream>
using std::cout;
using std::endl;
using std::vector;
using std::string;

int main()
{
        /*第一个主串，第二个模式串*/
        vector<vector<string>> testStrings = {
                                              {"ababcabcaabcbaabc", "ababcabababc"},
                                              {"ababcabcaabcbaabc", "cabc"},
                                              {"HA", "HAHAHA"},
                                              {"WQN", "WQN"},
                                              {"ADDAADAADDAAADAAD", "DAD"},
                                              {"BABABABABABABABABB", "BABABB"},
                                             };
        vector<int> correct = {-1,
                                4,
                               -1,
                                0,
                               -1,
                               12
                               };

        for (size_t i = 0; i < testStrings.size(); i++) {
                /*调用具体的函数*/
                int result = KMP(testStrings[i][0], testStrings[i][1], 0);

                cout << (result == correct[i] ? "Correct" : "Wrong") << endl;
        }

        return 0;
}
```



## 参考资料

[字符串](https://zh.wikipedia.org/zh-hans/%E5%AD%97%E7%AC%A6%E4%B8%B2)

[子串](https://zh.wikipedia.org/zh-hans/子串)

[字符串模式匹配算法](http://dsqiu.iteye.com/blog/1700312)

[克努斯-莫里斯-普拉特算法](https://zh.wikipedia.org/wiki/%E5%85%8B%E5%8A%AA%E6%96%AF-%E8%8E%AB%E9%87%8C%E6%96%AF-%E6%99%AE%E6%8B%89%E7%89%B9%E7%AE%97%E6%B3%95)

[数据结构](https://book.douban.com/subject/2024655/)
