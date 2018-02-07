---
layout: post
title: Docker之基本概念
date: 2018-2-7 
tags: Docker        
---
<br>    
### 简介    

Docker编程语言go，其采用linux原有技术：    
- Linux Namespace是Linux提供的一种内核级别环境隔离的方法。    
- Linux CGroup全称Linux Control Group， 是Linux内核的一个功能，用来限制，控制与分离一个进程组群的资源（如CPU、内存、磁盘输入输出等）。    
- AUFS是一种Union File System，所谓UnionFS就是把不同物理位置的目录合并mount到同一个目录中。    
- DeviceMapper在内核中支持逻辑卷管理的通用设备映射机制，它为实现用于存储资源管理的块设备驱动提供了一个高度模块化的内核架构，它包含三个重要的对象概念，Mapped Device、Mapping Table、Target device。    

下面的图片比较了 Docker 和传统虚拟化方式的不同之处：    
![](/images/posts/2018-2-6-DockerBaseInfo/DockerBaseInfo1.jpg)     

Docker 跟传统的虚拟化方式相比优势：    
- 更高效的利用系统资源    
- 更快速的启动时间    
- 一致的运行环境    
- 持续交付和部署    
- 更轻松迁移    
- 更轻松维护和扩展    

<br> 
### Docker生命周期建立    

#### 镜像（Image）

Docker的镜像概念类似于虚拟机里的镜像，是一个只读的模板，一个独立的由多层文件系统联合文件系统，包括运行容器所需的数据，可以用来创建新的容器。    
采用技术UnionFS    
镜像是只读的，可以理解为静态文件。    

#### 容器（Container）    

容器是镜像的实例化，实质运行于自己的独立命名空间的进程。因此容器具有自己的root文件系统，网络配置，进程空间，用户ID空间。    
采用技术CGroup，Namespace，DeviceMapper    
相对于镜像来说容器是动态的，容器在启动的时候，以镜像为基础层创建一层可写层作为最上层。    

#### 仓库（Repository）
Docker 仓库的概念跟Git 类似，注册服务器可以理解为 GitHub 这样的托管服务。    

**三者关系**交互图如下所示：    
![](/images/posts/2018-2-6-DockerBaseInfo/DockerBaseInfo2.jpg)     

参考链接：    
[Docker-从入门到实践](https://yeasy.gitbooks.io/docker_practice/)        
[酷 壳–COOLSHELL](https://coolshell.cn/?s=DOCKER)        

转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [VMWare虚拟机网络配置](https://clodfisher.github.io/2018/07/DockerBaseInfo/)   



