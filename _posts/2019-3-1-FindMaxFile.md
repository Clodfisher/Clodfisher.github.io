---
layout: post    
title: 找到占用内存大文件    
date: 2019-02-26    
tags: 实战运维              
---
<br>
### 现场问题描述            
在远程支持现场问题的时候，发现linux操作系统的内存占用过大，而且目录多深度广（可以认为是整个操作系统），不太好找到具体哪个文件占用过大内存，导致若是拷贝一些文件或生产文件的时候导致失败。       

<br>
### 达到目的    
在整个操作系统中，找到过大的且无用的文件，删除。     

<br>
### 解决方法    
1. 通过`df`命令找到系统中那些目录占用较大内存。 可以看出`/`目录占用100%。       
![](/images/posts/2019-3-1-FindMaxFile/FinMaxFile0.jpg)         

2. 进入`/`目录，通过`du`命令一级一级的查找，其中命令-d1是查找的深度。    
![](/images/posts/2019-3-1-FindMaxFile/FinMaxFile1.jpg)         

3. 进入第2步中找到的目录，再接着查找定位。        
![](/images/posts/2019-3-1-FindMaxFile/FinMaxFile2.jpg)         

<br>
### 总结    
对于目录包含内容的大小要用`du`命令，而不是`ls -alh`,`ls`只能够查看文件大小（文本文件或目录文件）。    
![](/images/posts/2019-3-1-FindMaxFile/FinMaxFile3.jpg)         




<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [找到占用内存大文件](https://clodfisher.github.io/2019/03/FindMaxFile/)           
