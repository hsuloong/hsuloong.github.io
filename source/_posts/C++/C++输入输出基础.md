---
title: C++输入输出基础
urlname: cpp-io
date: 2018-03-12 9:05:24
tags:
- C++
categories:
- C++
---

## C++输入输出基础知识

C++语言自身没有定义输入输出语句，其通过标准库来提供IO能力。在C++中iostream库为常用的IO库，该库包含两个基础类型istream和ostream，分别表示输入流（即读）与输出流（即写）。其中流指的是一个字符序列。标准库中定义了四个IO对象，分别为cin（标准输入）、cout（标准输出）、cerr（非缓冲标准错误流）、clog（缓冲标准错误流）。在使用诸如“cout<<something;”语句时，该语句返回输入输出流对象的引用，这也是为什么可以采用“cout<<something1<<something2;”这种格式的原因。

在使用一个istream对象作为条件时，其返回的是输入流的有效状态标记，如果流是有效的则返回真，否则如果读到流末尾（EOF）或者无效输入（读入一个整数实际输入不是整数）返回假。

## C++IO库概览

C++IO库不仅定义了istream和ostream，还定义其他IO类型，具体见下：

- 头文件iostream
 
	1. istream，wistream：用来从流中读取数据，w代表宽字符（wchar_t）
	2. ostream，wostream：用来向流写入数据
	3. iostream，wiostream：读写流。

- 头文件fstream

	1. ifstream，wifstream：从文件读取数据
	2. ofstream，wofstream：向文件写入数据
	3. fstream，wfstream：读写文件

- 头文件sstream

	1. istringstream，wistream：从string中读取数据
	2. ostringstream，wostringstream：向string写入数据
	3. stringstream，wstringstream：读写string

- 头文件iomanip

在以上流类型中，ifstream和istringstream继承于istream，而ofstream和ostringstream则继承于ostream。IO对象是一种不能进行拷贝或者赋值的对象，而且由于在写过程中对象的状态会发生改变，流对象也不能设置为const（const对象不能调用非const类型的成员函数）。

在进行IO操作时，难免会发生错误，因此IO类定义了一系列条件状态，如下所示：

	1. ::iostate：是一种机器相关的类型，提供了条件状态的完整功能
	2. ::badbit：表明流已崩溃
	3. ::failbit：表明一个IO操作失败
	4. ::eofbit：表明流已达到末尾
	5. ::goodbit表明流未处于错误状态，值保证为0
	6. .eof()：若eofbit置位则返回true
	7. .fail()：若badbit或failbit置位则返回true
	8. .good()：流有效则返回true
	9. .clear()：所有条件状态复位，流状态成为有效，返回void
	10. .clear(iostate flag)：根据flag标志位将相应条件状态复位，返回void
	11. .setstate(iostate flag)：根据flag标志位将相应条件状态置位，返回void
	12. .rdstate()：返回流当前条件状态，返回值为iostate类型

一旦一个流发生错误，其上所有的后续操作均会失败，只有处于无错状态时才能读写数据。

## 输出缓冲管理

每一个输出流都有一个缓冲区用于保存程序读写的数据。缓冲机制的存在使得输出不一定被立刻显示或者写入文件出来，而导致真正读写发生（即缓冲刷新）主要有如下原因：

	1. 程序正常结束，执行缓冲刷新
	2. 缓冲区满
	3. 使用endl等操作符显式刷新缓冲区
	4. 每个输出操作后用unitbuff设置流的内部状态来清空缓冲区，cerr默认设置unitbuff，因此cerr是立刻刷新的
	5. 一个输出流被关联到另外一个流时，当读写该流时关联流的缓冲区会被立刻刷新，默认cin和cerr关联到cout，导致读cin或者写cerr会刷新cout缓冲区

以下代码为立刻刷新缓冲区的方式：

```cpp
cout << "Hello world!" << endl; //输出Hello world!+换行并刷新缓冲区
cout << "Hello world!" << flush; //输出Hello world!并刷新缓冲区
cout << "Hello world!" << ends; //输出Hello world!+空字符并刷新缓冲区
cout << unitbuff; //所有输出操作立刻刷新缓冲区
cout << nounitbuff; //还原正常缓冲
```

在程序崩溃后，输出缓冲将不会被刷新。

## 关联输入输出流

当一个输入流被关联到一个输出流时，任何从输入流读取数据的操作均会导致输出流被刷新。关联流的函数为tie，其重载为两个版本：

	1. tie()无参数，返回指向关联到输出流的指针，没有关联返回空指针
	2. tie(ostream对象指针)，将自己关联到输出流
	
以下下代码演示关联操作
```cpp
cin.tie(&cout); //将cin关联到cout上，标准库默认关联
ostream *oldTie = cin.tie(nullptr); //cin不在与其他流关联
cin.tie(&cerr); //关联到cerr，读取cin会刷新cerr
```

## 文件输入输出

头文件fstream定义了文件读写的相关类型。由于fstream继承于iostream，因此iostream操作都可以使用。但是新增了一些操作，如下所示：

	1. fstream fObject("fileName")：创建一个流并打开某个文件
	2. fstream fObject("fileName", mode)：创建一个流并打开某个文件，mode指定读写模式
	3. fObject.open("fileName")：并打开某个文件
	4. fObject.close()：关闭流绑定的文件
	5. fObject.is_open()：判断流是否和文件关联且未关闭并返回bool值

fstream对象在销毁时会自动调用close函数。

在文件输入输出中，主要有如下读写模式：

	1. in：只读打开，只能对ifstream和fstream设置
	2. out：只写，会覆盖原始数据，只能对ofstream和fstream对象设置
	3. app：每次写均定位到文件末尾，只要trunc没被设置
	4. ate：打开文件后立即定位末尾，可和任何模式组合使用
	5. trunc：截断文件，只有当out被设置时才能被设定
	6. binary：二进制方式读写，可和任何模式组合使用

## string流

在sstream头文件中定义了三个类型来支持内存IO，这几个类型可以直接项string读写数据。sstream也继承自iostream，但有自己一些特殊的操作，如下：

	1. sstream sObject(string s)：保存字符串s的一个拷贝，此构造函数是explicit的，编译器不会自动执行转换
	2. sObject.str()：返回保存的string拷贝
	3. sObject.str(string s)：将字符串s拷贝到对象中，返回void

以下为一个sstream使用范例：

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
using namespace std;

int main(int argc, _TCHAR* argv[])
{
        stringstream Test;
        string data("Hello World!");
        Test.str(data);
        string outPut;
        while (Test >> outPut) {
                cout << outPut << endl;
        }
        Test.clear();
        Test << "Hi" << "World";
        cout << Test.str() << endl;
        return 0;
}
```

整个控制台输出为：

	Hello
	World!
	HiWorldorld!

## 格式化输入输出

标准库定义了一组操纵符用来修改流的格式状态，且格式控制由设置和复原成对出现。具体操作符可见相关手册。

同时IO库还定义了一组未格式化的输入输出操作，用于将流当成无解释的字节序列（不跳过空格）来处理，具体见下：

- 单字节操作

	1. object.get(ch)：读取下一个字节存入ch中，返回对象
	2. object.put(ch)：将字符写到流中，返回对象
	3. object.get()：读取下一个字节并作为int类型返回
	4. object.putback(ch)：读将字符放入流中，返回对象
	5. object.unget()：将对象object向后移动一个字节，返回对象
	6. object.peek()：读取下一个字节并作为int类型返回但不从流中删除

- 多字节操作

	1. obejct.get(sink, size, delim)：从流中最多读出size个字节，遇到delim或者文件结尾或者读满size个字节则停止读写，将delim放在输入流中且不丢弃delim，读出的数据放入sink数组中
	2. obejct.getline(sink, size, delim)：和get类似，不过会读取并丢弃delim
	3. obejct.read(sink, size)：读取最多size个字节并放入sink数组
	4. obejct.gcount()：返回上一个未格式化操作读取的字节数
	5. obejct.write(src, size)：将数组src的size个字节写入流中
	6. obejct.ignore(size, delim)：读取并忽略size个字节，包括delim

## 流随机访问

大多数系统cin\cout\cerr\clog的流不支持随机访问，因此一般只有istream和ostream支持随机访问。通常由两组函数tell（返回当前位置）和seek（定位到某一位置）根据是输入流还是输出流主要有tellg\tellp、seekg\seekp，g表示读取数据，p表示写入数据，函数各定义如下：

	1. tellg\tellp：返回输入流或者输出流标记的当前位置
	2. sekg(pos)\seekp(pos)：将输入输出流重定位到决定位置pos
	3. sekg(off, from)、sekp(off, from)：输入输出定位到from之前或者之后的off个字符，from可以是beg（流开始）、cur（流当前）、end（流结尾）	





## 参考文章

[C++标准设备的输入/输出（cin,cout,cerr,clog,>>,<<）](http://www.weixueyuan.net/view/5875.html)

[c++里关于cerr,clog,cout三者的区别](http://blog.csdn.net/zx824/article/details/6644455)

[clog，cout，cerr 输出机制](http://www.cnblogs.com/xiezhw3/p/4349766.html)

[C/C++ 标准输入输出重定向](http://www.cnblogs.com/hjslovewcl/archive/2011/01/10/2314356.html)









