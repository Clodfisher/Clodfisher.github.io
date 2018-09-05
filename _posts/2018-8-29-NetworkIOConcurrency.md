---
layout: post    
title: Linux网络IO并行化技术总结    
date: 2018-8-29    
tags: linux网络           
---

### 前言    
本文主要是对网络性能需要有所改善的技术，进行简单的总结。    

在对NIC进行调整的时候，发现RSS(receive side scaling)和RPS(Receive Packet Steering)是两个需要关注的点。RSS和RPS都是网卡为了在接受数据包的时候使用多核架构而进行的性能增强，RSS是在硬件层面而RPS在软件层面。在数据包接收到之后在用户态的处理逻辑怎么处理，应用层的响应数据包如何发送都会影响系统性能，RFS(Receive Flow Steering)和XPS(Transmit Packet Steering)这两个机制就是为了解决这两个问题而产生的。    

<br>
### 基本概念    

#### DMA(Direct Memory Access)     
直接内存访问（Direct Memory Access，DMA）是计算机科学中的一种内存访问技术。它允许某些计算机内部的硬件子系统（计算机外设），可以独立地直接读写系统内存，而不需中央处理器（CPU）介入处理 。在同等程度的处理器负担下，DMA是一种快速的数据传送方式。很多硬件的系统会使用DMA，包含硬盘控制器、绘图显卡、网卡和声卡。    

#### RAM(Random Access Memory)    
随机存取存储器（英语：Random Access Memory，缩写：RAM），也叫主存，是与CPU直接交换数据的内部存储器。它可以随时读写（刷新时除外，见下文），而且速度很快，通常作为操作系统或其他正在运行中的程序的临时数据存储介质。主存（Main memory）即计算机内部最主要的存储器，用来加载各式各样的程序与数据以供CPU直接运行与运用。由于DRAM的性价比很高，且扩展性也不错，是现今一般计算机主存的最主要部分。2014年生产计算机所用的主存主要是DDR3 SDRAM，而2016年开始DDR4 SDRAM逐渐普及化，笔电厂商如华硕及宏碁开始在笔电以DDR4存储器取代DDR3L。     

#### RSS(Receive-Side Scaling)    
RSS为网卡数据传输使用多核提供了支持，RSS在硬件/驱动级别实现多队列并且使用一个hash函数对数据包进行多队列分离处理，这个hash根据源IP、目的IP、源端口和目的端口进行数据包分布选择，这样同一数据流的数据包会被放置到同一个队列进行处理并且能一定程度上保证数据处理的均衡性。    

#### RPS(Receive Packet Steering)    
RPS是和RSS类似的一个技术，区别在于RSS是网的硬件实现而RPS是内核软件实现。RPS帮助单队列网卡将其产生的SoftIRQ分派到多个CPU内核进行处理。在这个方案中，为网卡单队列分配的CPU只处理所有硬件中断，由于硬件中断的快速高效，即使在同一个CPU进行处理，影响也是有限的，而耗时的软中断处理会被分派到不同CPU进行处理，可以有效的避免处理瓶颈。         

#### RFS(Receive Flow Steering)    
在使用RPS接收数据包之后，会在指定的CPU进行软中断处理，之后就会在用户态进行处理；如果用户态处理的CPU不在软中断处理的CPU，则会造成CPU cache miss，造成很大的性能影响。RFS能够保证处理软中断和处理应用程序是同一个CPU，这样会保证local cache hit，提升处理效率。RFS需要和RPS一起配合使用。         

####  XPS(Transmit Packet Steering)    
XPS通过创建CPU到网卡发送队列的对应关系，来保证处理发送软中断请求的CPU和向外发送数据包的CPU是同一个CPU，用来保证发送数据包时候的局部性。xps主要是针对多队列的网卡发送时的优化，当发送一个数据包的时候，它会根据CPU来选择对应的队列。        

<br>
参考链接：    
[开启网卡多队列功能](https://support.huaweicloud.com/usermanual-ecs/zh-cn_topic_0058758453.html)           

<br>
参考链接：    
[容器云负载均衡之三：RSS、RPS、RFS和XPS调整](https://blog.csdn.net/cloudvtech/article/details/80182074)            