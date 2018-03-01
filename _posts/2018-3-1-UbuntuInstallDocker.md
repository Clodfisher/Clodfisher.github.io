---
layout: post
title: Ubuntu16.04安装docker
date: 2018-3-1 
tags: Docker        
---

在第一次在新的主机上安装Docker CE之前，需要设置docker仓库之后，再从存储库安装和更新docker。    

<br>
### 设置存储仓库    
    
1.**更新数据源**    

```
$ sudo apt-get update
```

2.**安装包，允许 apt 命令 HTTPS 访问 Docker 源**     

```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

3.**添加 Docker 官方的 GPG key值**    

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4.**验证fingerprint值是否正确**    

```
$ sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```

5.**将 Docker 的源添加到 /etc/apt/sources.list**    

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
<br>
### 安装Docker CE    

1.**更新数据源**    

```
$ sudo apt-get update
```

2.**安装Docker CE**

```
$ sudo apt-get install docker-ce
```

3.**验证是否安装成功**    

```
sudo docker run hello-world
```
输入如下：   
![](/images/posts/2018-3-1-UbuntuInstallDocker/UbuntuInstallDocker1.jpg)     

<br>
### 安装镜像版Apache HTTP Server    

环境就绪，运行如下命令：   
```
sudo docker run -d -p 80:80 httpd
```
其过程可以简单的描述为：
> 从 Docker Hub 下载 httpd 镜像。镜像中已经安装好了 Apache HTTP Server。    
> 启动 httpd 容器，并将容器的 80 端口映射到 host 的 80 端口。    

命令运行结果：    
![](/images/posts/2018-3-1-UbuntuInstallDocker/UbuntuInstallDocker2.jpg)     

通过浏览器验证容器是否正常工作。在浏览器中输入 http://[your ubuntu host IP]   
![](/images/posts/2018-3-1-UbuntuInstallDocker/UbuntuInstallDocker3.jpg)    

<br>
### 镜像下载加速    
由于 Docker Hub 的服务器在国外，下载镜像会比较慢。幸好 DaoCloud 为我们提供了免费的国内镜像服务。    

下面介绍如果使用镜像。    

1.在 daocloud.io 免费注册一个用户。    

2.登录后，点击顶部菜单“加速器”。    
![](/images/posts/2018-3-1-UbuntuInstallDocker/UbuntuInstallDocker4.jpg)     

3.copy “加速器”命令并在 host 中执行（你的命令可能跟我的会稍有不同）。    
![](/images/posts/2018-3-1-UbuntuInstallDocker/UbuntuInstallDocker5.jpg)     

4.重启 Docker deamon，即可体验飞一般的感觉。    

```
# systemctl restart docker.service
```
<br>
参考链接：    
[Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)       
[运行第一个容器](http://www.cnblogs.com/CloudMan6/p/6727146.html)       
[Ubuntu16.04安装docker](http://www.cnblogs.com/tangjizhong/p/6136747.html)       
[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)       

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Ubuntu16.04安装docker](https://clodfisher.github.io/2018/03/UbuntuInstallDocker/)   

    
