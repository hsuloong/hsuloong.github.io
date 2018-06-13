---
title: 结构体struct与类class的关系和区别
urlname: struct-class
date: 2018-03-12 15:47:43
copyright: true
mathjax: true
tags:
- C++
categories:
- C++
---

## struct与class基础

在C中，struct是用来定义自定义类型的，通过基本数据类型组合成一种新的抽象数据类型，其只能定义成员变量而不能定义函数。而C++中的struct有较大的变化，C++极大地扩充了struct的含义，C++中的struct可以定义函数，可以定义构造函数、析构函数，可以继承，C++中的struct和class基本可以混用（除泛型编程替代typename），除此之外两者唯一的区别就是默认的访问控制权限和默认的继承权限不同。

在C++中，struct默认的访问权限和继承权限是public，而class是private。而且struct可以继承于class，而默认的继承权限取决于子类是struct还是class。


## strcut与class实验

在C++环境下，有如下代码：

```cpp
struct S1
{
	int x;
};

struct S2 : S1
{
	int y;
};

class C1
{
	int m;
};

class C2 : C1
{
	int n;
};

struct S3 : C1
{
	
};

class C3 : S1
{

};

int main()
{
	S1 S_S1 ;
	S_S1.x = 0; //合法，因为数据访问权限为public
	S2 S_S2 ;
	S_S2.x = S_S2.y = 0; //合法，因为默认继承权限为public且数据访问权限为public
	S3 S_S3 ;
	//S_S3.m = 0; //非法，因为默认继承权限为public但数据访问权限为private
	C1 C_C1 ; 
	//C_C1.m = 0; //非法，因为数据访问权限为private
	C2 C_C2 ;
	//C_C2.n = 0; //非法，因为默认继承权限为private
	C3 C_C3 ;
	//C_C3.x = 0; //非法，因为默认继承权限为private
	return 0;
}
```


## 参考文章

[Struct和Class的区别](http://blog.csdn.net/yuliu0552/article/details/6717915)

[c++ 结构体和类的区别](http://blog.csdn.net/fengxinziyang/article/details/5909237)

[C++类class和结构体struct区别](http://www.weixueyuan.net/view/6337.html)