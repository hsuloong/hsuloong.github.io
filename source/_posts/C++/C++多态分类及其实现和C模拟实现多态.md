---
title: C++多态分类及其实现和C模拟实现多态
urlname: cpp-polymorphism
date: 2018-03-12 12:59:43
tags:
- C++
categories:
- C++
---

## 多态基础

多态是指程序在运行时，相同的消息给予不同的对象可能会产生不同的行为，可以简单概括为“一个接口，多种方法”。多态可分为变量多态和函数多态。或者可以按照多态发生的时机分为动态多态和静态多态。多态是面向对象程序设计三大特征之一（封装、继承、多态）。

## 动态多态

动态多态是指在运行时确定采用的行为，这在C++中动态多态是通过虚函数和继承实现，同时在使用基类指针或者引用调用虚函数时会执行动态绑定。

## 静态多态

静态多态是指在编译时期就能确定行为的一种多态行为，静态动态在C++实现中可分为函数重载与泛型编程。以下分别介绍

- 函数重载

在C++中，函数重载是指函数具有相同的名字但是具有不同参数的函数，在调用函数时编译器根据参数匹配规则选择一个最佳匹配的函数进行调用，调用的函数在编译阶段即完成

- 泛型编程

泛型编程实现的多态又称为参数多态，是指在声明定义函数、复合类型、变量时不指定具体的类型，而把类型作为参数使用，使得该声明定义对各种具体的类型都适用。STL即使用了泛型编程技术。

## C模拟实现多态

在C++实现动态多态中，其内部原理是采用虚函数表+虚函数表指针的方式。在定义了虚函数的类中，编译器会自动添加一个虚函数表指针，通过该指针在运行过程可以正确找到需要调用函数的入口地址（函数指针）。在C中，有一种指针void *称为无类型指针，这种指针可以指向任意的数据类型，因此我们可以通过该指针实现不同数据的定义以及处理方式，比如我定义一个数据处理基结构，其定义如下：

```cpp
struct Base
{
	void *data; //数据
	//dealFun是一个指针，指向一个返回值为void且参数 void *类型
	void (*dealFun)(void *inData); 
};

struct CString
{
	char *cData;
	void (*dealFun)(void *inData); //假设为一逐字符打印函数
};

//链表节点
struct Node
{
	int data;
	Node *next;
};

struct List
{
	Node *listData; //单链表
	void (*dealFun)(void *inData); //假设为打印单链表函数
};

void printCString(void *CString)
{
	char *cycleIter = (char *)CString;
	while (cycleIter != nullptr && *cycleIter != '\0') {
		printf("%c", *cycleIter); ++cycleIter;
	}
	printf("\n");
}

void printNode(void *inNode)
{
	Node *cycleIter = (Node *)inNode;
	while (cycleIter != nullptr) {
		printf("%d\t", cycleIter->data);
		cycleIter = cycleIter->next;
	}
	printf("\n");
}



int main()
{
	CString cData = {"Hello world", &printCString};
	Node nodeData[2] = {{0,&nodeData[1]},{1,nullptr}};
	List listData = {nodeData, &printNode};
	Base *test1 = (Base *)&cData;
	Base *test2 = (Base *)&listData;
	test1->dealFun(test1->data);
	test2->dealFun(test2->data);
	return 0;
}

```

上述程序输出的结果为：

	Hello world
	0       1



## 参考文章

[多态分类-强制多态，参数多态，过载多态，包含多态的理解](http://www.cnblogs.com/gaojing/archive/2007/05/03/735004.html)

[C++ 多态深度剖析](http://blog.jobbole.com/107432/)

[多态 (计算机科学)](https://zh.wikipedia.org/wiki/多型_(计算机科学))

[技巧：用C语言实现程序的多态性](https://www.ibm.com/developerworks/cn/linux/l-cn-cpolym/index.html)

[C语言实现多态](https://www.jianshu.com/p/bcecbfa8ff81)