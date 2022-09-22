---
title: '方法调用的编译和运行: dispatch'
tags:
  - 编程
categories: []
toc: false
uuid: cba15320-2bd8-1ba2-4edd-d20f497df431
---

## 1. 背景
静态分派(static dispatch)和动态分派(dynamic dispatch)是用来处理编程语言语言方法调用的两种计算机制.
一个方法是如何被调用的,这两种机制在编译期和运行时分别做了什么,他们各自的优缺点是什么,分别适用于什么样的场景,让我们带着问题看下去吧.
## 2. dispatch介绍
我们都知道一个方法会在运行时被调用,一个方法被唤起,是因为编译器有一个计算机制,用来选择正确的方法,然后通过传递参数来唤起它.
这个机制通常被成为分派(dispatch). 分派就是处理方法调用的过程.
分派在处理方法调用的时候,可能会存在多个合理的可被调用的方法列表,此时就需要去选择最正确的方法.选择正确方法的整个过程,就是人们熟知是分派(dispatch).
每种编程语言都需要分派机制来选择正确的唤起方法.

方法从书写完成到调用完成,概括上会经历编译期和运行期两个阶段,而前面说的确定哪个方法被执行,也是在这两个时期进行的.
选择正确方法的阶段,可以分为编译期和运行期,而分派机制通过这两个不同的时期分为两种: 静态分派(static dispatch)和 动态分派(dynamic dispatch).
## 3. static dispatch
static dispatch是在编译期就完全确定调用方法的分派方式.它是一种方法分派形式.用于描述一个语言或者环境是如何选择被调用的方法的实现的.
例如结合了函数重载的C++的模板和其他语言的泛型,都是这样实现的.

用于在多态情况下,在编译期就实现对于确定的类型,在函数调用表中推断和追溯正确的方法,包括列举泛型的特定版本,在提供的全部函数定义中选择的特定实现.

与static dispatch相反的dymanic dispatch,是基于运行期的给定信息来确定调用方法的,可能通过虚函数表实现,也可能借助其他的运行期的信息.dynamic dispatch的细节我们会在下面进行详细说明.
static dispatch可以确保某个方法只有一种实现.static dispatch明显的快于dynamic dispatch,因为dynamic dispatch本身就意味着较高的性能开销.
## 4. 何时使用static dispatch
static dispatch在编译期确定需要调用的方法,在运行期进行调用.所有的编程语言都是支持static dispatch的,不同语言默认的分派方式不同,有的默认为static dispatch,有的默认是dynamic dispatch.
有的语言可以通过声明关键字,来标明使用static dispatch,比如final或private或static等.这样用于避免基类的方法属性等不被子类修改.
## 5. static dispatch如何实现
在编译器确定使用static dispatch后,会在生成的可执行文件内,直接指定包含了方法实现内存地址的指针.在运行时,直接通过指针调用特定的方法.这是static dispatch的标准做法.
static dispatch还可以进行进一步优化,优化的一种实现方式叫做内联(inline:inline expansion).inline是指编译期从指定被调用的方法指针,改为将方法的实现平铺在调用方的可执行文件内.下面就讲一下inline具体是如何实现的,它有什么优缺点.
## 6. inline
inline也叫内联展开,它可以人为声明,也可以通过编译器优化来实现.inline是将被调用方法的指针替换为方法实现体.inline的具体实现其实就是内联展开,它和宏展开(macro expansion很像.
内联展开和宏展开的区别在于,内联发生在编译期,并且不会改变源文件.但是宏展开是在编译前就完成的,会改变源码本身,之后再对此进行编译.
内联是一种非常重要的优化方式,但是内联对于性能的影响比较复杂.从经验法则来讲,有些内联可以通过很小的内存消耗来提升运行速度.但是无节制的内联,也可能会降低速度,因为内联的代码需要大量的CPU 缓存,并且也会消耗内存空间.

内联方法的运行比传统的方法调用要快一些,因为节省了指针到方法实现体的调用的消耗,但是会带来一些内存损失.如果一个方法被内联10次,那么会出现10份方法的副本.所以内联适用于会被频繁调用的比较小的方法.在C++中,如果方法通过class去定义,默认使用内联(不需要inline关键字).其他情况想要使用内联,需要标明inline的关键字.
但是如果一个方法特别大,被inline关键字修饰的话,编译器也可能会选择不适应内联实现.
所以inline关键字是一个desire声明而非require.只能告诉编译器倾向使用内联方式,但是最终实现是编译器决定的.
## 7. 内联的作用
内联可以用于消减方法被调用的时间.非常适用于会被频繁调用的方法.如果方法本身很小的话,可以降低内存上的消耗.内联还为进一步的编译优化提供了基础.program optimization
编译器一般会将陈述式进行内联.
在[函数式编程语言]
内联在性能方面会有提升,而且还可以基于内联,做进一步的编译优化,感兴趣的可以参考内联文章的Effect on performance,Compiler support和Implementation部分,这里就不展开说了.
## 8. dynamic dispatch
在计算机科学中,dynamic dispatch是 用于在运行期选择调用方法的实现的流程.
dynamic dispatch被广泛应用,并且被认为是面向对象语言(Object-Oriented programming:OOP)的基本特性.

OOP是通过名称来查找对象和方法的.但是多态就是一种特殊情况了,因为可能会出现多个同名方法,但是内部实现各不相同.如果把OOP理解为向对象发送消息的话.在多态模式下,就是程序向不知道类型的对象发送了消息,然后在运行期再将消息分派给正确的对象.之后对象再确定执行什么操作.

与static dispatch在编译期确定最终执行不同,dynamic dispatch的目的是为了支持在编译期无法确定最终最合适的实现的操作.这种情况一般是因为在运行期才能通过一个或多个参数确定对象的类型.例如 B继承自A, 声明var obj : A = B(),编译期会认为是A类型,但是真正的类型B,只能在运行期确定.

dynamic dispatch和late binding(也叫做dynamic binding)不同,程序使用Name binding通过名字去关联操作.在多态操作中,会有多个不同的操作关联同一个方法名.这个绑定关系可以在编译期或者运行期确定.在dynamic dispatch中,是在运行期去选择方法的实现的.
虽然late binding的绑定实现也是在运行期才能确定,但是dynamic dispatch并不意味着late binding,late binding也不等同于dynamic dispatch.之后会详细讲解late binding,这里先挖个坑.

#### single and multiple dispatch
通过对象类型去选择调用方法的模式,叫做single dispatch,这是面向对象语言普遍支持的一种方式,比如C++,Java,Objective-C,Swift,JavaScript,Python等.
在下面示例的方法调用中,参数divisor是可选类型.
```language
dividend.divide(divisor)  # dividend / divisor
```
我们是向对象dividend发送一个包含参数divisor的名为divide消息.在选择方法实现时,只会通过dividend,也就是消息对象的类型来进行选择.忽略参数divisor的类型.这种方式叫做single dispatch.
与此相反,一些语言(比如Common Lisp, Dylan, Julia)在方法分派时,会根据组合的信息进行判断.拿上面例子来说,是通过接收对象dividend和参数divisor一起来决定那个divide方法会被调用.这种方式就成为multiple dispatch
#### multiple dispatch
Multiple dispatch和single dispatch不同之处在于,multiple dispatch也会根据方法名结合方法的参数,一起来判断需要执行的方法.
在命名方法的时候,一般会使用符合方法目的的描述.当方法目的相似时,我们也会使用同名但是不同入参的方式进行命名.这种情况时,只使用方法名去查找实现的方式就不够充分了.这时候也需要用到入参,通过入参的个数和类型去进行判断.
对于single dispatch语言来说,在调用方法时,参数会被特殊处理用于去选择方法实现,在很多语言中,这种特殊参数是在语法上就进行指明的.例如,很多语言会把特殊参数放在调用语言之前special.method(other, arguments, here)
但是在multiple dispatch中,选择方法时,也会对参数的个数和类型进行匹配.没有特殊参数去持有方法调用.
#### dynamic dispatch的实现机制
一种语言可能有多种dynamic dispatch的实现机制.语言的特性不同,动态分派的实现也各有差异.下面我们只针对一种实现虚函数表(vtable: virtual function table)来进行详细说明.
这里提一句,因为动态分派经常会引起高性能消耗,所以很多语言对某些特定的方法,提供了静态分派的方式.
虚函数表
虚函数表是用于支持动态分派的一种实现机制.
当一个类定义了虚函数virtual function之后,大部分编译器会对类增加一个隐藏的属性,属性指向一个包含了虚函数表,表内包含被收纳了调用方法的指针数组.这些方法指针用于在运行期来调用正确的方法实现.
用来实现动态分派的方式有很多,虚函数是在C类语言,例如C++中最普遍的实现方式.

Java所有的实例方法都默认使用虚函数表实现.因为所有方法都可以被子类重载使得类变得特别复杂.当类不可被继承时,理论上是不需要虚函数表的.所以当使用final或private等静态修饰符去修饰时,编译器就可以放心的去使用static dispatch.

Python是不支持static dispatch的.实际上Python所有的方法和属性的实现都使用了late binding.

虚函数表实现
对象的虚函数表包含对象绑定的方法地址.方法的调用需要从虚函数表内获取方法地址.同一个类的所有对象,生成的虚函数表都是一样的.属于同一系列的派生类,他们对象的虚函数表都有相同的布局.同一个方法在表内的位移都是相同的.所以,在知道了方法的位移之后,就可以通过虚函数表直接获取正确的方法.

编译器会为每个类创建单独的虚函数表.当对象创建后,会生成一个隐藏对象,对象是一个指向虚函数表的指针.编译器也会生成包含了虚函数表指针的代码.
在不同的语言中,虚函数表可能在对象的最后或者第一个属性内,这不影响实际的功能实现.

虚函数表示例
以C++为例.

class B1 {
public:
  virtual ~B1() {}
  void f0() {}
  virtual void f1() {}
  int int_in_b1;
};

class B2 {
public:
  virtual ~B2() {}
  virtual void f2() {}
  int int_in_b2;
};
下面是它们的派生类D的声明

class D : public B1, public B2 {
public:
  void d() {}
  void f2() {}  // override B2::f2()
  int int_in_d;
};
之后创建对象

B2 *b2 = new B2();
D  *d  = new D();
GCC的g++3.4.6编译之后,b2生成了如下的32位内存结果

b2:
  +0: pointer to virtual method table of B2
  +4: value of int_in_b2

virtual method table of B2:
  +0: B2::f2()   
分析一波内存分布.b2的起始位置是B2类虚函数表的指针.占4个字节.之后是一个int类型.

下面看看d的内存分布

d:
  +0: pointer to virtual method table of D (for B1)
  +4: value of int_in_b1
  +8: pointer to virtual method table of D (for B2)
 +12: value of int_in_b2
 +16: value of int_in_d

Total size: 20 Bytes.

virtual method table of D (for B1):
  +0: B1::f1()  // B1::f1() is not overridden

virtual method table of D (for B2):
  +0: D::f2()   // B2::f2() is overridden by D::f2()
d是通过多继承,B1和B2的派生类.可以看到内存中,前面几个部分和父类的内存结构完全一样.后面添加了自定义的属性.
需要注意的是,没有通过virtual关键字声明的方法f0()和d(),都没有出现在虚函数表内.可能对于缺省构造函数会有一些特殊操作,这里不展开说.
通过D重载的B2的f2()方法,是在B2的虚函数表内,将过去的B2::f2()的指针替换为D::f2()的指针.

多继承和thunks
可以看到g++编译器实现通过使用B1和B2两个虚函数表来实现多继承.每个表用来说明每个基类.这里就需要说到指针修正,也叫做thunks

D  *d  = new D();
B1 *b1 = d;
B2 *b2 = d;
代码执行后,d和b1会指向同一个内存地址,但是b2会指向d+8.所以b2所指向的d的内存范围会"看起来"像是B2的实例.也就是说,和B2拥有相同的内存布局.

方法调用,在多继承中的实现
在单继承中,调用一个d->f1()方法.可以分解为如下的伪代码

(*((*d)[0]))(d)
*d是D的虚函数表.[0]代表虚函数表内的一个方法.参数d为对象的this指针.
在多继承情况下,调用B1::f1()和D::f2()就会复杂很多.

(*(*(d[+0]/*pointer to virtual method table of D (for B1)*/)[0]))(d)   /* Call d->f1() */
(*(*(d[+8]/*pointer to virtual method table of D (for B2)*/)[0]))(d+8) /* Call d->f2() */
方法d->f1(0的调用会把B1当做参数传入.方法d->f2(0的调用会把B2当做参数传入.第二个调用,就会用到指针修正来指向正确的指针.因为B2::f2的地址并不在D的虚函数表中.
比较来看,d->f0()的调用就简单很多

(*B1::f0)(d)
虚函数表的性能分析
相比非虚函数调用的直接跳转到编译指针,虚函数表调用至少需要一次额外的索引重定向,有时还需要进行指针修正.所以虚函数表的调用一定是慢于非虚函数调用的.
此外,在不能使用即时编译的环境中,虚函数调用一般是不能够inline的.虽然有一些不常见的内联方式,这里就不展开了.
为了避免额外的性能消耗,编译器会通过计算,如果调用可以在编译期确定,那么就不会创建虚函数表.
所以,对于上面示例中f1的调用就可以不需要查表操作.因为编译器可以分辨d只会持有D类型的指针,而D没有重载f1.或者编译器(或优化器)也可以发现B1的所有子类都没有重载f1.所以B1::f1 或 B2::f2的调用因为可以确定它的最终实现,就可以不用查表.(虽然仍需要对'thsi'指针进行修正).

late binding
美国计算机科学家Alan Kay曾经说过,OOP对我来说,意味着几个方面:消息,本地存储,保护机制,状态流的隐藏,和极致的late binding of all things.

后期微软又大程度的对他们面向OOP的COM库进行了升级,COM也同样提升了early binding和late binding,要知道,很多语言在语法层面是同时支持这两种特性的.
什么是late binding呢?
late binding(也叫dynamic binding或dynamic linkage)是一种用于处理在运行时通过对象调用方法或者通过函数名去调用包含参数的方法的一种编程机制.
对于OOP语音的early binding或 static binding来说,在编译阶段就处理了所有的变量和表达式.通常这些数据存储在编译程序的虚函数表内,通过位移的方式获取,非常高效.对于late binding而言,编译器不会解读足够的信息去确认方法是否存在也不会将其绑定到虚函数表内.late binding是在运行时通过方法名去查找的.

在组件对象模型编程中,使用late binding的最大优势在于,不要求编译器在编译期间去引用包含对象的库.这使得编译过程可以更有效的去避免类的虚函数表突然更改带来的冲突.

late binding的实现
大部分的动态类型
)语言都可以在运行时去修改对象的方法列表,因此就需要late binding.

说说OC运行时
苹果官方对于OC dynamic binding文档中指出,dynamic binding就是在运行期来决定方法调用的实现.dymanic binding也叫做late binding.在OC中所有的方法都是在运行期动态判断的.真正执行的方法是通过方法名和接收对象一起来确定的.