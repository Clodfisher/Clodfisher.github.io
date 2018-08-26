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
其各个字段含义如下所示：     
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect3.jpg)       
 
优先级顺序是：raw ---> mangle --->  nat ---> filter。也就是说在某一个链上有多张表，数据包都会依次按照hook点的方向进行传输，每个hook点上Netfilter又按照优先级挂了很多hook函数（即表），就是按照这个顺序依次处理。无论那一个Filter表其匹配原则都是“First Match”，即优先执行，第一条规则逐一向下匹配，如果封包进来遇到第一条规则允许通过，那么这个封包就通过，而不管下面的rule2、rule3的规则是什么都不重要；相反如果第一条规则说要丢弃，即便是rule2规则允许通过也不起任何作用，这就是“first match”原则。       
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect5.jpg)       
   
###  Iptables基本操作    
启动 `iptables` ： `service iptables start`    

关闭 `iptables` ： `service iptables stop`   

重启 `iptables` ： `service iptables restart`    

查看 `iptables` 状态： `service iptables status`    

保存 `iptables` 配置： `service iptables save`    

`Iptables` 服务配置文件： `/etc/sysconfig/iptables-config`    

`Iptables` 规则保存文件： `/etc/sysconfig/iptables`    

打开 `iptables` 转发： `echo "1"> /proc/sys/net/ipv4/ip_forward`   

### Iptables语法    
通过向防火墙提供有关对来自某个源地址、到某个目的地或具有特定协议类型的信息包要做些什么的指令，规则控制信息包的过滤。通过使用iptables系统提供的特殊命令iptables建立这些规则，并将其添加到内核空间特定信息包过滤表内的链中。关于添加、去除、编辑规则的命令，一般语法如下：    
> `Iptables [-t table] command [chain] [rules] [-j target]`    

#### table    
表名： `Filter, NAT, Mangle, Raw`
起包过滤功能的为表 Filter ，可以不填，不填默认为 Filter。   
各表实现的功能如下所示：        

| 表名字 | 实现功能      | 具有的链  |
| ------|:-------------:| -----:|
| Filter| 是Netfilter内最重要的机制，其任务为执行包的过滤工作，也是防火墙的主要功能 |  INPUT,FORWARD,OUTPUT |
| Nat   | 是防火墙上的IP分享，转发器，可以针对数据进行IP和端口级别的转发。（目前我们一些实验室、小型公司上网基本都是通过Nat技术实现，因为共有IP十分稀有，还有它对于数据包的转发、隐藏IP，等都用到SNAT、DNAT技术      |   PREROUTING,POSTROUTING,OUTPUT |
| Mangle| 经由它可以修改行经防火墙的数据包内容      |    PREROUTING,INPUT,FORWARD,OUTPUT,POSTROUTING |
| Raw   | 负责加快包穿越防火墙的速度，由此提高防火墙的性能      |   PREROUTING,OUTPUT |

#### command    
command部分是iptables命令最重要的部分。它告诉iptables命令要做什么，例如插入规则、将规则添加到链的末尾或删除规则。    

看命了规则之前先看命了行语法选项的约定：  
  
| 元素         | 描述     |
| ------------- |--------------:|
| &#124; |竖条或管道符号将替换的语法选项分隔开来。例如，许多iptables命令同时有缩写和完全形式，例如-L和--list,它们将作为可替换的选项被呈现，因为你仅可以使用它们中的一个-L或--list |
| \<value> |尖括号表示一个用户提供值，例如一个字符串或数值 |
| [] | 方括号指明了它包含的命令、选项或值是可选的。例如，大多数匹配符号可以使用否定符号！，这将匹配任何除匹配值之外的值。是否符合通常放在匹配符号和用于匹配的值之间 |
| \<value>:<value> | 冒号指明了某个值得范围。这两个定义了范围内的最大值和最小值。由于范围本身可选的，因此常见的形式为\<value>[:\<value>] |


命令格式如下所示：   

| 选项名         | 功能及特点     |
| ------------- |--------------:|
| -A &#124; --append \<chain>\<rule specification> |在指定链的末尾添加（ –append ）一条新的规则 |
| -D &#124; --delete \<chain>\<rule number> &#124;\<rule specification>| 删除（ –delete ）指定链中的某一条规则，按规则序号或内容确定要删除的规则      |
| -I &#124; --insert\<chain>\[\<rule number>]\<rule specification> | 在指定链中插入（ –insert ）一条新的规则，默认在链的开头插入      |
| -R &#124;--replace\<chain>\[\<rule number>]\<rule specification> | 修改、替换（ –replace ）指定链中的一条规则，按规则序号或内容确定      |
| -C &#124; --check\<chain>\<rule specification> | 检查规则链中是否有某条规则匹配\<rule specification>      |
| -L &#124; --list[\<chain>] | 列出（ –list ）指定链中的所有的规则进行查看，默认列出表中所有链的内容      |
| -F &#124; --flush[\<chain>] | 清空（ –flush ）指定链中的所有规则，默认清空表中所有链的内容      |
| -N &#124; --new-chain[\<chain>] | 新建（ –new-chain ）一条用户自己定义的规则链      |
| -X &#124; --delete-chain[\<chain>] | 删除指定表中用户自定义的规则链（ –delete-chain ）      |
| -P &#124; --policy\<chain>\<policy> | 为内建规则链表设置默认规则，策略可以为接受（ACCEPT）或（拒绝）      |
| -n &#124; --numeric | 用数字形式（ –numeric ）显示输出结果，若显示主机的 IP 地址而不是主机名      |
| -v &#124; --verbose | 列出每条规则的额外信息，例如字节和数据包计数、规则选项以及相关的网络接口      |
| -h &#124; \<some command> -h | 列出iptables的命令和选项，如果-h后面跟着某个iptables命令，则列出此命令的语法和选项      |
| --line-numbers | 查看规则列表时，同时显示规则在链中的顺序号      |

#### chain    
可以根据数据流向来确定具体使用哪个链，在 Filter 中的使用情况如下：
> INPUT链 – 处理来自外部的数据。     
> OUTPUT链 – 处理向外发送的数据。    
> FORWARD链 – 将数据转发到本机的其他网卡设备上。    

#### rules     
条件匹配分为基本匹配和扩展匹配，拓展匹配又分为隐式扩展和显示扩展。    
a) 基本匹配包括：    

| 匹配参数        | 说明     |
| ------------- |--------------:|
| -p |指定规则协议，如 tcp, udp,icmp 等，可以使用 all 来指定所有协议 |
| -s |指定数据包的源地址参数，可以使 IP 地址、网络地址、主机名 |
| -d |指定目的地址 |
| -i |输入接口 |
| -o |输出接口 |

b) 隐式扩展包括：    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect6.jpg)       

c) 常用显式扩展：    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect7.jpg)       
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect8.jpg)           

#### target     
数据包控制方式包括四种为：    
> ACCEPT：允许数据包通过     
> DROP：直接丢弃数据包，不给出任何回应信息     
> REJECT：拒绝数据包通过，必须时会给数据发送端一个响应信息     
> LOG：在/var/log/messages 文件中记录日志信息，然后将数据包传递给下一条规则     
> QUEUE：防火墙将数据包移交到用户空间     
> RETURN：防火墙停止执行当前链中的后续Rules，并返回到调用链(the calling chain)     

### Iptables 常见命令    
   

<br>
参考链接：    
[Netfilter/Iptables入门](http://blog.51cto.com/chenguang/1601462)        
[Linux 防火墙在内核中的实现](https://www.ibm.com/developerworks/cn/linux/network/l-netip/index.html)        
[关于Linux防火墙'iptables'的面试问答](https://linux.cn/article-5948-1.html)      
[Linux管理员应该了解的20条IPTables防火墙规则用法](https://www.sysgeek.cn/iptables-firewall-rules-examples/)      
[Linux下Netfilter/IPTables防火墙案例分析](http://blog.51cto.com/me2xp/1545899)      
[看了那么多iptables的教程，这篇教程还是比较全面易懂的](https://www.91yun.co/archives/1690)    
[使用netfilter/iptables构建防火墙](http://lavafree.iteye.com/blog/795916)          
[科普一下Linux防火墙Netfilter/iptables](http://lib.csdn.net/article/linux/28213)      
[Netfilter 概述及其hook点](https://blog.csdn.net/liukun321/article/details/54577433)      
[玩转高性能超猛防火墙nf-HiPAC](https://blog.csdn.net/dog250/article/details/41289217?locationNum=2&fps=1)      
[25个常用的iptables命令](http://blog.51cto.com/nashsun/1847526)      
[搭建基于netfilter/iptables的防火墙实验环境](http://tech.sina.com.cn/roll/2008-12-18/1719922251.shtml)      





       
<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Iptables汇总 ](https://clodfisher.github.io/2018/08/IptablesCollect/)      


  