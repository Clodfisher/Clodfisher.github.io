---
layout: post    
title: 深度探索C++对象模型    
date: 2018-8-5    
tags: C++ 读书提炼           
---

### 前言    
最近目标是学习C++底层知识，因此找了一本《深度探索C++对象模型》来啃，以前也看过一些书，但由于没有进行总结和提炼，导致只有个浅显的印象，而没有个全局的统筹、知识点扩展、和已有知识点类比。为了解决自己这个问题，想到了一种解决办法，将所有看过的书进行一些记录，方便以后查阅，知识总结和形成自己的理论。对于此系列的文章，我将以读书提炼作为tags,所涉及到的内容，主要有以下方面：    
* 为什么要学习这本书，学习这本书要达到的目标和收获。      
* 每个章节所讲的内容，和自己的提炼和感悟。    
* 对整本书读后的感悟和收获。    
* 提炼出整本书的知识点。    

### 为什么，目标         
由于工作中对于C++只知道怎么用，对于C++底层的实现，比如虚函数表，成员变量，成员函数，继承和多态等如何在内存中布局，以及之间有和联系等，不太了解，只是知道有这回事。根据大家的推荐，这本书讲解的不错，因此拿来啃，以及进行知识的梳理和总结，提炼出设计思想，同时对工作中用到C++知识点的地方进行反思。希望通过这本书的研读，自己对于C++底层实现有个很好的认识；当长时间不用C++的时候，可通过底层原理推出如何使用C++语言，不用再进行C++语法浏览学习；同时对于里面所涉及到设计思想进行提炼和总结。    

### 每章概述和感悟    

<br>
#### **第一章：关于对象**    
  
##### 在C++代码中对于对象的描述大致分为三种：    
* 程序性（procedural）-程序模型        
  “数据”和“处理函数的操作”是分开来声明的，例如C语言的struct。  
* 抽象数据类型（abstract data type,ADT）-抽象数据类型模型       
  以函数重载的尝试实现某个类型数据对象的方法，例如operator引导的函数。    
* class层级封装（class hierarchy）-面向对象模型    
  有一些彼此相关的类型，通过一个抽象的base class封装起来，用以提供共同接口用于继承。例如点坐标类。      

##### C++对象模型是怎样：     
在C++对象中，有两种class data members:static和nostatic，以及三种class member functions:static、nostatic和virtual。在该模型中，Nostatic data members被配置于每一个class object之内，static data members则被存放于个别的class object之外。static和nostatic function members也被放在个别的class object之外。Virtual function则有以下两个步骤支持：    
1. 每个class产生一堆指向virtual functions的指针，放在表格之中，这个表格成为virtual table(vtbl)。   
2. 每个class object被按插一个指针，指向相关的virtual table。通常这个指针被成为vptr。vptr的设定和重置都由每一个class的构造、析构和拷贝赋值运算符自动完成。        
  以下以class Point为了进行说明：    
* 代码如下：        

```
class Point {    
public:    
  Point(float xval);       
  virtual ~Point();    
 
  float x() const;    
  static int PointCount();    

protected:    
  virtual ostream& print(ostream &os) const;    

  float _x;    
  static int _point_count;    
};  
```    

* 其对象模型如下所示：    
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel0.jpg)   
该模型的主要优点在于它的空间和存取时间的效率；主要缺点则是，如果应用程序代码本身未曾改变，但所用到的class objects 的nostatic data members有所修改，那么这些应用程序代码同样得重新编译。    

##### 多态        
###### 多态的作用：    
多态的主要用途是经由一个共同的接口来影响类型的封装，这个接口通常被定义在一个抽象的base class中。这个共享接口是以virtual function机制引发的，它可以在执行期根据object的真正类型解析出到底是哪一个函数实例被调用。      

###### 类对象内存表示     
需要多少内存才能表现一个class object?    
* 其nonstatic data members的总和大小。    
* 加上任何由于对齐的需求而填补上去的空间，可能存在于members之间，也可能存在于集合体边界。    
* 加上为了支持virtual而由内部产生的任何额外负担。     
以下以class基类源码为例进行说明：     
* 代码如下：
    
```
class ZooAnimal {    
public:    
  ZooAnimal();    
  virtual ~ZooAnimal();    
  
  //创建一个虚函数
  vitrual void rotate();    

protected:    
  int loc;    
  string name;    
};    

ZooAnimal za("Zoey");    
ZooAnimal *pza = &za;        
```   

* 对象和指针布局如下所示：       
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel1.jpg)     

以下以class继承类源码为例进行说明：     
* 代码如下：  
   
```
class Bear : public ZooAnimal{   
public:   
  Bear();    
  ~Bear();    

  //实现子类的虚函数，同时自己创建一个虚函数    
  void rotate();    
  virtual void dance();    

  //创建自己的成员变量    
  enum Dances {...}    
  Dances dances_known;    
  int cell_block;    
};    

Bear b("Yogi");    
Bear *pb= &b;    
Bear &rb = *pb;    
``` 

* 可能的内存布局如下所示：    
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel2.jpg)       

* 常见问题思索：    

1,说出如下代码中一个Bear指针和一个ZooAnimal指针有什么不同？    

```
Bear b;    
ZooAnimal *pz = &b;    
Bear *pb = &b;    
```    

答：    
它们都指向Bear object的第一个byte。其间的差别是，pb所涵盖的地址包含整个Bear object,而pz所涵盖的地址只包含Bear object中的ZooAnimal subobject。除了ZooAnimal subobject中出现的members，你不能使用pz来直接处理Bear的任何members(成员变量、成员函数、未覆盖基类的虚函数)。唯一例外是通过virtual机制。

2,说出如下代码中为什么rotate()所调用的是ZooAnimal实例而不是Bear实例？此外如果初始化函数将一个object内容完整拷贝到另一个object去，为什么za的vptr不执行Bear的virtual table？    

```
Bear b;    
ZooAnimal za = b;   //这里会引起切割（sliced）    

//调用ZooAnimal::rotate()
za.rotate();    
```    
答：    
第一个问题是，za并不是一个Bear,它是一个ZooAnimal。多态所造成的一个“一个以上的类型”的潜在力量，并不能够实际发挥在“直接存取objects”。多态不支持对于object的直接处理，只支持指针与引用类型。   
第二个问题是，编译器在初始化及赋值操作之间做了仲裁。编译器必须确保如果某个object含有一个或一个以上的vptrs,那些vptrs的内容不会被 base class object初始化或改变。    

3,不同类型的指针之间的差异？    
答：    
之间的差异，既不在其指针表示法不同，也不在其内容（代表一个地址）不同，而是在其所寻址出来的object类型不同，也就是说，“指针类型”会教导编译器如何解释某个特定地址中的内存内容及其大小。    

<br>
#### **第二章：构造函数语意学**   

##### 默认构造        
* 程序性 


<br>
持续更新中......    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [深度探索C++对象模型](https://clodfisher.github.io/2018/08/InsideTheC++ObjectModel/)      

