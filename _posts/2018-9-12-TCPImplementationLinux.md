---
layout: post    
title: linux内核协议栈架构简述    
date: 2018-9-12    
tags: linux内核协议栈           
---

<br>
### 前言    
本文档简要概述了如何在Linux内核协议栈中实现TCP传输过程。对于其中涉及到的关键点，以及整个网络传输的性能影响参数进行了总结说明。对于设计到的细节描述不太全面，有兴趣者可以根据其中涉及到的点进行由线到面的自行扩充，本文档主要是根据自己理解进行整理，若是有错误之处，希望不吝赐教。        

<br>
### 

<br>
参考链接：       
[Linux kernel design patterns - part 1](https://lwn.net/Articles/336224/)    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [linux内核协议栈架构简述](https://clodfisher.github.io/2018/09/TCPImplementationLinux/)            