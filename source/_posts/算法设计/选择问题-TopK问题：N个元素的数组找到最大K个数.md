---
title: 选择问题-TopK问题：N个元素的数组找到最大K个数
urlname: top-k-problem
date: 2018-04-16 23:05:40
tags:
- 算法设计
categories:
- 算法设计
---

## 问题描述

选择问题或者说TopK问题是指给一堆无序的含有N个元素的数组，找出里面的最大的K个数（也可以是找出最小的K个数）。在本问题的基础上可以加上诸如元素数量较多，无法完整放入整个内存中，此时常规的先排序的思路是无效的。TopK问题的解决思路主要有如下几种，分别介绍如下。

## 常规排序思路

当不存在内存限制时，最先想到就是采用常规的排序算法了，目前基于比较的算法最好能够优化到O(nlgn)。如果要求稳定性则可以采用归并排序，否则可以使用堆排序或者快速排序，具体排序算法可见本人博客：[各大排序算法的分析与实现](https://hsuloong.github.io/algorithms-design/sort-algorithm.html)

如果元素范围比较小还可以采用非比较的算法如计数排序，时间复杂度可以优化到O(N)。

## 部分排序思路

部分排序的思路是不对整个数组进行排序，只对K个元素进行排序，其基本思路如下：

    1. 先读入K个元素并以非递增顺序排序
    2. 读入新数据，如果新数据小于K个元素最小的数据，则直接丢弃，否则插入到正确的位置
    3. 在结束后保存下来的K个元素的数组即为最大的K个数

在进行排序可以使用插入排序、堆排序思想。插入排序较为简单，以下实现采用堆排序的算法代码。采用堆排序的思路时间复杂度为建堆O(K)+更新堆O((N-K)lgk)，合计为O(NlogK)。

```cpp
#include <vector>
using std::vector;

//end指向最后元素的下一个元素，和STL保持一致,最小堆
void PerDown(vector<int> &intVec, int index, int end)
{
        int tmp = intVec[index], i = index;
        while (2*i + 1 < end) {
                int child = 2*i+1;
                if (child + 1 < end && intVec[child] > intVec[child+1]) {
                        ++child;
                }
                if (intVec[child] < tmp) {
                        intVec[i] = intVec[child];
                        i = child;
                }
                else {
                        break;
                }
        }
        intVec[i] = tmp;
}

vector<int> TopK(vector<int> &intVec, int k)
{

        if (k > int(intVec.size()) || k <= 0) {
                return intVec;
        }
        vector<int> result(k);
        for (int i = 0; i < k; i++) {
                result[i] = intVec[i];
        }
        //建堆
        for (int i = int(result.size())/2; i >= 0; i--) {
                PerDown(result, i, k);
        }
        for (int i = k; i < int(intVec.size()); i++) {
                if (intVec[i] > result[0]) {
                        result[0] = intVec[i];
                        PerDown(result, 0, k);
                }
        }
        return result;
}
```

## 快速排序思路下的快速选择

快速排序会把比基准元素大的部分放到一边，其他放在另外一边，如果分割元素刚好处在分割K个最大元素的位置则可以直接返回结果。快速选择最好时间复杂度为O(N)，最坏情况发生在每次分割元素取的最大或最小值，使得元素全部被分到一边，此时递推式为T(N)=T(N-1)+O(N)=O(N<sup>2</sup>)。

由于TopK问题特点，将比基准元素大的一方放到左边，当基准元素下标为K是则停止算法，实现代码如下。

```cpp
#include <vector>
#include <algorithm>
using namespace std;

//会修改原数组，end和STL保持一致
void SelectCore(vector<int> &intVec, int start, int end, int k)
{
        int middle = start + (end-start)/2; 
        //选择中间元素作为分割元素，有一种称为五分化中项的中项的选择方式
        swap(intVec[middle], intVec[end-1]);
        int i = start, j = end-2;
        while (i <= j) {
                while (i < end && intVec[i] > intVec[end-1]) {
                        ++i;
                }

                while (j > start && intVec[j] < intVec[end-1]) {
                        --j;
                }

                if (i < j) {
                        swap(intVec[i], intVec[j]);
                        ++i; --j;
                }
                else {
			break;
		}
        }

        swap(intVec[i], intVec[end-1]);

        if (i == k) {
                return;
        }
        else if (i < k) {
                SelectCore(intVec,i+1, end, k);
        }
        else {
                SelectCore(intVec,start, i, k);
        }
}

vector<int> QuickSelect(vector<int> &intVec, int k)
{
        if (k > int(intVec.size()) || k <= 0) {
                return intVec;
        }

        SelectCore(intVec, 0, int(intVec.size()), k);

        vector<int> result(k, 0);
        copy(intVec.begin(), intVec.begin() + k, result.begin());
        return result;
}
```

## 算法测试

上述代码的测试程序如下：

```cpp
#include <vector>
#include <random>
#include <ctime>
#include <iostream>
#include <algorithm>
#include <chrono>
#include <functional>
using namespace std;
using namespace chrono;

int main()
{
        const int testNum[] = { 20, 100, 1000, 10000, 100000, 1000000, 10000000 };

        default_random_engine randEngine(unsigned(time(nullptr)));
        uniform_int_distribution<int> intDis(int(-10e4), int(10e4));

        int testTimes = 5;

        while ((testTimes--) > 0) {
                for (size_t i = 0; i < sizeof(testNum) / sizeof(testNum[0]); i++) {
                        vector<int> testData1, testData2;

                        for (int j = 0; j < testNum[i]; j++) {
                                int tmp = intDis(randEngine);
                                testData1.push_back(tmp); testData2.push_back(tmp);
                        }

                        auto startTime = system_clock::now();
            
                        vector<int> result = QuickSelect(testData1, 10); //调用具体的TopK算法
                        sort(result.begin(), result.end(), greater<int>());

                        auto endTime = system_clock::now();

                        sort(testData2.begin(), testData2.end(), greater<int>());

                        vector<int> correctResult(result.size(), 0);
                        copy(testData2.begin(), testData2.begin() + 10, correctResult.begin());
                        sort(correctResult.begin(), correctResult.end(), greater<int>());

                        cout << ((result == correctResult) ? "Correct;" : "Wrong;");

                        auto duration = duration_cast<microseconds>(endTime - startTime);
                        cout << "TimeCost:" << duration.count() << endl;
                }
        }
	return 0;
}
```

## 参考文章

[数据结构与算法分析](https://book.douban.com/subject/1139426/)

[快速选择select算法-五分化中项的中项](https://blog.csdn.net/hgqqtql/article/details/42157767)