---
layout: post
title: Ubuntu镜像容器安装vim
date: 2018-3-6 
tags: Docker        
---

<br>
### 运行Ubuntu镜像容器    

使用`sudo docker run -it ubuntu`运行Ubuntu容器，-it 参数的作用是以交互模式进入容器，并打开终端。`34a11bd82bc9` 是容器的内部 ID。    
![](/images/posts/2018-3-6-UbuntuDockerInstallVim/UbuntuDockerInstallVim0.jpg)      

<br>
### 安装vim    

确认没有安装vim。     
![](/images/posts/2018-3-6-UbuntuDockerInstallVim/UbuntuDockerInstallVim1.jpg)     

指令安装vim。    
![](/images/posts/2018-3-6-UbuntuDockerInstallVim/UbuntuDockerInstallVim2.jpg)      
提示：
        Reading package lists... Done
        Building dependency tree       
        Reading state information... Done
        E: Unable to locate package vim

这时候需要敲：apt-get update，这个命令的作用是：同步 `/etc/apt/sources.list` 和 `/etc/apt/sources.list.d` 中列出的源的索引，这样才能获取到最新的软件包。等更新完毕以后再敲命令：`apt-get install vim`命令即可。            
![](/images/posts/2018-3-6-UbuntuDockerInstallVim/UbuntuDockerInstallVim3.jpg)     

<br>
### 保存为新镜像    
1. 在新窗口中查看当前运行的容器。    
![](/images/posts/2018-3-6-UbuntuDockerInstallVim/UbuntuDockerInstallVim4.jpg)    
`hardcore_tereshkova` 是Docker为我们容器随机分配的名字。    

2. 执行 docker commit 命令将容器保存为镜像。    
![](/images/posts/2018-3-6-UbuntuDockerInstallVim/UbuntuDockerInstallVim5.jpg)    
`ubuntu-with-vim`新的镜像名字。    

3. 查看新镜像的属性。    
从 size 上看到镜像因为安装了软件而变大了。    
![](/images/posts/2018-3-6-UbuntuDockerInstallVim/UbuntuDockerInstallVim6.jpg)    

4. 从新镜像启动容器，验证 vi 已经可以使用。    
![](/images/posts/2018-3-6-UbuntuDockerInstallVim/UbuntuDockerInstallVim7.jpg)    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Ubuntu镜像容器安装vim](https://clodfisher.github.io/2018/06/UbuntuDockerInstallVim/)   



   
        
