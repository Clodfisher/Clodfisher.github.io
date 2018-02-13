---
layout: post
title: Python Currying
date: 2018-2-13 
tags: python        
---
<br>
### 名词解释    

Currying:因为是美国数理逻辑学家哈斯凯尔 加里（Haskell Curry）发明了这种函数使用技巧，所以这样的用法就以他的名字命名为Currying，中文翻译为“加里化”。    

<br> 
### 加里化（Currying）与偏函数（Partical）区别    

偏函数是将所要承载的函数作为partical()函数的第一个参数，原函数的各个参数一次作为partical()函数的后续参数，除非使用关键字参数。    
**偏函数解决的问题**：函数有多个参数时，希望固定其中某个参数的值。    
**加里化**：将一个函数的多个参数分解成多个函数，然后将函数多层封装起来，每层函数都返回一个函数取接收下一个参数，简化函数的多个参数。    

**偏函数**的例子如下：    
* 在C语言中的例子如下：    

```
int foo(int a, int b, int c){
	return a + b + c;
}

int foo23(inta+intc){
	return foo(a, 23, c);
}
```
* 在Python语言中的例子如下：    

```
from functlools import partial

def foo(a, b, c):
	return a + b +c

foo23 = partial(foo, b = 23)

foo23(a = 1, c = 3) #=>27
```
**加里化**的例子如下：
* 在Python语言中的例子如下：   

```
def inc(x):
	def incx(y):
		return x + y
	return incx

inc2 = inc(2)
inc5 = inc(5)

print inc2(5) 	# => 7
print inc5(5) 	# => 10
print inc(6)(5) # => 11
```
函数加里化提供了一种非常自然的方式来实现某些偏函数应用。如上所示，将第一个参数被固定为2，只需要inc2 = inc(2)即可。    
通过上面例子Inc()函数返回了另一个函数incx(),预示可以用inc()函数来构造各种版本的inc函数，比如：inc2()和inc5()。    
其完全符合了函数编程的理念：    
> 把函数当成变量使用，关注表述问题而不是怎样实现，这样可以让代码更易读。    
> 因为函数返回里面这个函数，所以函数关注的是表达式，关注的是描述这个问题，而不是怎样实现这个问题。   

<br> 
### 函数加里化和偏函数应用的总结

* 偏函数应用是找一个函数，固定其中的几个参数值，从而得到一个新的函数。   
* 函数加里化是一种使用匿名单参数函数来实现多参数函数的方法。    
* 函数加里化能够让你轻松的实现某些偏函数应用。   

<br>
参考链接：    
[Functions as Returned Values andCurrying in Python](http://www.idc-online.com/technical_references/pdfs/information_technology/Functions_as_Returned_Values_and_Currying_in_Python.pdf)    
[函数加里化(Currying)和偏函数应用(Partial Application)的比较](http://www.vaikan.com/currying-partial-application/)       
[Python函数式编程——偏函数](http://www.pythoner.com/54.html)       


<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Python Currying](https://clodfisher.github.io/2018/02/PythonCurryingFunction/)   