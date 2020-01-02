---
layout: post    
title: Iptables汇总    
date: 2018-8-26    
tags: 网络安全 iptables           
---

<br>
### 前言    
本文主要是对于网上或书上关于iptables感觉不错的文章进行汇总，以供自己参考、学习、提升为理论知识，从而更好的了解Linux内核中对于netfilter实现过程。在防火墙关闭状态下，不要通过iptables指令（比如 iptables -nL）来查看当前状态！因为这样会导致防火墙被启动，而且规则为空。虽然不会有任何拦截效果，但所有连接状态都会被记录，浪费资源且影响性能并可能导致防火墙主动丢包！若是要加载`/etc/sysconfig/iptables`中规则，需要重启防火墙`systemctl restart iptables.service`，或是开机启动`systemctl enable iptables.service`。            

<br>
### Netfilter概述             
Netfilter/IPTables是Linux2.4.x之后新一代的Linux防火墙机制，是linux内核的一个子系统。Netfilter采用模块化设计，具有良好的可扩充性。其重要工具模块IPTables从用户态的iptables连接到内核态的Netfilter的架构中，Netfilter与IP协议栈是无缝契合的，并允许使用者对数据报进行过滤、地址转换、处理等操作。Netfilter支持通过以下方式对数据包进行分类：    
* 源IP地址    
* 目标IP地址    
* 使用接口    
* 使用协议（TCP、UDP、ICMP等）    
* 端口号    
* 连接状态（new、ESTABLISHED、RELATED、INVALID)     
    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect0.jpg)       

<br>
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
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect5.png)       
    
上图说明了四种表，作用的内置链，其中主要的表有三个，表raw有着特殊的用途，三个主要的表作用的内置链分别如下：    

* filter - filter表示默认表。它包含了实际防火墙的过滤规则。其内建的规则链包括：        
> * INPUT;    
> * OUTPUT;    
> * FORWARD。    

* nat - nat表包含了源地址和目的地址转换以及端口转换的规则。这些规则在功能上与防火墙filter规则不同。内建的规则包括：    
> * PREROUTING - DNAT/REDIRECT;    
> * OUTPUT - DNAT/REDIRECT;    
> * POSTROUTING - SNAT/MASQUERADE。    

* mangle - mangle表包含了设置特殊数据包路由标志的规则。这些规则接下来将在filter表中进行检查。其内建的规则链包括：    
> * PREROUTING - 被路由的数据包；    
> * INPUT - 到达防火墙并通过PREROUTING规则链的数据包；    
> * FORWARD - 修改通过防火墙路由的数据包；    
> * POSTROUTING - 在数据包通过OUTPUT规则链之后，但在离开防火墙之前修改数据包；    
> * OUTPUT - 本地生成的数据包。    

换一种角度，从链的HOOK点来看表,不包括raw表:    
**NF_IP_PRE_ROUTING** 在这个HOOK上主要是对数据报作报头检测处理，以捕获异常情况。涉及功能（优先级顺序）：    
> Conntrack(-200)、mangle(-150)、DNAT(-100)    

**NF_IP_LOCAL_IN** 在这个HOOK上主要是对数据报作报头检测处理，以捕获异常情况。涉及功能（优先级顺序）：    
> mangle(-150)、filter(0)、SNAT(100)、Conntrack(INT_MAX-1)    

**NF_IP_FORWARD** 在这个HOOK上主要是对数据报作报头检测处理，以捕获异常情况。涉及功能（优先级顺序）：    
> mangle(-150)、filter(0)    

**NF_IP_LOCAL_OUT** 在这个HOOK上主要是对数据报作报头检测处理，以捕获异常情况。涉及功能（优先级顺序）：    
> Conntrack(-200)、mangle(-150)、DNAT(-100)、filter(0)     

**NF_IP_POST_ROUTING** 在这个HOOK上主要是对数据报作报头检测处理，以捕获异常情况。涉及功能（优先级顺序）：    
> mangle(-150)、SNAT(100)、Conntrack(INT_MAX)     

<br>   
###  Iptables基本操作       

#### 配置防火墙        
```
#先检查是否安装了iptables
service iptables status
#安装iptables
yum install -y iptables
#升级iptables
yum update iptables
#安装iptables-services
yum install iptables-services

#禁用/停止自带的firewalld服务
#停止firewalld服务
systemctl stop firewalld
#禁用firewalld服务
systemctl mask firewalld
```
 
#### service命令    

启动 `iptables` ： `service iptables start`    

关闭 `iptables` ： `service iptables stop`   

重启 `iptables` ： `service iptables restart`    

查看 `iptables` 状态： `service iptables status`    

保存 `iptables` 配置： `service iptables save`    

#### systemctl命令    

`systemctl start iptables.service` #启动防火墙    

`systemctl stop iptables.service` #停止防火墙    

`systemctl restart iptables.service` #重启防火墙，才能使规则生效        

`systemctl enable iptables.service` #设置防火墙开机启动    

`systemctl disable iptables.service` #防火墙开机禁用     

`systemctl status iptables.service` # 查看状态    

#### 其它    

`Iptables` 服务配置文件： `/etc/sysconfig/iptables-config`    

`Iptables` 规则保存文件： `/etc/sysconfig/iptables`    

对配置的规则进行存档`service iptables save`    

保存当前iptables规则到配置文件 `iptables-save > /etc/sysconfig/iptables `    

从配置文件，恢复iptables规则`iptables-restore < /etc/sysconfig/iptables`    

打开 `iptables` 转发： `echo "1"> /proc/sys/net/ipv4/ip_forward`   

查看当前系统中生效的所有参数`/usr/sbin/sysctl –a`     

<br>
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
| \<value>:\<value> | 冒号指明了某个值得范围。这两个定义了范围内的最大值和最小值。由于范围本身可选的，因此常见的形式为\<value>[:\<value>] |
| 在大括号({})之间；将选项用管线(&#124;)隔开。示例：{on&#124;off} | 用户必须从中只选择一个选项的选项集 |


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
| -p &#124; --protocol[!]\[\<protocol> |指定规则协议，如 tcp, udp,icmp 等，可以使用 all 来指定所有协议 |    
| -s &#124; --source &#124; --src[!]\<address>[\<\/mask> |指定数据包的源地址参数，可以使 IP 地址、网络地址、主机名 |   
| -d &#124; --destination &#124; --dst[!]\<address>[\<\/mask> |指定目的地址 |    
| -i &#124; --in-interface[!]\[\<interface>] |输入接口 |   
| -o &#124; --in-interface[!]\[\<interface>] |输出接口 |    
| -j &#124; --jump\<target> |如果数据包匹配规则，则设置此数据包的处置策略。默认的目标包括内建的策略、扩展策略，或用户自定义规则链 |

b) 隐式扩展包括：    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect6.jpg)       

c) 常用显式扩展：    
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect7.jpg)       
![](/images/posts/2018-8-26-IptablesCollect/IptablesCollect8.jpg)           

#### target     
数据包控制方式包括四种为：    
> ACCEPT：允许数据包通过     
> DROP：直接丢弃数据包，不给出任何回应信息       
> REJECT: 拒绝，使用--reject-with选项可以提示信息，有以下可用值
>  * icmp-net-unreachable
>  * icmp-host-unreachable
>  * icmp-port-unreachable
>  * icmp-proto-unreachable
>  * icmp-net-prohibited
>  * icmp-host-prohibited  
>  * icmp-admin-pro-hibited
>  * 未做设置设置的话默认是icmp-port-unreachable     

> LOG：在/var/log/messages 文件中记录日志信息，然后将数据包传递给下一条规则   
> * --log-level   记录日志级别，有debug，info，notice，warning，error，      crit，alert，emerg 
> * --log-prefix   给记录日志加上标签，最多29个字符  


> QUEUE：防火墙将数据包移交到用户空间     
> RETURN：防火墙停止执行当前链中的后续Rules，并返回到调用链(the calling chain)    
> REDIRECT：端口重定向
>  * --to-ports
 
> MARK：做防火墙标记
> DNAT：目标地址转换
> * --to-destination  

> SNAT：源地址转换
> * --to-source

> MASQUERADE：地址伪装
> 自定义链：由自定义链上的规则进行匹配检查

<br>
### Iptables 常见命令    

#### 流量统计    
##### 对特定IP进行流量统计    
> 统计服务器上的IP：192.168.0.10的入网流量：    
> `iptables -I INPUT -d 192.168.0.10`    
> 统计该IP的出网流量：    
> `iptables -I OUTPUT -s 192.168.0.10`    

##### 查看流量    
> `iptables -n -v -L -t filter` 默认是使用易读的单位，也就是自动转化成M，G。如过需要Bytes做单位，则增加一个-x参数    
`iptables -n -v -L -t filter -x`    

##### 显示每条规则的序列号和额外信息    
> `iptables -L -n -v --line-number`    

##### 在几号规则前插入一条所有都能通过规则    
> `iptables -I FORWARD 1 -j ACCEPT`    

<br>
### iptables显式扩展
#### 查看iptables支持哪些显示扩展    
在使用该扩展之前，必须使用-m或–match选项指定模块。有很多种显式匹配扩展模块可用，可以用如下命令查看可用的扩展模块    
`[root@localhost ~]# rpm -ql iptables | grep "[[:lower:]]\+\.so$"1`    
扩展模块的使用方法可以man iptables（CentOS 6）或man iptables-extensions（CentOS 7）查找对应内容，下面介绍几个常用显式扩展模块      
官网链接：    
[iptables-extensions.man.html](http://ipset.netfilter.org/iptables-extensions.man.html)    

#### 查看某个扩展模块命令输入    
```
iptables -m connlimit --help
```

#### 用到的显式扩展模块    
##### connlimit扩展     
允许您限制每个客户端IP地址（或客户端地址块）与服务器的并行连接数。    
```
--connlimit-upto n
    如果现有连接数低于或等于n，则匹配。
--connlimit-above n
    如果现有连接数高于n，则匹配。
--connlimit-mask prefix_length
    使用前缀长度对主机进行分组。对于IPv4，它必须是介于（包括）0和32之间的数字。对于IPv6，
    介于0和128之间。如果未指定，则使用适用协议的最大前缀长度。
--connlimit-saddr
    将限制应用于源组。如果未指定--connlimit-daddr，则这是默认值。
--connlimit-daddr
    将限制应用于目标组。

Examples:
# 每个客户端主机允许2个telnet连接:
iptables -A INPUT -p tcp --syn --dport 23 -m connlimit --connlimit-above 2 -j REJECT
#你也可以匹配其他方式：
iptables -A INPUT -p tcp --syn --dport 23 -m connlimit --connlimit-upto 2 -j ACCEPT
# 将并行HTTP请求的数量限制为每个C类源网络16个（24位网络掩码）:
iptables -p tcp --syn --dport 80 -m connlimit --connlimit-above 16 --connlimit-mask 24 -j REJECT
# 将链接本地网络的并行HTTP请求数限制为16:
(ipv6) ip6tables -p tcp --syn --dport 80 -s fe80::/64 -m connlimit --connlimit-above 16 --connlimit-mask 64 -j REJECT
# 限制与特定主机的连接数:
ip6tables -p tcp --syn --dport 49152:65535 -d 2001:db8::1 -m connlimit --connlimit-above 100 -j REJECT
 
```        

<br>
### iptables各模块     
#### nf_conntrack模块         
[传送门](https://clodfisher.github.io/2018/09/nf_conntrack/)    

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
[流量统计 iptables](https://blog.csdn.net/u014015972/article/details/50647039)     
[CentOS7启用iptables防火墙](https://www.jianshu.com/p/01edf3c67a1b)     
[CentOS7安装iptables防火墙](https://www.cnblogs.com/kreo/p/4368811.html)       
[netfilter内核模块知识 - 解决nf_conntrack: table full, dropping packet](https://github.com/digoal/blog/blob/master/201612/20161229_04.md)  
[iptables-extensions.man.html](http://ipset.netfilter.org/iptables-extensions.man.html)     
[iptables基础](https://blog.csdn.net/u014721096/article/details/78594645)        
[iptables之显式扩展](https://blog.csdn.net/u014721096/article/details/78605695)    
[Linux里的防火墙（下）：iptables的扩展模块——l7-filter的安装与功能实现](https://blog.csdn.net/deansrk/article/details/6706461)    
[Netfilter-wikipedia](https://en.wikipedia.org/wiki/Netfilter)    




       
<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Iptables汇总 ](https://clodfisher.github.io/2018/08/IptablesCollect/)      


  