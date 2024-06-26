---
title: "C++基础"
date: 2024-03-26T18:59:47+08:00
lastmod: 2024-03-26T18:59:47+08:00
author: ["LittleStoneMax"]
keywords: 
- 
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- 
description: "C++面经"
weight:
slug: ""
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
reward: true # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---

# 基础
## 引用和指针的区别
1. 引用在定义时必须被初始化并且不能被初始化为NULL，指针没有要求
2. 引用不能改变引用关系（底层用指针常量实现），指针随意
3. 有多级指针没有多级引用
4. 自增自减表达的含义不同。指针自加即指针向后偏移一个类型的大小，引用自增即引用的实体自加
5. 指针用sizeof计算大小结果不同，指针在32位内存下是4字节，在64位内存下是8字节，引用结果为类型大小
6. 指针是一个实体，需要分配空间；引用是变量的别名，不需要分配内存空间
7. 访问实体的方式不同，指针需要显示解引用，引用编译器自己处理

>在语法概念上,引用就是一个别名,没有独立空间,和其引用实体共用同一块空间.但是在底层实现上实际是有空间的,因为引用是按照指针方式来实现的


---
## malloc free和new delete的区别
1. malloc和free是c语言中的库函数 new和delete是c++中操作符
2. new自动计算所需分配内存大小 malloc需要手动计算
3. new返回的是对象类型的指针 malloc返回的是void\*之后进行类型转换
4. new分配失败会抛出异常 malloc分配失败返回的是NULL
5. new是在freestore（自由存储区）上分配内存 malloc通常在堆上分配
6. delete需要对象类型的指针 free是void\*类型的指针

> malloc分配的是虚拟内存不一定有物理内存 若小于128k分配在堆上若大于128k分配在文件映射区

> new
> 1 operator new
> 2 申请足够的空间
> 3 调用构造函数初始化成员变量
> delete
> 1 先调用析构函数
> 2 operator delete
> 3 释放空间

----
## free(p)怎么知道该释放多大的空间
malloc在分配空间时会多分配16个字节用来存储内存块的描述信息
在free释放空间时会左偏移16个字节读取信息
## malloc出错会发生什么
出错原因：
一是物理内存或虚拟内存不足，即系统可提供的空闲内存无法满足程序的需求；
二是前面的程序出现了内存访问越界，导致malloc相关函数调用出现问题，这种情况下，再次使用malloc函数申请内存就可能失败
申请空间失败会返回NULL 会导致空指针

----
## 函数重载、重写、隐藏的区别
**重载（overload）**：在同一作用域中，同名函数的形式参数（参数个数，类型或顺序）不同时构成函数重载
```cpp
class A{
public:
	int func(int a);
	void func(int a,int b);
	void func(int a,int b,int c);
	int func(char* pstr,int a);
}
```
1. 函数返回值类型与是否构成函数重载无关，**不能通过函数返回类型不同来重载函数**
2. 类的静态成员函数与普通成员函数可以形成重载
3. 函数重载发生在同一作用域，如类成员之间的重载，全局函数之间的重载
**const重载**

```cpp
class B{
public:
	void funcA();
	void funcA() const;//可以通过const重载
	void funcB(int a);
	void funcB(const int a);//不行 编译过不了
};
```
funcA函数是合法重载而funcB函数是非法的不能通过编译
**重写/覆盖（override）**：派生类中与基类同返回值类型、同名和同参数的虚函数重定义，构成虚函数覆盖，也叫虚函数重写
重写的特殊情况--协变：当基类和派生类中该函数的返回值为父子关系的指针或者引用，返回值可以不同，这种情况称作协变
**隐藏（hiding）**：不同作用域中定义的同名函数构成隐藏（不要求函数返回值和参数类型相同）比如派生类成员函数隐藏与其同名的基类成员函数，类成员函数隐藏全局外部函数
>在子类中只要函数名和父类相同不是重写就是隐藏
>两个函数参数不同，无论基类函数是不是虚函数，都会被隐藏

```cpp
class Base{
public:
	void funA(){cout<<"funA()"<<endl;}
	virtual void funB(){cout<<"funB()"<<endl;}
};
class Heri:public Base{
public:
	void funA(){cout<<"funA():Heri"<<endl;}//函数隐藏因为不是虚函数
	void funA(int a){cout<<"funA(int a):heri"<<a<<endl;}//函数隐藏参数不同
	void funB(){cout<<"funB():heri"<<endl;}//函数重写
};
```
**三者对比**
1. 函数重载发生在相同作用域
2. 函数隐藏发生在不同作用域
3. 函数覆盖就是函数重写，是函数隐藏的特例

| 三者 | 作用域 | 有无virtual | 函数名 | 形参列表   | 返回值类型   |
| :--- | :----- | :---------- | :----- | :--------- | :----------- |
| 重载 | 相同   | 可有可无    | 相同   | 不同       | 可同可不同   |
| 隐藏 | 不同   | 可有可无    | 相同   | 可同可不同 | 可同可不同   |
| 重写 | 不同   | 有          | 相同   | 相同       | 相同（协变） |

----
## 深拷贝和浅拷贝？
c++中，浅拷贝不需要自己实现，编译器会自动生成默认的拷贝构造函数，浅拷贝新旧对象共享一块内存，任何一方的值改变都会影响另一方；深拷贝需要自己手动编写拷贝构造函数，浅拷贝新旧对象不共享内存
```ad-abstract
title:浅拷贝
collapse:close 
c++的浅拷贝是通过拷贝构造函数实现的，如果不重载拷贝构造函数和赋值函数，编译器将以浅拷贝的方式自动生成默认的函数，也就是在拷贝时简单的赋值某个对象的指针，容易造成问题
假设A类有两个对象a和b,a.data为"hello",b.data为"world"当将a的值赋给b时,可能有以下问题
1.b.data的内存没释放,造成内存泄露
2.b.data和a.data指向了同一块内存,a或b任何一方的值改变都会修改另一方的值
3.在对象被析构时,data被释放了两次


```
```ad-abstract
title:深拷贝
collapse:close 
深拷贝必须显式提供拷贝构造函数和赋值运算符,而且新旧对象不共享内存,在编写拷贝构造时会开辟一个新的内存空间
什么时候会使用深拷贝?
1.一个对象以值传递的方式传入函数体
2.一个对象以值传递的方式从函数体返回
3.一个对象需要通过另一个对象进行初始化

```
## 虚函数表和虚函数表指针的创建时机？
**虚函数表**：编译器编译的时候生成，发现virtual关键字修饰的函数后,虚函数表依赖类存在，不依赖对象
**虚函数表存储在:代码段(常量区)**
**虚函数表指针**：类对象构造的时候，把类的虚函数表地址赋值给vptr;类的对象前8个字节就是虚函数表的地址；在B继承A的情况下，虚函数表指针赋值过程为先调用基类构造函数，把A的虚函数表的地址赋值给vptr，再调用子类构造函数，把B的虚函数表的地址赋值给vptr;
## 构造函数和析构函数能不能是虚函数？
**构造函数不可以**
1. 因为创建一个对象时要确定对象的类型，而虚函数是在运行时确定其类型的，而在构造一个对象时，由于对象还未创建成功，编译器无法知道对象的实际类型。
2. 虚函数对应一个虚表，可是这个虚表其实是存储在对象的内存空间的。如果构造函数是虚的，就需要通过虚表来调用，可是对象还没有实例化，也就是内存空间还没有，怎么找虚表呢？所以构造函数不能是虚函数。
**析构函数可以**
1、在多态当中，基类的方法被定义成虚函数，才可以通过基类指针动态调用派生类的方法，同理当我们delete 基类指针，如果基类析构函数不是虚函数，就无法动态调用到派生类的析构函数，导致派生类的对象无法析构，造成内存泄漏。反之，基类析构函数被定义成虚函数的时候，delete基类指针时，会先析构派生类对象，再析构基类对象
## 多态是怎么实现的？
**静态多态**：
编译阶段实现，其原理是由函数重载实现，通过不同的实参调用其相应的同名函数。
**动态多态**：
必须通过基类的指针或者引用调用
被调用的必须是虚函数，且在派生类中实现了该虚函数的重写 （注意：只有虚函数才有重写这个概念）
动态多态的实现是依靠不同对象的虚函数指针指向的虚函数表实现的，因为里面存储了对应的虚函数
对象的指针或者引用调用虚函数时，不是编译时决议，而是运行时决议，即程序开始运行时到指定的对象虚函数表中去调用对应的虚函数，所以指向基类对象，调用是基类虚函数，指向派生类对象，调用是派生类虚函数
>1.先将基类中的虚表内容拷贝一份到派生类虚表中
>2.如果派生类重写基类中某个虚函数，派生类自己的虚函数覆盖虚表中基类的虚函数
>3.派生类自己新增加的虚函数按其在 派生类中的声明次序增加到派生类虚表的最后
## C和C++的区别 
C++语言支持函数重载，而C不支持函数重载

## C ++编译过程
1. **预处理阶段**：引入头文件，宏替换，**删除注释**，生产以i为结尾的**预编译文件**
2. **编译阶段**：检查语法错误，没有错误将代码解释为汇编，生成以s为结尾的**汇编文件**
3. **汇编阶段**：将汇编代码解释为二进制的cpu指令，生成以o结尾的**可重定向目标文件**
4. **链接阶段**：目标文件不能直接执行，因为某个.cpp文件调用另外一个.cpp文件，为了解决这类问题，调用目标文件与被调用的目标文件连接起来，最终得到可执行程序exe，链接分为静态静态链接和动态链接
	- **静态链接**：目标文件直接拷贝到可执行文件中的链接方式，使用静态链接生成静态库，静态库对函数库的链接是在**编译时期**完成的
	- **动态链接**：**程序运行**时动态加载目标文件的链接方式
## C++有几种传参方式，区别是什么？
值传递：形参是实参的拷贝，在外部某个函数中改变形参的值并不会影响主函数内的数据
指针传递：形参为指向实参地址的指针，当对形参的指向操作时，就相当于对实参本身进行的操作，函数传递的时候只传入一个指针占用四个字节的空间
引用传递：形参相当于是实参的“别名”，对形参的操作其实就是对实参的操作，在引用传递过程中，被调函数的形式参数虽然也作为局部变量在栈中开辟了内存空间，但是这时存放的是由主调函数放进来的实参变量的地址
## include头文件的顺序以及双引号和尖括号的区别？
尖括号<>的头文件是系统文件（标准库），双引号""的头文件是自定义文件
使用尖括号的查找路径：编译器设置的头文件路径->系统变量
使用双引号的查找路径：当前头文件目录->编译器设置的头文件路径->系统变量
## 常量指针和指针常量的区别
```cpp
int a = 1;
int b = 2;
int c = 3;
int const *p1 = &b;//const在前,p1为常量指针
int *const p2 = &c;//*在前，p2为指针常量
p1 = &a;//正确，p1是常量指针，可以指向新的地址(即&a),即p1本身可以改变
*p1 = a;//错误，*p1是指针p1指向对象的值,不可以改变，因此不能对*p重新赋值
p2 = &a;//错误，p2是指针常量，本身不可以改变，因此将a的地址赋给p2是错误的
*p2 = a;//正确，p2指向的对象允许改变
```
常量指针p1：即指向const对象的指针，指向的地址可以改变，但其指向的内容（即对象的值）不可以改变
指针常量p2：指针本身是常量，即指向的地址本身不可以改变，但内容（即对象的值）可以改变
## 声明、定义和初始化
声明：声明主要用于向程序表明变量的类型和名字，它不会为变量分配存储空间，可多次声明
定义：定义则是为变量分配存储空间，同时可以为变量指定初始值。当定义一个变量时，同时声明了它的类型。对于变量而言，定义只能有一次
初始化：初始化则是为变量赋初值
## vector插入元素，进行几次内存拷贝？

```ad-note
title:test
collapse: open
icon: triforce
color: 200,50, 200
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod nulla.
```

## 内存泄漏、内存溢出和内存越界
内存泄漏：程序中已动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费
内存溢出：系统中存在无法回收的内存或使用的内存过多，最终使得程序运行要用到的内存大于能提供的最大内存
内存越界：程序在运行过程中访问了不在预分配的内存空间或者超出了已分配内存空间的大小
## 栈区会内存泄漏吗？
栈区一般不会发生内存泄露
## 如何解决栈溢出？
1. 减少函数递归层次
2. 减少局部变量数量
3. 扩大栈的空间
## 空指针和野指针
**野指针**：指针指向的空间是不可知的、随机的、没有明确限制的
产生野指针的三种情况：
1. 指针未初始化
2. 指针越界访问  
3. 指针指向的空间释放
规避野指针：
1. 指针初始化  
2. 小心指针越界  
3. 指针指向空间释放及时置NULL  
4. 避免返回局部变量的地址  
5. 指针使用之前检查有效性
**空指针**：指向对象为空的指针
空指针p并不表示指向0x00000000地址的存储单元，而是表示不指向任何有效空间，虽然输出是如此
# 内存相关
## 堆和栈的区别
1. **堆栈空间分配不同**：栈由操作系统自动分配释放，存放函数的参数值，局部变量的值等；堆一般由程序员分配释放。
2. **堆栈缓存方式不同**：栈使用的是一级缓存，它们通常都是被调用时处于存储空间中，调用完毕立即释放；堆则是存放在二级缓存中，速度要慢些。
3. **堆栈数据结构不同**：堆类似数组结构；栈类似栈结构，先进后出。
## 内存管理
内存分成5个区，他们分别是堆、栈、自由存储区、全局/静态存储区和常量存储区。
1. **栈**：在执行函数时，函数内局部变量的存储单元都可以在栈上创建，函数执行结束时这些存储单元自动被释放
2. **堆**：由new创建的内存块，一般一个new对应一个delete
3. **自由存储区**：由malloc分配的内存块，类似于堆，用free释放
4. **全局/静态存储区**：全局变量和静态变量被分配到同一块内存
5. **常量存储区**：存放常量，不允许修改
## 内存泄漏
常见的内存泄漏：
1. new和malloc申请资源使用后，没用delete和free释放
2. 子类继承父类时，父类析构函数不是虚函数
3. windows句柄资源使用后没有释放
## 程序有哪些section，分别的作用？程序启动的过程？怎么判断数据分配在栈上还是堆上？
![[Pasted image 20240321202842.png]]
从低地址到高地址，一个程序又代码段、数据段、BSS段、堆、共享区、栈等组成
1. 数据段：存放程序中已经初始化的全局变量和静态变量的一块内存区域
2. 代码段：存放程序执行代码的一块内存区域，只读，代码段头部还包含一些只读的常数变量
3. BSS段：存放程序中未初始化的全局变量和静态变量的一块内存区域
4. 堆区：动态申请内存使用，堆从低地址向高地址增长
5. 共享区：位于堆和栈之间
6. 栈区：存储局部变量、函数参数值。栈从高地址向低地址增长，是连续的空间

# 关键字
## const关键字
const:常量限定符，通知编译器该变量不可修改
const修饰一般常量和数组
```cpp
int const a = 100;
const int a = 100; //与上面等价
int const arr [3] = {1,2,3};
const int arr [3] = {1,2,3};//与上面等价

```
```cpp
char *p = "hello";     // 非const指针非const数据
const char *p = "hello";  // 非const指针,const数据
char * const p = "hello";   // const指针,非const数据
const char * const p = "hello";  // const指针,const数据
```
## inline关键字（内联）
编译器在编译阶段完成对inline函数的处理，即对inline函数的调用替换为函数的本体，但这只是对编译器的建议，编译器可以这样做，也可以不这样做
1. 将inline函数体复制到inline函数调用处
2. 为所用inline函数中的局部变量分配到内存空间
3. 将Inline函数的输入参数和返回值映射到调用方法的局部变量空间中
4. 如果inline函数有多个返回点，将其转变为inline函数代码块末尾的分支
**inline通过消除调用开销来提升性能**
相比宏函数的优点：
- 内联函数同宏函数一样在被调用处进行代码展开，省去了参数压栈、栈帧开辟和回收、结果返回等，从而提高程序运行速度
- 内联函数相比宏函数来说，在代码展开时，会做安全检查或自动类型转换（同普通函数），而宏定义不会
缺点：
- 代码膨胀，空间换时间
- inline函数无法随着函数库升级而升级
- 是否内联，程序员不可控，inline关键字只是对编译器的建议，决定权在编译器
```ad-tip
- 使用函数指针调用内联函数会导致内联失败
- 如果函数代码过长或者有多重循环语句,if或witch分支语句或递归时,不宜用内联
- 类的构造函数、析构函数和虚函数不适合inline函数
```
## explicit关键字
explicit用于修饰只有一个参数的类构造函数，表明该构造函数必须显式调用，禁止隐式转换
# C++11
## 左值引用和右值引用
引用：是变量的别名，不占内存空间，声明时必须初始化（不能为NULL），通过引用
		修改变量值
左值：可以在等号两边，能取到地址的有名字的都是左值
		变量名、返回左值引用的函数调用、前置自增/自减，解引用，赋值运算等
右值：只能在等号右边，不能取到地址没有名字的是右值
		纯右值：字面值、返回非引用类型的函数调用，后置自增/自减，各种表达式
		将亡值：c++11新引入的与右值引用（移动语义）相关的值类型，用来触发移动
		构造或移动赋值构造，并进行资源转移，之后调用析构函数
左值引用：对左值的引用，避免对象拷贝，函数传参，函数返回值
右值引用：对右值的引用，实现移动语义，实现完美转发
const左值引用能指向右值（不能修改这个值），为此引入右值引用
右值引用通过std::move()可以指向左值
声明出来的左值引用或右值引用都是左值
# STL

