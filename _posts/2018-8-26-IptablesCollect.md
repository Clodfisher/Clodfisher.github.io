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
    |    name    | age |   
    | ---------- | --- |   
    | LearnShare |  12 |   
    | Mike       |  32 |  
       

### 每章概述和感悟    

<br>
#### **第一章：Java程序设计概述**    
  