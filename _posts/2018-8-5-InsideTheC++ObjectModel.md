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
  
##### **在C++代码中对于对象的描述大致分为三种**：    
* 程序性（procedural）-程序模型        
  “数据”和“处理函数的操作”是分开来声明的，例如C语言的struct。  
* 抽象数据类型（abstract data type,ADT）-抽象数据类型模型       
  以函数重载的尝试实现某个类型数据对象的方法，例如operator引导的函数。    
* class层级封装（class hierarchy）-面向对象模型    
  有一些彼此相关的类型，通过一个抽象的base class封装起来，用以提供共同接口用于继承。例如点坐标类。      

##### **C++对象模型是怎样**：     
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

##### **多态**        
###### **多态的作用**：    
多态的主要用途是经由一个共同的接口来影响类型的封装，这个接口通常被定义在一个抽象的base class中。这个共享接口是以virtual function机制引发的，它可以在执行期根据object的真正类型解析出到底是哪一个函数实例被调用。      

###### **类对象内存表示**     
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

###### **常见问题思索**：        

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

##### **默认构造**        
###### **默认构造定义**    
对于classX,如果没有任何用户定义的构造函数，那么会有个默认构造（没有参数的构造函数）被隐式声明出来，若这个声明出来的构造函数，只是简单的申请内存不做一些其它的操作，如调用其它构造函数、设置虚拟指针或虚拟基类指针等，那么这样的声明是无用的声明，即不是被编译器合成的那种，而被编译器合成的那种成为有用默认构造函数。     
* 无用 ： 只简单的申请内存，不进行任何初始化操作。        
* 有用 ： 调用其它构造函数，虚拟指针，虚拟基类指针等，合成到默认构造函数或用户定义的一些构造函数。         

###### **C++新手一般常见的误解**       
* 任何class 如果没有定义默认构造，就会被合成一个来。   
* 编译器合成出来的默认构造，会显示设定class内每一个数据成员的默认值。      

###### **常见问题思索**    
1，用户定义了默认构造函数，编译器还会进行合成吗？    
答： 如果class A需要做有用的合成，那么编译器会扩展已存在的构造函数，在其中安插一些代码，在用户代码（花括号里面）执行之前。    

2，若是没有无参默认构造，只有用户自定义的其它默认构造函数，若是有需要合成的代码，会被合成吗？    
答： 编译器会扩展现有的每一个构造函数，将不会合成一个新的默认构造函数，因为用户提供了其它构造函数。    

##### **拷贝构造**       
###### **调用拷贝构造的常见操作**    
1，对一个object做显示的初始化操作，如下所示：    
```
class X{...};    
X x;
X xx = x;   //调用拷贝构造    
X xxx(x);   //调用拷贝构造    
```    

2，当object被当做参数交给某个函数时，如下所示：    
```
extern void foo(X x);
void bar()
{
  X xx;
  
  //以xx作为foo()第一个参数的初始值（隐式的初始化操作）
  foo(xx);
  
  //...
}
```

3, 函数返回一个class object时，例如：   
```
X foo_bar()
{
  X xx;
  //...
  return xx;
}
```

###### **拷贝构造定义**    
如果class没有声明一个拷贝构造，就会有隐式的声明或隐式的定义出现。其也区分无用和有用两种，只有有用的实例才能被合成于程序之中。决定一个拷贝构造是否为无用的标准在于class是否展现出所谓的位逐次拷贝。    

###### **类不展现出位逐次拷贝的条件：**    
1. 当class内含一个member object而后者的class声明有一个copy constructor时（无论是被class设计者显示地声明，或是被编译器合成）。    
2. 当class继承自一个base class而后者存在一个copy constructor时（再次强调，不论是被显示声明或是被合成而得）。    
3. 当class声明一个或多个virtual functions时。    
4. 当class派生自一个继承串链，其中有一个或多个virtual base class时。    

###### **拷贝构造和拷贝赋值区别**    
1. 拷贝构造函数是对象被创建时调用。    
2. 拷贝赋值函数只能被以存在的对象调用。    

```
string a("abc")；  //自定义构造函数执行
string b = a;      //自定义拷贝构造函数执行
string c(b);       //自定义拷贝构造函数执行
c = a;			   //自定义拷贝赋值函数执行
```

##### **参数列表**    
###### **参数列表的作用**    
初始化列表是指示成员初始化方式，即对所有成员变量、基类申请内存，再根据指示方式或未指示方式进行初始化。 以在类中声明的顺序进行初始化，并且在用户显示定义的代码之前。       

###### **必须使用初始化列表的情况**    
1. 当初始化一个reference member时。    
2. 当初始化一个const member时。    
3. 当调用一个base class的构造函数，而它拥有一个参数时。
4. 当调用一个member class的constructor，而它拥有一组参数时。    

###### **使用初始化列表的好处**    
使用初始化列表主要是基于性能问题，对于内置类型，如int, float等，使用初始化类表和在构造函数体内初始化差别不是很大，但是对于类类型来说，最好使用初始化列表，因为使用初始化列表少调用一次默认构造函数。    
具体说明参加如下链接：    
![](http://www.cnblogs.com/graphics/archive/2010/07/04/1770900.html)      

<br>
#### **第三章：Data语意学**   

##### **数据成员结构**    
###### **类结构大小受那些因素影响**    
1. 语言本身所造成的额外负担（overhead）当语言支持virtual base classes时，就会导致一些额外负担。    
2. 编译器对于特殊情况所提供的优化处理。    
3. Alignment对齐的限制。    

###### **数据成员绑定**    
```
#include <stdio.h>
typedef int length;
int x;
class Point3d
{
	public:
		Point3d():x(2){}
		void numble(length val) {_val = val;}

		typedef float length;
		length _val;
		int x;
};


int main()
{
	Point3d p3d;

	printf("x = %d\n", x);
	printf("p3d.x = %d\n", p3d.x);
	//numble(x); //这里报错类型不匹配
	return 0;
}
``` 

```
输出：
x = 0
p3d.x = 2
```
在一个inline member function躯体之内的一个data member绑定操作，会在整个class声明完成之后才发生，此称为延迟评估。然而，这个对于成员函数function的argument list（不包括构造函数的参数列表）并不为真。Argument list中的名称还是会在他们第一次遭遇时被适当地决议（resolved）完成。因此在extern和nested type names之间的非直觉绑定操作还是会发生。例如上面代码，length的类型在两个member function signature中都决议（resolved）为global typedef，也就是int,当后续再有length的nested typedef声明出现时，C++standard就把稍微早点绑定标识为非法。    

###### **数据成员的存取**    
***静态数据成员***    
1，静态数据成员，是被编译器提出于class之外，存放在程序的global data segment之中，每个静态数据成员只有一个实例。每个成员的存取许可（private、protected、public）,以及class关联，并不会招致任何空间上或执行时间上的额外负担，不论是在个别的class object还是在static data menber本身。     

2， 若取一个static data member的地址，会得到一个指向其数据类型的指针，而不是一个指向其class member的指针，因为静态成员并不内含在一个class object之中，例如：    
```
&Point3d::chunkSize;
```

会获得类型如下的内存地址：    
```
const int *
```  

而不是如下执行Point3d data member的指针类型：    
```
int Point3d::*
```

若上述看不明白可参见如下所示：
```
float Point3d::*px = &Point3d::x
```

***非静态数据成员***    
1，非静态数据成员直接存放在每一个class object之中，除非经由显示或隐式的，否则没有办法直接取得他们。    
 
2，对于非静态成员变量的存取操作，编译器需要把class object的起始地址加上data member的偏移（offset），例如：   
> origin._y = 0.0;
> 那么地址&origin._y将等于：    
> &origin + （&Point3d::_y - 1）
> **注**：offset值总是被加一的，编译器为了区分出“一个指向data member的指针，用于指出class的第一个member”和“一个指向data member的指针，没有指出任何member”两种情况。    

###### **继承与数据成员**    
对于下面例子其成员结构为：
```
class Point2d{
public:
  //constructor(s)
  //opertations
  //access functions
private:
  float x, y;
}

class Point3d{
public:
  //constructor(s)
  //opertations
  //access functions
private:
  float x, y, z;
}
```

***非继承数据布局图***    
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel3.jpg)     

***只要继承不要多态数据布局图***     
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel4.jpg)     

***加上多态数据布局图***        
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel5.jpg)     

***多重继承数据布局图***         
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel6.jpg)     

![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel7.jpg)     

***虚拟继承数据布局图***     
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel9.jpg)     

![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel10.jpg)     

###### **虚拟继承特点**    
Class如果内含一个或多个`virtual base class subobjects`,像`iostream`那样，将被分割为两部分：一个**不变区域**和一个**共享区域**。不变区域中的数据，不管后继如何衍化，总是有固定的offset（从object的开头算起），所以这一部分数据可以直接存取。至于共享区域，所表现的就是`virtual base class subobjects`。这一部分的数据，其位置会因为每次的派生操作而又变化，所以它只可以被间接存取。代码如下所示：    
```
void Point3d::operator+=( const Point3d &rhs)
{
  _x += rhs._x;
  _y += rhs._y;
  _z += rhs._z;
}

Point2d *p2d = pv3d;
```

在cfront策略之下，这个运行符会被内部转换为：
```
//C++伪代码
_vbcPoint2d->_x += rhs._vbcPoint2d->_x;
_vbcPoint2d->_y += rhs._vbcPoint2d->_y;
_z += rhs._z

Point2d *2d = pv3d?_pv3d->_vbcPoint2d:0;
```

##### **常见问题思索**：        
1，以下两种方式进行存取x有什么重大差异吗？    
```
Point3d origin, *pt = &origin;
origin.x = 0.0;
pt->x = 0.0;
```    
答：当Point3d是一个`deriverd class`,而其继承结构中有一个`virtual base class`，并且被存取的member x是一个从该`virtual base class` 继承而来的`member`时，就会有重大的差异。这时候我们不能够说pt必然指向哪一种`class type`（因此，我们也就不知道比那一时期这个member真正的offset位置），所以这个存取操作必须延迟至执行期，经由一个额外的间接引导，才能够解决。如果使用`origin`，就不会有这些问题，其类型无疑是`Point3d class`,而即使它集成自`virtual base class`，`member`的offet位置也在编译时期固定了。    

2，把两个原本不相干的class凑成一对`type/subtype`，并带有继承关系，易犯的错误？
答：    
* 重复设计一些相同操作的函数，例如构造函数和承载函数，有可能直接可以使用基类的。    
* 把一个class分解成两层或更多层，有可能会为了“表现class体系之抽象化”而膨胀所需的空间。膨胀的原因是因为C++语言保证“出现在derive class 中的base class subobject”有其完整原样性。    

3，什么叫自然多态？    
答：单一继承就是提供了一种“自然多态”形式，是关于classes体系中的`base type`和`derived type`之间的转换。对于`base type`和`derived type`的object都是从相同的地址开始，期间差异只在于derived object比较大，用于多容纳它自己的nonstatic data member。     

4，多重继承的复杂度在于？    
答：在于derived class和其上一个base class乃至上上个base class......之间的“非自然关系”。    

5，存取第二个（或后继）base class中的一个data member，会付出额外的成本吗？    
答：不，member的位置在编译时就固定了，因此存取member只是一个简单的offset运算，就像单一继承一样简单，不管是经由一个指针、一个reference或是一个object来存取。    

6，多重继承与虚拟继承的区别？    
![](/images/posts/2018-8-5-InsideTheC++ObjectModel/InsideTheC++ObjectModel8.jpg)     

<br>
#### **第四章：Function语意学**   

C++支持三种类型的member functions:static、nonstatic和virtual，每种类型被调用的方式都不相同。    

##### **成员的各种调用方式**    
###### **非静态成员函数**    
C++的设计准则之一就是：`nonstatic member function`至少必须和一般的nonmember function 有相同的效率，即选择`member function`不应该带来什么额外负担，这是因为编译器内部已将“member函数实例”转换为对等的“nonmember函数实例”。    



##### **常见问题思索**      
1，static member functions不能实现的方式？    
答：    
* 函数中直接存取nonstatic数据。    
* 函数被声明为const。例如`float Point3d::mangintude() const {...}`

2，

<br>
持续更新中......    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [深度探索C++对象模型](https://clodfisher.github.io/2018/08/InsideTheC++ObjectModel/)      

