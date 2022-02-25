---
layout: post
title: 数据对齐
excerpt: "c/c++结构体/类数据结构内存对齐问题解释。"
modified:
tags: [c, c++, struct, class, 内存对齐，数据结构]
comments: true
share: false
---

## 引言

使用手动管理内存的语言时，很多编译器的默认行为会造成困惑，甚至是某些“灵异”时间，本文对数据结构的数据对齐问题进行了整理和测试，对需要降低程序内存占用率或者平台移植有些许帮助。

## 结构体中的数据对齐

**数据结构对齐**（[Data structure alignment][wiki]）指将数据放在内存字长整数倍的位置上，因CPU处理内存的方式而提高系统性能。数据对齐会在数据直接填充一些空字节。

以下将通过几个实际例子来说明在结构体里是如果进行数据对齐的。

### 例一：基础

{% highlight c %}
#include <stdio.h>

struct foo{
	char a_;	// 1 byte
	int b_;		// 4 byte
	short c_;	// 2 byte
};

int main(int argc, char** argv){
	struct foo f;
	printf("sizeof struct foo %ld.\n", sizeof(struct foo));
	printf("address of  f %p\n", &f);
	printf("address of  a_ %p\n", &f.a_);
	printf("address of  b_ %p\n", &f.b_);
	printf("address of  c_ %p\n", &f.c_);
	return 0;
}
/*
output:
sizeof struct foo 12.
address of  f 0x7fff5b9e9510
address of  a_ 0x7fff5b9e9510
address of  b_ 0x7fff5b9e9514
address of  c_ 0x7fff5b9e9518
*/
{% endhighlight %}

很明显foo的大小不是1(char)+4(int)+2(short)=7byte，这就是一个典型的数据结构对齐，编译器默认对foo中的数据结构的布局进行了处理。从数据成员的地址可以得出数据成员在内存中的布局如下表：

|---------+---------+---------+---------+---------|
|地址\偏移| 0       | 1       | 2       | 3       |
|:--------|:--------|:--------|:--------|:--------|
| 0x7fff5b9e9510 | a_      | padding | padding | padding |
| 0x7fff5b9e9514 | b_      | b_      | b_      | b_      |
| 0x7fff5b9e9518 | c_      | c_      | padding | padding |

可以看到数据在内存中的位置并不是紧密排列的。而是中间存在许多“空洞”（上表中写padding的字节）如：a_成员后三个字节的位置。这是数据结构对齐的规则一:**基本数据类型要在自身大小的整数倍的内存地址处开始存放**，我们将这个位置叫**边界**，而规则一称作**数据对齐(data alignment)**。

再看c_成员后也有两个字节的填充，然而c_后并没有其他成员，这是数据结构对齐的规则二：**结构体总长度要是数据成员中长度最长成员的长度的整数倍**。我们将这个**长度最长成员的长度**称为**标量**，而规则二称作**结构体填充(data structure padding)**。

### 例二：数组

数组的例子分结构体中有数组成员的情况，和结构体数组的情况两种来讲，先看结构体数据成员中含有数组的情况：

{% highlight c %}
#include <stdio.h>


struct foo{
	char a_;	// 1 byte
	int b_;		// 4 byte
	short c_;	// 2 byte
	char d_[5];	// 5 byte
};

int main(int argc, char** argv){
	struct foo f;
	printf("sizeof struct foo %ld.\n", sizeof(struct foo));
	printf("address of  f %p\n", &f);
	printf("address of  a_ %p\n", &f.a_);
	printf("address of  b_ %p\n", &f.b_);
	printf("address of  c_ %p\n", &f.c_);
	printf("address of  d_[0] %p\n", &f.d_[0]);
	printf("address of  d_[1] %p\n", &f.d_[1]);
	printf("address of  d_[2] %p\n", &f.d_[2]);
	printf("address of  d_[3] %p\n", &f.d_[3]);
	printf("address of  d_[4] %p\n", &f.d_[4]);
	return 0;
}
/*
output:
sizeof struct foo 16.
address of  f 0x7fff4eb9ef00
address of  a_ 0x7fff4eb9ef00
address of  b_ 0x7fff4eb9ef04
address of  c_ 0x7fff4eb9ef08
address of  d_[0] 0x7fff4eb9ef0a
address of  d_[1] 0x7fff4eb9ef0b
address of  d_[2] 0x7fff4eb9ef0c
address of  d_[3] 0x7fff4eb9ef0d
address of  d_[4] 0x7fff4eb9ef0e
*/
{% endhighlight %}

数据成员内存布局如下：

|---------+---------+---------+---------+---------|
|地址\偏移| 0       | 1       | 2       | 3       |
|:--------|:--------|:--------|:--------|:--------|
| 0x7fff4eb9ef00 | a_      | padding | padding | padding |
| 0x7fff4eb9ef04 | b_      | b_      | b_      | b_      |
| 0x7fff4eb9ef08 | c_      | c_      | d_[0]   | d_[1]   |
| 0x7fff4eb9ef0a | d_[2]   | d_[3]   | d_[4]   | padding |

数据成员是数组的情况下，也只是将数组中的每个元素按例一的规则做排放。下面看结构体数组这种情况：

{% highlight c %}
#include <stdio.h>


struct foo{
	char a_;	// 1 byte
	int b_;		// 4 byte
	short c_;	// 2 byte
	char d_[5];	// 5 byte
};

int main(int argc, char** argv){
	struct foo f[3];
	printf("sizeof struct f %ld.\n", sizeof(f));
	printf("address of  f %p\n", &f);
	printf("address of  f[0] %p\n", &f[0]);
	printf("address of  f[1] %p\n", &f[1]);
	printf("address of  f[2] %p\n", &f[2]);

	return 0;
}
/*
output:
sizeof struct f 48.
address of  f 0x7fff0505ff70
address of  f[0] 0x7fff0505ff70
address of  f[1] 0x7fff0505ff80
address of  f[2] 0x7fff0505ff90
*/
{% endhighlight %}

结构体内数据成员的布局不表，看下结构体在数组中布局：

|---------+---------+---------+---------+---------|
|地址\偏移| 0       | 1       | 2       | ...15    |
|:--------|:--------|:--------|:--------|:--------|
| 0x7fff0505ff70 | f_[0]   | f_[0]   | f_[0]   | f_[0]   |
| 0x7fff0505ff80 | f_[1]   | f_[1]   | f_[1]   | f_[1]   |
| 0x7fff0505ff90 | f_[2]   | f_[2]   | f_[2]   | f_[2]   |

每个结构体在数组中紧密排列，元素之间没有“空洞”，这里也是为什么规则二要对结构体整体进行填充的原因。

### 例三：位域bit fields

当结构体中包含[位域(Bit field)][bit_fields]的情况，由于位域成员不能去做取地址操作只能输出a_,b_成员的地址了，代码如下：

{% highlight c %}
#include <stdio.h>


struct foo{
	char a_;	// 1 byte
	int b_;		// 4 byte
	int c_:12;	// 2 byte
	int d_:2;	// 1 byte
};

int main(int argc, char** argv){
	struct foo f;
	printf("sizeof struct foo %ld.\n", sizeof(struct foo));
	printf("address of  f %p\n", &f);
	printf("address of  a_ %p\n", &f.a_);
	printf("address of  b_ %p\n", &f.b_);
	return 0;
}
/*
output:
sizeof struct foo 12.
address of  f 0x7fff013de3f0
address of  a_ 0x7fff013de3f0
address of  b_ 0x7fff013de3f4
*/
{% endhighlight %}

内存布局如下：

|---------+---------+---------+---------+---------|
|地址\偏移| 0       | 1       | 2       | 3        |
|:--------|:--------|:--------|:--------|:--------|
| 0x7fff013de3f0 | a_      | padding | padding | padding |
| 0x7fff013de3f4 | b_      | b_      | b_      | b_      |
| 0x7fff013de3f8 | c_      | c_      | d_      | padding |

对于位域c和c++有些区别，c中位域大小不能超过其类型自身大小，而c++可以超过。

### 例四：嵌套

当结构体包含其他结构体的时候，内部结构体的布局依旧按照规则一、二，外部结构体在布局结内部构体时按照规则三：**内部结构体使用其标量进行对齐**。如下:

{% highlight c %}
#include <stdio.h>


struct bar{
	char a_;	// 1 byte
	int b_;		// 4 byte
};

struct foo{
	char a_;	// 1 byte
	struct bar b_;	// 8 byte
};

int main(int argc, char** argv){
	struct foo f;
	printf("sizeof struct bar %ld.\n", sizeof(struct bar));
	printf("sizeof struct foo %ld.\n", sizeof(struct foo));
	printf("address of  f %p\n", &f);
	printf("address of  foo a_ %p\n", &f.a_);
	printf("address of  foo b_ %p\n", &f.b_);
	printf("address of  bar a_ %p\n", &f.b_.a_);
	printf("address of  bar b_ %p\n", &f.b_.b_);
	return 0;
}
/*
output:
sizeof struct bar 8.
sizeof struct foo 12.
address of  f 0x7fff4fc01bd0
address of  foo a_ 0x7fff4fc01bd0
address of  foo b_ 0x7fff4fc01bd4
address of  bar a_ 0x7fff4fc01bd4
address of  bar b_ 0x7fff4fc01bd8
*/
{% endhighlight %}

内存布局如下：

|---------+---------+---------+---------+---------|
|地址\偏移| 0       | 1       | 2       | 3        |
|:--------|:--------|:--------|:--------|:--------|
| 0x7fff4fc01bd0 | a_      | padding | padding | padding |
| 0x7fff4fc01bd4 | b\_.a_  | padding | padding | padding |
| 0x7fff4fc01bd8 | b\_.b_  | b\_.b_  | b\_.b_  | b\_.b_  |

可以看到，b_.a_并没有被放在0x7fff4fc01bd1的位置，而是在bar的标量4的整数倍为值开始排列。

## #pragma pack

使用上节中的规则已经可以处理大部分情况了，用下面这个例子引出我们这节要将的内容，在32位机器上执行下面代码,

{% highlight c %}
#include <stdio.h>


struct foo{
    char a_;    // 1 byte
    double b_;     // 8 byte
};

int main(int argc, char** argv){
    struct foo f;
    printf("sizeof struct foo %ld.\n", sizeof(foo));
    printf("address of  f %p\n", &f);
    printf("address of  a_ %p\n", &f.a_);
    printf("address of  b_ %p\n", &f.b_);

    return 0;
}
/*32-bit machine
output:
sizeof struct foo 12.
address of  f 0xbfc98de8
address of  a_ 0xbfc98de8
address of  b_ 0xbfc98dec
*/
{% endhighlight %}

内存布局如下：

|---------+---------+---------+---------+---------|
|地址\偏移| 0       | 1       | 2       | 3        |
|:--------|:--------|:--------|:--------|:--------|
| 0xbfc98de8 | a_      | padding | padding | padding |
| 0xbfc98dec | b_      | b_      | b_      | b_      |
| 0xbfc98df0 | b_      | b_      | b_      | b_      |

发现foo的b_成员在0xbfc98dec处排放，0xbfc98dec是4的整数倍，不是8的整数倍，也就是说他没有按照上面讲的规则一，同时sizeof(bar)是12,也是4的整数倍，不符合规则二。这里需要引入一个新概念**最大对齐长度(maximum alignment)**,也就是说规则一二中讲到的，边界和标量，不会随着成员类型的大小而无限增长，是在一个范围内的，这个范围的的最大值就是最大对齐长度。

最大对齐长度属于编译器的行为，我们可以通过#pragma pack(n)来进行设置，n=2^m(m为N)，n的最大值笔者测试gcc编译器在32位是4字节，64位是16字节,默认值（没有找到官方文档的说明）即最大值。更多关于#pragma pack的信息查看这里[windows][pragma_pack_vc]平台，[linux][pragma_pack_gcc]平台。

## c与c++

谈到结构体，虽说c++兼容c，但是毕竟还是有差别的，本节将从两个方面讲解结构体在c和c++上的差异，以及这些差异对数据结构对齐问题上带来的影响。

### 空类型

{% highlight c %}
#include <stdio.h>

struct foo{
};

int main(int argc, char** argv){
    struct foo f1, f2;
    printf("sizeof struct f %ld.\n", sizeof(struct foo));
    printf("address of  f1 %p\n", &f1);
    printf("address of  f2 %p\n", &f2);

    return 0;
}
/*
output in c:
sizeof struct f 0.
address of  f1 0x7ffffafb7c90
address of  f2 0x7ffffafb7c90

output in c++:
sizeof struct f 1.
address of  f1 0x7fff5c5bcd4f
address of  f2 0x7fff5c5bcd4e
*/
{% endhighlight %}

可以看到，c中空结构体就是空的，大小为0，而c++中空结构体的大小则是1。在c中，当空结构体出现时，他在内存里是不占空间的，连续声明了两个foo，地址是一样的，所以c++中对空结构体/类做了处理，使其最小为1。类似以下这种结构体大小就要分语言来看待了。

{% highlight c %}
struct bar{
};

struct foo{
	int a_;
	struct bar b_;
};
{% endhighlight %}

### 继承

c中没有继承的概念不表，c++中类和结构体的区别看[这里][diff_struct_class],下面使用类的继承来对数据结构对齐做说明。

{% highlight c %}
#include <iostream>

class Base{
private:
	int a_;
};

class Derived : public Base{

private:
	char a_;
};

int main(int argc, char** argv){

	std::cout<< "sizeof Base " << sizeof(Base) << std::endl;
	std::cout<< "sizeof Derived " << sizeof(Derived) << std::endl;
    return 0;
}
/*
output:
sizeof Base 4
sizeof Derived 8
*/
{% endhighlight %}

子类继承父类的所有成员，按照以上规则进行对齐和填充没有特殊之处，在看另一个例子：

{% highlight c %}
#include <iostream>

class Base{
private:
	int a_;
};

class Derived :virtual public Base{

private:
	char a_;
};

int main(int argc, char** argv){

	std::cout<< "sizeof Base " << sizeof(Base) << std::endl;
	std::cout<< "sizeof Derived " << sizeof(Derived) << std::endl;
    return 0;
}
/*
output in 64-bit mechine:
sizeof Base 4
sizeof Derived 16
*/
{% endhighlight %}

和上一个例子相比,区别是继承方式变为虚继承，因为虚继承或者父类有virtual函数时，class会多一个虚函数表指针，在64位平台，一个指针是8位，因此Derived大小是16。c++对象模型不是本文重点，有兴趣的话可以看下《[C++对象模型][cpp_obj]》。

## 为什么要数据对齐

如今高级语言横行天下，也不用自己去管理内存，那么为什么还要学这些低层的内存布局类的知识呢？我认为学习低层原理和使用高级语言并无冲突，项目选择使用什么语言也是多方面的衡量与取舍，使用高级语言开发有库完善，开发效率快，上手相对容易等优点，而偏低层的语言则有效率和稳定性等优势，了解语言的原理有助于对语言的控制力，也是一条高手到大师的进阶。

扯回来，看为什么要考虑数据对齐问题：

1. 跨平台，不同架构的寻址方式不同，因此数据的布局也有相应的差异；
2. [性能][ibm]，时间上cpu的访问更快，空间上合理的内存布局能大大节省程序运行时内存占用；
3. 当然还有，对付笔试面试→_→；

## 扩展与联想

看完了结构体中数据的对齐，扩展下思维，数据成员在结构体里需要进行对齐和填充，那么在任何在内存上的数据是否需要进行对齐？比如函数形参在栈空间上的布局，局部变量，堆上变量又是如何布局的？原理很都是相似的。

## 总结

## 参考

* [https://en.wikipedia.org/wiki/Data_structure_alignment][wiki]
* 《Data alignment: Straighten up and fly right》：[http://www.ibm.com/developerworks/library/pa-dalign/][ibm]
* [http://en.cppreference.com/w/cpp/language/bit_field][bit_fields]
* [https://msdn.microsoft.com/en-us/library/2e70t5y1.aspx][pragma_pack_vc]
* [https://gcc.gnu.org/onlinedocs/gcc/Structure-Packing-Pragmas.html][pragma_pack_gcc]
* [http://stackoverflow.com/questions/92859/what-are-the-differences-between-struct-and-class-in-c][diff_struct_class]
* 《C++对象模型》：[http://www.cnblogs.com/skynet/p/3343726.html][cpp_obj]
* 《The Lost Art of C Structure Packing》：[http://www.catb.org/esr/structure-packing/][lost_art]

[wiki]:https://en.wikipedia.org/wiki/Data_structure_alignment
[ibm]:http://www.ibm.com/developerworks/library/pa-dalign/
[bit_fields]:http://en.cppreference.com/w/cpp/language/bit_field
[pragma_pack_vc]:https://msdn.microsoft.com/en-us/library/2e70t5y1.aspx
[pragma_pack_gcc]:https://gcc.gnu.org/onlinedocs/gcc/Structure-Packing-Pragmas.html
[diff_struct_class]:http://stackoverflow.com/questions/92859/what-are-the-differences-between-struct-and-class-in-c
[cpp_obj]:http://www.cnblogs.com/skynet/p/3343726.html
[lost_art]:http://www.catb.org/esr/structure-packing/