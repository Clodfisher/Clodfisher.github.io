---
layout: post
title: go中强制类型转换的特点
date: 2018-3-24 
tags: Go        
---

<br>
### 问题提出    
对于string类型内部是通过[]byte切片进行存储的，而本身string类型是静态类型，若通过下标进行修改，将会报错，若何进行下标修改呢？若是将string类型强制转换为[]byte切片，用转换类型进行拷贝赋值，这是是会想真正的切片一样进行引用拷贝，还是和string类型一样进行内容拷贝呢？    

<br>
### 进行验证    
对于第一个问题可以将string类型强制为[]byte切片后再进行下标修改，对于第二个问题猜测是进行内容拷贝，强制不能改变本身类型的属性。    

直接上代码：    
```
package main

import (
	"fmt"
)

func main() {
	fmt.Println("---------验证slice的赋值等于为引用----------------")
	nSliceOld := []int{1, 2, 3, 4, 5, 6, 7}
	fmt.Printf("slice old = %v, addr=%p\n", nSliceOld, &nSliceOld)

	nSliceNew := nSliceOld
	fmt.Println("copy change old string ...")
	nSliceNew[0] = 0
	fmt.Printf("slice old = %v, addr=%p\n", nSliceOld, &nSliceOld)
	fmt.Printf("slice new = %v, addr=%p\n", nSliceNew, &nSliceNew)

	fmt.Println("----------验证string进行slice强转后，赋值等于不是引用---------------")

	strNameOld := "ClodFisher"
	strSliencOld := []byte(strNameOld)
	fmt.Printf("string old = %s, string old addr = %p\n", strNameOld, &strNameOld)
	fmt.Printf("string old slice = %v, string slice old addr = %p\n", strSliencOld, &strSliencOld)

	strSliencOld[0] = 'S'
	strNameNew := string(strSliencOld)
	fmt.Println("copy change slice ...")
	fmt.Printf("string old = %s, string old addr = %p\n", strNameOld, &strNameOld)
	fmt.Printf("string new = %s, string new addr = %p\n", strNameNew, &strNameNew)
	fmt.Printf("string new slice = %v, sting slice new addr = %p\n", strSliencOld, &strSliencOld)
}
```

结果        
```
[ `go run main.go` | done: 724.1389ms ]
	---------验证slice的赋值等于为引用----------------
	slice old = [1 2 3 4 5 6 7], addr=0xc042002400
	copy change old string ...
	slice old = [0 2 3 4 5 6 7], addr=0xc042002400
	slice new = [0 2 3 4 5 6 7], addr=0xc042002440
	----------验证string进行slice强转后，赋值等于不是引用---------------
	string old = ClodFisher, string old addr = 0xc0420361c0
	string old slice = [67 108 111 100 70 105 115 104 101 114], string slice old addr = 0xc0420024a0
	copy change slice ...
	string old = ClodFisher, string old addr = 0xc0420361c0
	string new = SlodFisher, string new addr = 0xc0420361e0
	string new slice = [83 108 111 100 70 105 115 104 101 114], sting slice new addr = 0xc0420024a0
```

<br>
### 结论        
将string类型的数据，强制称切片，在进行赋值操作，这时候，会新建立一块内存，通过打印的地址可以看出地址不同。而真正的切片，赋值操作，是拷贝的引用，地址一样。强制转换，不会改变原来值得属性，只是使用而已，这一点和C++的去常很相似。   
**切片类型、字典类型、通道类型都属于应用类型。**    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [go中强制类型转换的特点](https://clodfisher.github.io/2018/03/GoForceTrait/)   