---
title: STL基础概述
urlname: stl-foundation
date: 2018-03-13 21:59:43
tags:
- C++
- STL
categories:
- C++
---

## STL基础知识

STL全称标准模板库（Standard Template Library），是一套程序库，采用了模板编程技术。STL并不是C++标准库的一部分，其由六大组件组成：

1. 容器（containers）：各种数据结构，有vector（动态数组），list（双向链表），dequeue（双端队列），set（红黑树），map（红黑树）。
2. 算法（algorithm）：常用的算法如排序、搜索、复制、删除等
3. 迭代器（iterator）：算法和容器之间的桥梁，是一种泛型指针，主要有五种类型，重载了一些运算符。
4. 仿函数（functor）：重载了operator()的类或者类模板，行为和函数类似
5. 配接器（adapter）：一种用来修饰容器或仿函数或迭代器的结构，比如stack和queue，行为和容器类似，不过底层操作采用其他容器实现。
6. 配置器（allocator）：负责进行空间配置与管理

六大组件交互过程如下：容器通过配置器获取存储空间，算法通过迭代器存取容器的内容，仿函数协助算法完成不同策略的操作，配接器可以修饰或套接仿函数，通过适配器接受一种已有的事物而表现起来像另外一种事物。

盗用一张交互图：[图片来源](http://blog.csdn.net/u010275850/article/details/51935404)

![六大组件交互图](/images/stl-components.jpg)


## STL语法

STL有一些使用语法并不是非常常规，主要有如下几点：

- 临时对象

临时对象是一种无名对象，如果临时对象的产生并不是编程人员刻意为之，那么一般意味着程序效率存在一种损失。在某些时候如果刻意使用临时对象则可以使程序非常简洁。其方法为在类型名称后直接加一括号并可指定初值，如int(8)定义了一个值为8的临时对象。STL常将这种操作应用于仿函数上。代码示例如下：

```cpp
template <typename T>
class print
{
public:
	void operator() (const T&ele) { cout << ele << endl;}
};

int main()
{
	vector<int> ia{0, 1, 2, 3, 4, 5, 6, 7};
	for_each(ia.begin(), ia.end(), print<int>()); //STL函数
	return 0;
}
```

在调用for_each()函数时传入一个无名对象。

- 静态常量整数成员在class内部直接初始化

C++11中，静态常成员可以直接在类内初始化，代码示例如下：

```cpp
template <typename T>
class print
{
public:
	void operator() (const T&ele) { cout << ele << endl;}
private:
	static const int iVal = 2;
	static const char cVal = 'g';
};
```

- ++或\-\-\以及解引用运算符

++与\-\-运算符在迭代器上的运用十分广泛，基本迭代器都需要实现上述操作。这是示例代码：

```cpp
template <typename T>
class INT
{
public:
	INT() : iVal(0) {}
	INT &operator++(); //前置++
	const INT operator++(int); //后置++
	int &operator*() const; //解引用
private:
	int iVal;
};

template <typename T>
INT<T> & INT<T>::operator++()
{
	++(this->iVal);
	return *this;
}

template <typename T>
const INT<T> INT<T>::operator++(int)
{
	INT tmp = *this;
	++(this->iVal);
	return tmp;
}

template <typename T>
int &INT<T>::operator*() const //const不能丢
{
	return (int&)this->iVal;
}
```

- 前闭后开区间表示

STL里面的算法需要获取一对迭代器表示操作的范围，STL规定该区间为左闭右开，即[first, end)区间。实际需要操作元素的位置为first到end-1。左闭右开区间带来了许多方便，比如在去换判断终止时可以采用iter != end的形式，示例代码如下：

```cpp
template <class InputIterator, class T>
InputIterator fin(InputIterator first, InputIterator end, const T&key)
{
	for ( ; first != end && *first != key; ++first);
	return first;
}
```

- 函数调用

STL算法大多提供两个版本，一个针对普通情况，一个针对特殊情况。比如对于排序算法，可能默认按照递增顺序排列，如果想要按照用户自己的意愿进行排列，则可以由用户指定策略，而这可以通过函数调用完成。比如考虑以下实例：

```cpp
int fcmp(const void *in1, const void *in2)
{
	const int *iP1 = (const int*)in1;
	const int *iP2 = (const int*)in2;
	if (*iP1 < *iP2) return -1;
	if (*iP1 > *iP2) return 1;
	return 0;
}

int main()
{
	int ia[] = {1,5,6,2,4,5,7};
	qsort(ia, sizeof(ia)/sizeof(ia[0]), sizeof(ia[0]), fcmp); //使用用户定义的排序策略
	return 0;
}
```
上示代码中函数指针无法持有自身的状态（即每次调用状态不会发生变化，仿函数由于是一个对象，实际可以携带自己的数据）。

## STL常见容器或配接器

1. vector：底层数据结构为数组 ，序列容器，支持快速随机访问
2. list：底层数据结构为双向链表，序列容器，支持快速增删
3. dequeue：底层数据结构为一个中央控制器和多个缓冲区，序列容器，支持首尾（中间不能）快速增删，也支持随机访问
4. stack：底层一般用list或deque实现，封闭一端即可
5. queue：和stack类似底层一般用list或deque实现
6. priority_queue：底层数据结构一般为vector为底层容器，使用数据结构堆
7. set：底层数据结构为红黑树，有序关联容器，关键字不重复
8. multiset：底层数据结构为红黑树，有序关联容器，关键字允许重复
9. map：底层数据结构为红黑树，有序关联容器，关键字不重复
10. multimap：底层数据结构为红黑树，有序关联容器，关键字允许重复
11. unordered_set：底层数据结构为hash表，无序，关键字不重复
12. unordered_multiset：底层数据结构为hash表，无序，关键字允许重复
13. unordered_map：底层数据结构为hash表，无序，关键字不重复
14. unordered_multimap：底层数据结构为hash表，无序，关关键字允许重复
15. bitset：存储系列位类似的固定大小的布尔向量。实现按位运算，没有迭代器，不是序列
16. valarray：数值类型的std::vector。牺牲泛型能力而专为数值计算做了优化。
17. forward_list：单向链表，只支持单向顺序访问，在链表任何位置进行插入和删除速度都很快


参考文章

[STL底层数据结构实现](http://www.cnblogs.com/hustlijian/p/3611424.html)

[标准模板库](https://zh.wikipedia.org/wiki/标准模板库)

[容器库](http://zh.cppreference.com/w/cpp/container)
