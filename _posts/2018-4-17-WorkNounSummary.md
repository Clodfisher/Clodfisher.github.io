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
持续更新中......    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [工作中遇到的名词总结](https://clodfisher.github.io/2018/04/WorkNounSummary/)      
