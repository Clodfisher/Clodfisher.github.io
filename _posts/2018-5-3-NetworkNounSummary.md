---
layout: post
title: 网络名词汇总
date: 2018-5-3 
tags: 知识汇总 test
---

<br>
### A    

**AH** : (Authentication Header, 验证头)主要用于IPSec中对数据添加验证头使用。   

**ACL** : （Access Control List，访问控制列表）基于ACL的包过滤防火墙是将制定出的不同区域间访问控制策略部署在路由器端口处过滤进出数据包，达到对访问进行控制，维护网络安全的目的。      

<br>
### C    

**CVE** :（Common Vulnerabilities and Exposures，通用漏洞与披露）用以查询漏洞的库。    

<br>
### D    

**DHCP** :（Dynamic Host Configuration Protocol，动态主机配置协议）通常被应用在大型的局域网络环境中，主要作用是集中的管理、分配IP地址，使网络环境中的主机动态的获得IP地址、Gateway地址、DNS服务器地址等信息，并能够提升地址的使用率。    

<br>
### E        

**ESP** : (Encapsulating Security Payload, 封装安全载荷)主要应用于IPSec中对于数据进行安全封装。    

<br>
### H    

**HDLC** :  高级数据链路控制（High-Level Data Link Control或简称HDLC），是一个在同步网上传输 数据、面向比特的数据链路层协议，它是由国际标准化组织（ISO）根据IBM公司的SDLC（Synchronous Data Link Control）协议扩展开发而成的。   

**HSTS** : HSTS的全英文名称为HTTP Strict Transport Security,中文译为HTTP严格传输安全，其是让浏览器强制使用HTTPS访问网站的一项安全策略，设计的初衷是缓解中间人攻击带来的风险。    

<br>
### N     

**NUD** : 邻居不可达检测（Neighbor Unreachable Detection）是节点确定邻居可达性的过程。邻居不可达检测机制通过邻居可达性状态机来描述邻居的可达性。    

<br>
### P     

**PDU** : 协议数据单元PDU（Protocol Data Unit）是指对等层次之间传递的数据单位。 协议数据单元(Protocol Data Unit )物理层的 PDU是数据位（bit），数据链路层的 PDU是数据帧（frame），网络层的PDU是数据包（packet），传输层的 PDU是数据段（segment），其他更高层次的PDU是报文（message）。   

<br>
### Q    

**QUIC**     
QUIC是快速UDP网络连接（英语：Quick UDP Internet Connections）的缩写，这是一种实验性的传输层网络传输协议，由Google公司开发，在2013年实现。QUIC使用UDP协议，它在两个端点间创建连接，且支持多路复用连接。在设计之初，QUIC希望能够提供等同于SSL/TLS层级的网络安全保护，减少数据传输及创建连接时的延迟时间，双向控制带宽，以避免网络拥塞。Google希望使用这个协议来取代TCP协议，使网页传输速度加快，计划将QUIC提交至互联网工程任务小组（IETF），让它成为下一代的正式网络规范。主要自己实现以下机制：自定义连接机制、自定义重传机制、无阻塞的多路复用、自定义流量控制、等。    

<br>
### S    

**SA** : (Security Association, 安全联盟)，IPSec中通信双方建立的连接叫作安全联盟，即通信双方结成盟友，使用相同的封装模式、加密算法、加密秘钥、验证算法、验证秘钥，相互信任亲密无间。安全联盟是单向的逻辑连接，为了使每个方向都得到保护，两个交互者双方每个方向都要建立安全联盟。为了区分这些不同方向方向的安全联盟，IPSec为每一个安全联盟都打上了唯一的标识符，这个标识符叫作SPI（Security Parameter Index）。建立安全联盟最直接的方式就是分别在交互的两者上认为设定好封装模式、加密算法、加密秘钥、验证算法、验证秘钥，即手工方式建立IPSec安全联盟。      



    
<br>
持续更新中......    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [网络名词汇总](https://clodfisher.github.io/2018/05/NetworkNounSummary/)      
