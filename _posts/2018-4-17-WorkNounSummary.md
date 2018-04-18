---
layout: post
title: 工作中遇到的名词总结
date: 2018-4-17 
tags: 知识汇总        
---

<br>
### PUB/SUB模型    

Pub/Sub模型定义了如何向一个内容节点发布和订阅消息，这些节点被称为主题（topic）。主题可以被称为是消息的传输中介，发布者（publisher）发布消息到主题，订阅这（subscriber）从主题订阅消息。主题是的消息订阅者和消息发布者保持相互独立，不需要接触即可保证消息的传送。    

主要使用场景：按服务种类的微服务中，跨板块间需要交互。此时需要一个类似于Pub/Sub 的高可用的并可以持久化的消息订阅通知的中间件来打通各个版块的数据和信息交换。    

<br>
### Twitter-Snowflake    
在全局唯一 ID 的算法中，可以采用UUID方式，但它的字符串占用的空间比较大，索引的效率比较低，生成的ID过于随机，完全不是人读的，而且没有随机递增，如果要安前后顺序排序的话，基本不可能。    

Twitter 的开源项目 Snowflake。Twitter-Snowflake算法产生的背景相当简单，为了满足Twitter每秒上万条消息的请求，每条消息都必须分配一条唯一的id，这些id还需要一些大致的顺序（方便客户端排序），并且在分布式系统中不同机器产生的id必须不同。    

它是一个分布式 ID 的生成算法。其核心思想是，产生一个 long 型的 ID，其中：    
* 41bits 作为毫秒数。大概可以用 69.7 年。
* 10bits 作为机器编号（5bits 是数据中心，5bits 的机器 ID），支持 1024 个实例。
* 12bits 作为毫秒内的序列号。一毫秒可以生成 4096 个序号。
![](/images/posts/2018-4-17-WorkNounSummary/WorkNounSummary0.jpg)          




<br>
持续更新中......    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [工作中遇到的名词总结](https://clodfisher.github.io/2018/04/WorkNounSummary/)      
