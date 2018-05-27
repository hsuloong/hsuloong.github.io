---
title: struct位段和字节对齐原理及其计算法则
urlname: struct-byte-alignment
date: 2018-03-12 17:04:21
tags:
- C++
categories:
- C++
---

## 位段基础知识

在实际编程过程中有些数据存储可能只需要1位或者几个比特位就可以，如果为了几位数据而定义一个完整字节将会浪费大量内存，正是由于这个原因，C语言提供一种称为位段（又称为位域）的数据结构。位段基本定义如下：

```cpp
struct pidTag
{
	unsigned int inactive : 1;
	unsigned int          : 1; /*1个位的填充*/
	unsigned int refcount : 6;
	unsigned int          : 0; /*填充到下一个字边界*/
	short pidID;
	struct pidTag *link;
};
```

位段的声明和struct类似，成员是一个或多个位的字段，这些字段在实际存储可能存储在一个或多个整型变量中。在声明时，位段成员必须是整形或枚举类型（通常是无符号类型），且在成员名的后面是一个冒号和一个整数，整数规定了成员所占用的位数。位域不能是静态类型。不能使用&对位域做取地址运算。如果位域的定义没有给出标识符名字，那么这是无名位域，无法被初始化。无名位域用于填充（padding）内存布局。只有无名位域的比特数可以为0。这种占0比特的无名位域，用于强迫下一个位域在内存分配边界对齐。而且位段在定义时，其最大长度不能超过依附类型的长度。

位段在存储时有自己的对齐规则：

1. 如果相邻位域字段的类型相同，且其位宽之和小于类型的sizeof大小，则后面的字段将紧邻前一个字段存储，直到不能容纳为止；
2. 如果相邻位域字段的类型相同，但其位宽之和大于类型的sizeof大小，则后面的字段将从新的存储单元开始，其偏移量为其类型大小的整数倍；
3. 如果相邻的位域字段的类型不同，则各编译器的具体实现有差异，VC6采取不压缩方式，Dev-C++和GCC采取压缩方式；
4. 如果位域字段之间穿插着非位域字段，则不进行压缩；
5. 整个结构体的总大小为最宽基本类型成员大小的整数倍，而位域则按照其最宽类型字节数对齐。

位段在存储时有多种实现，一般可分为两种[文字来源](https://zh.wikipedia.org/wiki/位段)：

### 大小端系统

通常在大端序系统（如PowerPC），安排位域从最重要字节（most-significant byte）到最不重要位（least-significant byte），在一个字节内部从最重要位（most-significant bit）到最不重要位（least-significant bit）；而在小端序系统（如x86），安排位域从最不重要位（least-significant byte）到最重要字节（most-significant byte），在一个字节内部从最不重要位（least-significant bit）到最重要位（most-significant bit）。<font color=red>共同遵从的原则是内存字节地址从低到高，内存内部的比特编号从低到高。</font>

### Microsoft Visual C++实现

在一个整数（integer）内的位域从最不重要位（least-significant）向最重要位（most-significant）依次分配。相邻的两个位域如果基类型（underlying type）的长度相同，在后的位域适合当前内存分配单元且没有跨内存分配边界，那么这两个位域分配到同一个（1、2或4字节的）分配单元。这可以通俗理解为：具有相同的基类型（underlying type）长度的相邻位域尽量装入基类型的同一个对象，如果装得下的话。

注：Most Significant Bit/Byte是指在一个字节序列中对序列取值影响最大那个字节，而Least Significant Bit/Byte则是取值影响最小的字节。

## 只含基本类型结构体字节对齐

在现代计算机中，受限于CPU寻址特点，每次CPU读取内存总是一次性读取多个字节，在32位计算机中一次性读取4个字节的数据，因此数据在内存中放置的位置并不是任意的，字节对齐后可以加快内存的访问速度。

在分析字节对齐时，需要有以下基本概念：

1. 数据类型自身的对齐值：char自对齐字节为1，short为2，int/float为4，double为8，指针根据平台不同略有差异，32位为4字节而64为8字节。
2. 结构体或者类的自身对齐值：成员中自对齐值最大的那个
3. 指定对齐值：使用#pragma pack(value)时指定对齐值为value
4. 数据成员、结构体、类有效对齐值：自身对齐值和指定对齐值的较小一个，即min(value,自身对齐值)

接下来分析一个具体实例，代码如下：

前提条件：int-4字节，char-1字节，short-2字节

```cpp
struct A{
	int    a;
	char   b;
	short  c;
};
struct B{
	char   b;
	int    a;
	short  c;
};
```

上述结构体中，sizeof(A)=8，sizeof(B)=12。假设B从地址空间0x0000开始存放，且指定对齐值默认为4(4字节对齐)。成员变量b的自身对齐值是1，比默认指定对齐值4小，所以其有效对齐值为1，其存放地址0x0000符合0x0000%1=0。成员变量a自身对齐值为4，所以有效对齐值也为4，只能存放在起始地址为0x0004~0x0007四个连续的字节空间中，符合0x0004%4=0且紧靠第一个变量。变量c自身对齐值为 2，所以有效对齐值也是2，可存放在0x0008~0x0009两个字节空间中，符合0x0008%2=0。所以从0x0000~0x0009存放的都是B内容。再看数据结构B的自身对齐值为其变量中最大对齐值(这里是b)所以就是4，所以结构体的有效对齐值也是4。根据结构体圆整的要求， 0x0000~0x0009=10字节，(10＋2)％4＝0。所以0x0000A~0x000B也为结构体B所占用。故B从0x0000到0x000B共有12个字节，sizeof(struct B)=12。之所以编译器在后面补充2个字节，是为了实现结构数组的存取效率。试想如果定义一个结构B的数组，那么第一个结构起始地址是0没有问题，但是第二个结构呢？按照数组的定义，数组中所有元素都紧挨着。如果我们不把结构体大小补充为4的整数倍，那么下一个结构的起始地址将是0x0000A，这显然不能满足结构的地址对齐。因此要把结构体补充成有效对齐大小的整数倍。其实对于char/short/int/float/double等已有类型的自身对齐值也是基于数组考虑的，只是因为这些类型的长度已知，所以他们的自身对齐值也就已知。


根据以上分析，字节对齐时一般需要遵循以下原则：

1. 结构体变量的首地址能够被其最宽基本类型成员的大小所整除
2. 结构体每个成员相对结构体首地址的偏移量(offset)都是成员大小的整数倍，如有需要编译器会在成员之间加上填充字节(internal adding)
3. 结构体的总大小为结构体最宽基本类型成员大小的整数倍，如有需要编译器会在最末一个成员之后加上填充字节(trailing padding)

以上规则解释如下：

- 第1条：编译器在给结构体开辟空间时，首先找到结构体中最宽的基本数据类型，然后寻找内存地址能被该基本数据类型所整除的位置，作为结构体的首地址。将这个最宽的基本数据类型的大小作为上面介绍的对齐模数。
- 第2条：为结构体的一个成员开辟空间之前，编译器首先检查预开辟空间的首地址相对于结构体首地址的偏移是否是本成员大小的整数倍，若是，则存放本成员，反之，则在本成员和上一个成员之间填充一定的字节，以达到整数倍的要求，也就是将预开辟空间的首地址后移几个字节。
- 第3条：结构体总大小是包括填充字节，最后一个成员满足上面两条以外，还必须满足第3条，否则就必须在最后填充几个字节以达到本条要求。

自定义对齐字节采用`#pragma pack(value)`和`#pragma pack()`成对使用，分别表示设定和复位。

## 结构体与联合体互相内嵌的情况

此处的结构体均是C中的struct和union，由于C++中结构体可以包含虚函数，使得分析更为复杂，故只以C中为准。

前提条件为：double-8字节，int-4字节，char-1字节，short-2字节

先定义一个结构体和一个联合体：

```
struct SData
{
        double dVal;
        int iVal1;
        short sVal;
        int iVal2;
        char cVal;
};

union UData
{
        int iArray[3];
        short sVal;
        char cVal;
};
```

### 结构体内嵌结构体

案例如下：

```
struct SWithS
{
        int iVal;
        SData sdVal;
        char cVal;
};
```

首先分析SData的基本情况。在SData中最长成员为8字节，因此其大小应该为8字节的整数倍。根据前面的规则分析有： 
 
`sizeof(SData) = 8(dVal) + 8(iVal1+sVal+padding)+8(iVal2+cVal+padding)=24。` 

对于整个结构体类型而言其自身的对齐值是内部最宽的数据类型即8字节，因此可以得出：  

`sizeof(SWithS)=8(iVal+padding)+24(sdVal)+8(cVal+padding)=40。`

将上述结构体在VS2017-64位平台以及g++7.3.0-64位测试结果为：

```
/*VS2017-64位平台*/
sizeof(SData)=24
sizeof(SWithS)=40

/*g++7.3.0-64位平台*/
sizeof(SData)=24
sizeof(SWithS)=40
```

### 结构体内嵌联合体

案例如下：

```
struct SWithU
{
        int iVal;
        UData udVal;
        char cVal;
};
```

首先分析UData的基本情况，由于UData中最长成员为4字节，因此其按照4字节对齐，由于iArray成员内存最大，因此：

`sizeof(UData)=12。`

联合体自身的对齐值是和结构体类似，因此：

`sizeof(SWithU)=4(ival)+12(udVal)+4(cVal+padding)=20。`

分别在两个平台测试，测试结果如下：

```
/*VS2017-64位平台*/
sizeof(UData)=12
sizeof(SWithU)=20

/*g++7.3.0-64位平台*/
sizeof(UData)=12
sizeof(SWithU)=20
```

### 联合体内嵌联合体

案例如下：

```
union UWithU
{
        int iVal;
        UData udVal;
        char cArray[13];
};
```

由于联合体内部的udVal成员对齐值为4，因此整个UWithU联合体对齐值为4。取最大成员同时对齐到4字节，由此：

`sizeof(UWithU)=16(cArray+padding)=16。`

分别在两个平台测试，测试结果如下：

```
/*VS2017-64位平台*/
sizeof(UWithU)=16

/*g++7.3.0-64位平台*/
sizeof(UWithU)=16
```

### 联合体内嵌结构体

案例如下：

```
union UWithS
{
        int iVal;
        SData sdVal; /*24*/
        char cArray[25];
};
```

由于联合体内部的sdVal成员对齐值为8，因此整个sdVal联合体对齐值为8。取最大成员同时对齐到8字节，由此：

`sizeof(UWithS)=32(cArray+padding)=32。`

分别在两个平台测试，测试结果如下：

```
/*VS2017-64位平台*/
sizeof(UWithS)=32

/*g++7.3.0-64位平台*/
sizeof(UWithS)=32
```

## 判断机器是大端或小端

```cpp
#include <iostream>
using namespace std;
int main()
{
	//定义一个两字节类型，一定要是两个字节，否则失效
	short x = 0x1234; 
	char *p = (char *)&x;
	if (*p == 0x34) {
		cout << "Little-Endian" << endl;
	}
	else if (*p == 0x12) {
		cout << "Big-Endian" << endl;
	}
	else {
		cout << "Unable to judge" << endl;
	}
	return 0;
}
```
在上述代码中，如果大端机器，则内存低位到高位依次存储12--34，小端则依次为34--12，因此通过判断低地址一个字节的值就可以判断大小端机器。



## 参考文章

[C专家编程](https://book.douban.com/subject/2377310/)

[失传的C结构体打包技艺](https://github.com/ludx/The-Lost-Art-of-C-Structure-Packing)

[struct/class等内存字节对齐问题详解](http://www.cnblogs.com/webary/p/4721017.html)

[C语言字节对齐问题详解](http://www.cnblogs.com/clover-toeic/p/3853132.html)

[C语言位域](http://c.biancheng.net/cpp/html/102.html)

[每个程序员都应当知道的“大小端”](https://www.jianshu.com/p/05aac833eacd)

[C结构体之位域（位段）](http://www.cnblogs.com/bigrabbit/archive/2012/09/20/2695543.html)

[字节对齐](https://www.jianshu.com/p/5cdf12b32222)

[详解大端模式和小端模式](https://blog.csdn.net/ce123_zhouwei/article/details/6971544)
