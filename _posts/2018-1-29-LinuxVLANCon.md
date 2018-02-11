---
layout: post
title: Linux配置VLAN总结
date: 2018-1-29 
tags: 虚拟化    
---

### 物理网卡、子网卡、虚拟VLAN网卡的关系    

  **物理网卡**：物理网卡这里指的是服务器上实际的网络接口设备，在系统中可以看到的，比如2个物理网卡分别对应是eth0和eth1这两个网络接口。    
  **子网卡**：子网卡在这里并不是实际上的网络接口设备，但是可以作为网络接口在系统中出现，如eth0:1、eth1:2这种网络接口。它们必须要依赖于物理网卡，虽然可以与物理网卡的网络接口同时在系统中存在并使用不同的IP地址，而且也拥有它们自己的网络接口配置文件。但是当所依赖的物理网卡不启用时（Down状态）这些子网卡也将一同不能工作。    
  **虚拟VLAN网卡**：这些虚拟VLAN网卡也不是实际上的网络接口设备，也可以作为网络接口在系统中出现，但是与子网卡不同的是，他们没有自己的配置文件。他们只是通过将物理网加入不同的VLAN而生成的VLAN虚拟网卡。如果将一个物理网卡添加到多个VLAN当中去的话，就会有多个VLAN虚拟网卡出现，他们的信息以及相关的VLAN信息都是保存在/proc/net/vlan/config这个临时文件中的，而没有独自的配置文件。它们的网络接口名是eth0.1、eth1.2这种名字。    

### 基本配置总结    

  一：宿主机IP配置：
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf0.jpg)  

  二：对于桥接到宿主机的，物理网卡的单独配置：    
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf1.jpg)  
  效果如下：    
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf2.jpg)  
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf3.jpg)  
  
  三：通过配置文件方式，配置linux虚拟机的vlan：    
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf4.jpg)  
  重启机器效果如下：    
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf5.jpg)  
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf6.jpg)  
  成功之后接着执行的效果：    
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf7.jpg)  
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf8.jpg)  
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf9.jpg)  

  四：通过配置文件方式，配置linux虚拟机的vlan：    
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf10.jpg)  
  重启机器效果如下：    
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf11.jpg)  
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf12.jpg)  
  成功之后接着执行的效果：    
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf13.jpg)  
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf14.jpg)  

  四：通过命令行方式，配置linux虚拟机的vlan：      
  ![](/images/posts/2018-1-29-LinuxVlanConf/LinuxVlanConf15.jpg)  
  此命令可以添加到开启启动时执行的脚本里面        

<br>
  参考链接：    
  [Virtual Local Area Networks](https://wiki.ubuntu.com/vlan)        
  [Linux 上的基础网络设备详解](https://www.ibm.com/developerworks/cn/linux/1310_xiawc_networkdevice/index.html)        

<br>  
  转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Linux配置VLAN总结](https://clodfisher.github.io/2018/01/LinuxVLANCon/)   
