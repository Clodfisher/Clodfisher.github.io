---
layout: post
title: Python列表的可变性讨论
date: 2018-2-2 
tags: python    
---
<br>    
 在Python中，一切皆对象，对于对象中的数字，字符串，元组，其是不可变的对象。而对于列表是可变的对象。可变与不可变性，是对内存地址而言的。由于Python是解释性语言，因而对于每个对象的赋值都是对内存块的引用，例如 a=2,此时a是2的引用，若这是你修改a=2的表达式，改为a=3,此时a就指向了3这个内存块。    

具体程序如下所示：    
```
a = 2
print id(a)
a = 3
print id(a)
```
输出结果：    
```
31095184
31095160
```
以上说明，a赋值不同的变量，只是引用不同的内存，不能对内存进行修改。这体现了不可变性。    
对于元组而言，也是属于不可变性，一个元组保存在同一块内存中，属于不可变。    
具体代码如下所示：    
```
print('Research on the variability of tuple ')  
tuple1 = ("juan", 24, "110")  
print tuple1  
  
tuple1[1] = 25  
print tuple1  
```
输出结果：    
```
TypeError: 'tuple' object does not support item assignment
```
以上说明不能对tuple1中的某个元素进行修改。    
对于列表而言，其属于是可变性。具体理解可以根据C语言中的链表类比，对于列表，首地址是不可变的，而对于列表内的所有元素进行修改，会改变单个元素的地址（指向不同的引用）。所以说对于列表中的单个元素而言是不可变的，对于整体列表而言是可变的，如下图所示：    

![](/images/posts/2018-2-2-PythonListChange/PythonListChange.jpg)    

列表操作代码如下所示：    
```
print('Research on the variability of lists')   
list1 = ["juan", 24, "110"]  
print list1  
print "ID-list1=", id(list1)  
print "ID-list1[0]=", id(list1[0])  
print "ID-list1[1]=", id(list1[1])  
  
  
list1[0] = "tao"  
list1[1] = 26  
print list1  
print "ID-list1=", id(list1)  
print "ID-list1[0]=", id(list1[0])  
print "ID-list1[1]=", id(list1[1]) 
```
输出结果：    
```
Research on the variability of lists  
['juan', 24, '110']  
ID-list1= 38301128  
ID-list1[0]= 38951672  
ID-list1[1]= 31094656  
['tao', 26, '110']  
ID-list1= 38301128  
ID-list1[0]= 38951872  
ID-list1[1]= 31094608 
```
**以上只是对这种现象的一种假想，没能进行源码的验证，若有错误，希望给予指正。**

<br>
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Python列表的可变性讨论](https://clodfisher.github.io/2018/02/PythonListChange/)   