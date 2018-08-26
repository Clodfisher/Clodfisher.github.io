---
layout: post    
title: Iptables汇总    
date: 2018-8-26    
tags: 网络安全           
---

### 前言    
本文主要是对于网上或书上关于iptables感觉不错的文章进行汇总，以供自己参考、学习、提升为理论知识，从而更好的了解Linux内核中对于netfilter实现过程。    

### Netfilter概述             
Netfilter/IPTables是Linux2.4.x之后新一代的Linux防火墙机制，是linux内核的一个子系统。Netfilter采用模块化设计，具有良好的可扩充性。其重要工具模块IPTables从用户态的iptables连接到内核态的Netfilter的架构中，Netfilter与IP协议栈是无缝契合的，并允许使用者对数据报进行过滤、地址转换、处理等操作。Netfilter支持通过以下方式对数据包进行分类：    
* 源IP地址    
* 目标IP地址    
* 使用接口    
* 使用协议（TCP、UDP、ICMP等）    
* 端口号    
* 连接状态（new、ESTABLISHED、RELATED、INVALID)     
    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect0.jpg)       

### Netfilter/IPTables的框架    
其主要通过表、链实现规则，可以这么说，Netfilter是表的容器，表是链的容器，链是规则的容器，最终形成对数据报处理规则的实现。简单地讲， tables 由 chains 组成，而 chains 又由 rules 组成。 iptables 默认有四个表 Filter, NAT, Mangle, Raw ，其对于的链如下图：    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect1.jpg)       

IPTables是用户空间的工具，它提供了4张表，分别是：    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect2.jpg)       

报文按照来源和去向可以分为三类：流入的、流出的、流经的，其中流入、流经要经过路由才能区分，流出和流经也要经过路由转发。那么Netfilter就在这些必经之路上提供了5个钩子位置，分别是：    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect4.jpg)       
 
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect3.jpg)       
 
优先级顺序是：raw ---> mangle --->  nat ---> filter。也就是说在某一个链上有多张表，数据包都会依次按照hook点的方向进行传输，每个hook点上Netfilter又按照优先级挂了很多hook函数（即表），就是按照这个顺序依次处理。无论那一个Filter表其匹配原则都是“First Match”，即优先执行，第一条规则逐一向下匹配，如果封包进来遇到第一条规则允许通过，那么这个封包就通过，而不管下面的rule2、rule3的规则是什么都不重要；相反如果第一条规则说要丢弃，即便是rule2规则允许通过也不起任何作用，这就是“first match”原则。       
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect5.jpg)       
   
       
<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Iptables汇总 ](https://clodfisher.github.io/2018/08/IptablesCollect/)      


  