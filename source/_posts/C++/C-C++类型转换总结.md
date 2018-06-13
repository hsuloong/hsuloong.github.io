---
title: C/C++类型转换总结
urlname: cpp-type-convert
date: 2018-03-13 10:40:43
copyright: true
mathjax: true
tags:
- C++
categories:
- C++
---

## 类型转换基础

对象的类型定义了对象能包含的数据和能够参与的运算，如果在表达式中混杂了多种不同的类型，编译器可能（有些类型转换非法则编译错误）会帮我们进行隐式的类型转换。如果两种类型能够进行相互转换则称两种类型是关联的。类型在进行隐式转换总体规则向上提升，即转换成精度和表达范围更广的数据类型。类型转换按照是否为自动可分为隐式转换和显式转换。隐式转换是由编译器自动完成的，无需编程人员的介入，显示转换是编程人员希望进行类型转换而显式要求编译器进行类型转换的过程。

## 隐式转换

在表达式中，如果我们在不同类型之间进行赋值，其根据类型不同得到不同的结果，如下为基本的转换规则

1. 非布尔->布尔：0->false; 非0->true
2. 布尔->非布尔：false->0; true->1
3. 浮点->整数：近似处理，保留浮点数小数点之前的部分
4. 整数->浮点数：小数部分记为0，如果整数所占空间超过浮点类型容量，精度会产生损失
5. 给无符号类型一个超过其表示范围的值时，结果是初始值对无符号类型表示数值的总数取模以后的余数。例如unsigned char(8 bit) 可以表示0-255，如果赋一个区间以外的值，则实际结果为该值对256取模所得的余数。unsigned char x = -1; // x = 255
6. 当赋给带符号类型一个超过其表示范围的值时，结果是未定义的。此时程序可能继续工作、可能崩溃，也可能产生垃圾数据（未定义）
7. 无符号数转换为更大的数据类型时, 只需简单地在开头添加0，这种运算称为0扩展
8. 将有符号数转换为更大的数据类型需要执行符号扩展，规则是将当前数符号位扩展至所需要的位数
9. 当数据类型转换时，同时需要在不同数据大小，以及无符号和有符号之间转换时，C语言标准要求先进行数据大小的转换，之后再进行无符号和有符号之间的转换。C语言中的强制类型转换保持二进制位值不变，只是改变解释位的方式。
10. 将一个大的数据类型转换为小的数据类型时，不管是无符号数还是有符号数都是简单地进行位截断
11. 进行整数的算术运算时，当结果变量的位数不足以存放实际实际结果的位数时，运算的结果就会因截断而产生溢出

如下图所示为一段测试代码：

```cpp
#include <iostream>
#include <limits>
using namespace std;

int main()
{
	bool bValue = true;
	char cValue = numeric_limits<char>::max();
	short sValue = numeric_limits<short>::max();
	int iValue = numeric_limits<int>::max();
	long lValue = numeric_limits<long>::max();
	long long llValue = numeric_limits<long long>::max();
	float fValue = numeric_limits<float>::max();
	double dValue = numeric_limits<double>::max();
	long double ldValue = numeric_limits<long double>::max();

	//测试开始
	cout << (bValue=cValue) << " " << (bValue=sValue) << " " << (bValue=iValue) <<endl;
	cout << (bValue=lValue) << " " << (bValue=llValue) << " " << (bValue=fValue) <<endl;
	cout << (bValue=dValue) << " " << (bValue=ldValue) << endl;
}
```

以上输出为全1。在进行未定义转换时，可能不同的编译器结果不一致。由于无符号与有符号之间的转换往往出乎意料，一般避免进行转换。

隐式转换主要发生在以下情况中：

        1. 在大多数表达式中，比int类型小的整型值提升为较大的整数类型
        2. 在条件中，非布尔值转换为布尔类型
        3. 初始化或者赋值语句，右侧转变为左侧类型
        4. 如果算数运算表达式有多种类型，需要转换成同一种类型
        5. 函数调用

## 整型提升

整型提升负责把小整数装换成较大的整数类型，对于bool、char、signed char、unsigned char、short、unisned short等类型，只要他们所有可能的值都能存在int类型中，就会提升为int，否则提升为unsigned int。wchar_t、char16_t、char32_6提升成int、unsigned int、long、unsigned long、long long、unsigned long long中最小的一种，前提是能容纳原始类型所有可能的值

无符号与有符号之间的转换有几种情况：

1. 如果无符号类型不小于带符号类型，则有符号类型转换成无符号类型，如int需要转换成unsigned int。如果有符号是负数，则取模运算
2. 如果带符号大于无符号且带符号类型可以容纳所有无符号值，则转换成带符号，否则转换成无符号。比如long和unsigned int，如果int和long大小相同，则long类型转换成unsigned int。

## 其他隐式转换

数组转换成指针，比如如下代码：

```cpp
int ia[10];
int *ip = ia; //ia转换成指向数组首元素的指针
```

当数组被用作decltype或者取址&、sizeof以及typeid等运算符时，上述转换不会发生。

常量整型和字面值nullptr能转换成任意的指针类型，任意非常量指针能转换成void \*指针，指向任意对象的指针能转换成const void\*。


## 显式转换

在C++中，有四种命名的强制类型转换关键字，分别为static_cast、dynamic_cast、const_cast、reinterpret_cast。基本使用格式为cast-name<type>(expression)，其中cast-name是上述四种转换类型，type为需要转换的类型，expression是要转换的值。现分别介绍如下：[文字来源](http://www.cnblogs.com/goodhacker/archive/2011/07/20/2111996.html)

### static_cast

类似于C风格的强制转换。无条件转换，静态类型转换。用于：

1. 基类和子类之间转换：其中子类指针转换成父类指针是安全的；但父类指针转换成子类指针是不安全的。(基类和子类之间的动态类型转换建议用dynamic_cast)
2. 基本数据类型转换。enum, struct, int, char, float等。static_cast不能进行无关类型（如非基类和子类）指针之间的转换。
3. 把空指针转换成目标类型的空指针。
4. static_cast不能去掉类型的const、volitale属性(用const_cast)。

### dynamic_cast

有条件转换，动态类型转换，运行时类型安全检查(转换失败返回NULL)：

1. 安全的基类和子类之间转换。
2. 必须要有虚函数。
3. 相同基类不同子类之间的交叉转换。但结果是NULL

### const_cast

去掉类型的const或volatile属性。

### reinterpret_cast

仅仅重新解释类型，但没有进行二进制的转换：

1. 转换的类型必须是一个指针、引用、算术类型、函数指针或者成员指针。
2. 在比特位级别上进行转换。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，在把该整数转换成原类型的指针，还可以得到原先的指针值）。
3. 最普通的用途就是在函数指针类型之间进行转换。
4. 很难保证移植性。

用法可以总结如下：

1. 去const属性用const_cast。
2. 基本类型转换用static_cast。
3. 多态类之间的类型转换用daynamic_cast。
4. 不同类型的指针类型转换用reinterpret_cast。

## 参考文章

[有符号数和无符号数的转换及思考](http://blog.csdn.net/gatieme/article/details/52557546)

[C++类型转换总结](http://www.cnblogs.com/goodhacker/archive/2011/07/20/2111996.html)

[c++数据类型转换：static_cast dynamic_cast reinterpret_cast const_cast](http://www.cnblogs.com/TenosDoIt/p/3175217.html)

[C++ - 隐式转换与四种强制类型转换](http://cuckootan.me/2016/05/28/C/C++/C++%20-%20隐式转换与四种强制类型转换/)

[c++ 四种强制类型转换介绍](http://blog.csdn.net/ydar95/article/details/69822540)