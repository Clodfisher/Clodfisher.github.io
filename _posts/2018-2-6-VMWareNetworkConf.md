---
layout: post
title: VMWare虚拟机网络配置
date: 2018-2-2 
tags: 虚拟化    
---
<br>    
### 名词解释    

该部分用于说明在VMWare虚拟机中涉及到的名词，图标，以及对应关系    

#### 虚拟交换机    

安装VMWare Workstation11就在物理机上有20个虚拟交换机，这些交换机彼此独立，互不连接。  
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf1.jpg)     
与虚拟机的关系为：    
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf2.jpg)    
只有连接同一个虚拟交换，且同一个网段的虚拟机、宿主机才能相互通信。

#### 虚拟网络适配器   

VMWare Workstation11在物理机中虚拟出来的一块网卡，与真实的网卡是平等关系，主要用于物理机和虚拟机进行通信。
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf3.jpg)    

相应的物理虚拟网卡适配器与连接在同一个虚拟交换机上的虚拟机通信。    
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf4.jpg)   
**注意：**虚拟机网卡与虚拟机网络适配器的不同点。    
虚拟机网卡，是指在VMWare Workstation11运行的虚拟机网卡例如linux虚拟机中的eth0等。    
虚拟机适配器称为物理机虚拟网卡，连接虚拟交换机使用，目的是与虚拟机网卡通信。    

#### VMWare DHCP Server    

VMWare WorkStation11安装后，会安装此服务，该服务可以为VMnet的计算机自动分配IP地址。 
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf5.jpg)    
在物理机的服务中可以看项。  
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf6.jpg)     

#### 桥接

桥接就是将物理机真实网卡与虚拟机虚拟网卡（这时候与物理机的虚拟网卡适配器没有关系）利用网桥进行通信。    
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf7.jpg)     
如上所示，将物理机真实网卡的出端口作为VMnet0虚拟交换的端口使用（交换的端口是没有IP和mac），而将物理机真实网卡IP地址设置为VMnet0交换机上的一个逻辑网卡。所以VM1访问C1是同个10.7.10.30访问的，而不是通过真实的网卡10.7.10.50。   

#### VMare Bridge Protocol    
虚拟桥接协议，物理网卡只有绑定了这个协议才能够让物理网卡连接到虚拟交换机VMnet。    
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf11.jpg)    

<br>    
### 三种工作模式    

#### Bridged(桥接模式) 
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf8.jpg)     

#### NAT(网络地址转换模式)
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf9.jpg)     

#### Host-Only(主机模式)
![](/images/posts/2018-2-6-VMWareNetworkConf/VMWareNetworkConf10.jpg)     

参考链接：    
[VMWare虚拟机网络配置](http://jwcqc.me/2016/08/18/vmware-network-configuration/)        
[Linux 上的基础网络设备详解](https://www.ibm.com/developerworks/cn/linux/1310_xiawc_networkdevice/index.html)        

转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [VMWare虚拟机网络配置](https://clodfisher.github.io/2018/02/VMWareNetworkConf/)   




