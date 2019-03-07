---
layout: post    
title: ebtable现场小实践    
date: 2019-02-26    
tags: 实战运维              
---
<br>
### 现场问题描述            
在远程支持现场中，有个如下图所示的现场环境，客户端通过交换机连接设备的两个网卡eth0（设备做了特殊处理，数据包处理的比较慢）和eth1，同时通过设备的eth3连接到服务器，由于现场中特殊要求，客户端只能和eth0通信，不和服务器通信，同时，服务器的IP、设备Ip和客户端Ip不可以变，同时网络拓扑也不可以修改，因为有其它用途。        
【现场网络拓扑图如下所示：】    
![](/images/posts/2019-3-7-EbtablesArp/EbtablesArp0.jpg)                

<br>
### 问题分析    
由于客户端Ip、设备eth0Ip和服务器IP都在同一个网段，但此时网卡eth0的Ip和服务器Ip出现冲突。在满足现场需求，需要将br0中客户端去往服务器的数据包拦截掉，此时采用ebtables，若采用iptables怕三层的forward连中配置的规则，影响到二层中forward链的数据。       
【iptables和ebtables数据流图如下所示：】     
![](/images/posts/2019-3-7-EbtablesArp/EbtablesArp1.jpg)             

<br>
### 解决方法    
1. 通过ebtables下发如下命令：    
`ebtables -I FORWARD -p IPv4 --ip-dst 192.168.6.101 -j DROP`    
此时客户端到服务器ping却是不通了，但是此时应该能够ping通eth0呢，没什么eth0也是ping不通，通过抓包可知，arp数据包能通过设备，学习到mac地址，但是ping数据请求会被丢弃。此时应当做的时客户端应该获取到192.168.10.101的mac为设备而不是服务器，此时ebtables应该丢弃arp请求，配置命令如下所示：    
`ebtables -I FORWARD -p arp --arp-ip-dst 192.168.6.101 -j DROP`。     
【不通问题，抓包图：】    
![](/images/posts/2019-3-7-EbtablesArp/EbtablesArp2.jpg)             

<br>       
### 总结        
1. 对于网络不通的情况，要从二层arp开始分析，不能单单局限于ip层，因为虽然第一次下发的规则，将ip层丢弃了，但是二层arp放过了，导致客户端获取到的mac还是服务器的mac。    



<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [ebtable现场小实践](https://clodfisher.github.io/2019/03/EbtablesArp/)           
