---
title: 《C++ Primer》 读书笔记 7
date: 2017-11-20 21:00:00
tags:
categories: 
- C++
- 读书笔记
description: Momenta实习生二面，竟然被一道初始化列表和构造函数的问题给问崩了，唉，天下武功，为什么偏偏要选C++。
---
<!-- more -->

# 摘要
Momenta实习生二面，竟然被一道初始化列表和构造函数的问题给问崩了，唉，天下武功，为什么偏偏要选c++。
本篇博客将记录在恶补《C++ Primer》第七章时的重点内容。


# 常量成员函数

```
class ScaleData{

private:
    string bookNO;

public:
    ScaleData(string str):bookNO(str){}
    string isbn() const { return "const_method " + bookNO; }
    string isbn() { return "no_const_method " + bookNO; }
};
```
在 ScaleData 类中，有两个 isbn() 方法，第一个 isbn() 方法后面跟了一个 const 修饰符，这个 isbn() 因此被称为常量成员函数。那么这个函数的“常量性”体现在哪呢？首先得从普通的成员函数说起，即第二个 isbn() 。在调用成员函数时，成员函数会通过一个名为 this 的额外的隐式参数，来访问调用它的那个对象。也就是说 isbn() 对 bookNO 变量的访问实际上是通过 this->bookNO 的方式得到的。this总是指向调用函数的对象本身，无法指向其他对象，因此是常量指针。但是 this 不是常量类型的，即可以修改 this 对象内部的变量，比如 bookNO 变量。如果要使 this 变成常量类型的常量指针，即不能修改 this 对象内部的变量，要该怎么办？常量成员函数就是用来解决这个问题的。也就是说 c++ 中通过常量成员函数，来保证 this 指针是常量类型的，从而无法更改其对象内部的变量。总结起来有以下几点规则：
- 常量对象只能调用常量成员函数，无法调用普通成员函数。
- 普通对象即可以调用常量成员函数，也可以调用普通成员函数（优先）。
- 常量成员函数可以访问常量成员变量和普通成员变量，但不允许修改普通成员变量。
- 普通成员函数只能访问普通成员变量，不允许访问常量成员变量。



# 构造函数与初始化列表
构造函数用于对类的成员变量进行初始化。只要类的对象被创建，就会执行构造函数。
构造函数不能被声明为 const 的，当创建类的一个const对象时，直到构造函数完成初始化过程，对象才能真正取得其“常量”属性。
当一个类不存在构造函数时，编译器会为其生成默认的构造函数。默认构造函数一般通过以下步骤进行初始化：
- 如果存在类内的初始值，用它来初始化成员变量。
- 否则，默认初始化。对于内置类型或复合类型，其初始化方式未定义。对于类对象，则利用默认构造函数进行初始化(其实是在默认初始化列表里完成的)。

初始化列表可以认为是构造函数的一部分。因此构造函数的执行可以分为两个阶段：初始化阶段和计算阶段。初始化阶段即执行初始化列表，计算阶段即执行构造函数的函数体。初始化阶段先于计算阶段。
初始化阶段将会对所有非静态类类型的成员变量进行初始化，如果该成员变量没有出现在构造函数的初始化列表中（此时调用该成员的默认构造函数）。如果该成员变量出现在初始化列表中，则会调用对应参数的构造函数或者是拷贝构造函数。
计算阶段则只能进行赋值操作进行初始化。

**由于初始化阶段先于计算阶段执行，分别利用这两者进行初始化会存在性能上的差别。**
对于内置类型，使用初始化列表和构造函数体进行初始化差别不大。
但是对于类类型，最好使用初始化列表。采用初始化列表进行初始化只会调用一次对应参数的构造函数或者是拷贝构造函数，而采用构造函数体进行初始化将会调用一次默认构造函数和一次赋值构造函数。

**由于某些类类型对象的特殊性，对于某些成员变量必须采用初始化列表进行初始化。**
1. 初始化一个reference成员变量。
2. 初始化一个const成员变量。
3. 成员变量对应的类不存在默认构造函数。
4. 基类不存在默认构造函数。


# 类的静态成员
类的静态成员包含静态成员变量和静态成员函数。
类的静态成员通过 static 关键字使得其与类关联在一起，也就是说类的静态成员在于任何对象之外。因此无法通过 this 指针对静态成员变量和静态成员函数进行访问。静态成员函数也不能被声明成const的，也不能在静态成员函数内部使用 this 指针。

对于静态成员，可以直接使用类名加作用域运算符（::）进行访问。
虽然静态成员不属于类的某个对象，但是我们仍然可以使用类的对象、引用或者指针来访问静态成员。
> 其实这一点还是有点奇怪，毕竟在调用成员函数时，会通过 this 指针。对于静态成员函数，应该有另一套实现机制。

与普通成员函数一样，静态成员函数可以在类的内部定义，也可以在类的外部定义。但要注意的是，当在类的外部定义静态成员函数时，不能重复 static 关键字，该关键字只出现在类的声明语句。

而对于静态成员变量，因为其不属于类的任何一个对象，因此不能在构造函数中对静态成员变量进行初始化。而且一般来说也不能在类的内部进行静态成员变量的初始化（const static 变量是例外）。对于静态成员变量的初始化，必须在类的外部定义和初始化。
即使一个常量静态变量在类的内部被初始化了，通常情况下也应该在类的外部定义一下该成员。

**静态成员与普通成员**
静态成员变量的第一个特性是，静态成员变量可以是不完全类型（声明之后，定义之前）。特别地，静态成员的类型可以就是它所属的类类型。而非静态数据则受限制，只能声明成它所属类的指针或引用。

静态成员变量的第二个特性是，静态成员变量可以作为默认参数，而非静态成员函数不能作为默认参数。


# 参考文献
《C++ Primer 第5版》







