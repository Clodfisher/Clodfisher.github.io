---
layout: post
title: Docker命令小结
date: 2018-3-15 
tags: Docker        
---

<br>
### 基本命令        
**`docker run`**         
* docker run 容器标识 : 运行容器，该命令实际上是 docker create 和 docker start 的组合。
* docker run 容器标识 命令 ： 容器启动时执行相应的命令。   

**`-d`**           
* docker run -d 容器标识 : 以后台守护进程形式运行容器。     

**`-it`**    
* docker run -it 容器标识 : 以交互模式进入容器，并打开终端。       

**`--name`**      
* docker run 容器标识 --name "my_test"： 在启动容器时可以通过 --name 参数显示地为容器命名。    
 
**`--restart`**    
* docker run -d --restart=always 容器标识 : 容器可能会因某种错误而停止运行。对于服务类容器，我们通常希望在这种情况下容器能够自动重启。--restart=always 意味着无论容器因何种原因退出（包括正常退出），就立即重启。该参数的形式还可以是 --restart=on-failure:3，意思是如果启动进程退出代码非0，则重启容器，最多重启3次。   

**`-m 或 --memory`**      
* docker run -m 200M --memory-swap=300M ubuntu ： -m 或 --memory：设置**内存**的使用限额，例如 100M, 2G。--memory-swap：设置 **内存**+**swap** 的使用限额。    

**`--vm`**    
* docker run -it -m 200M --memory-swap=300M progrium/stress --vm 1 --vm-bytes 280M : --vm 1：启动 1 个内存工作线程。--vm-bytes 280M：每个线程分配 280M 内存。     

**`-c 或 --cpu-shares`**    
* docker run --name "container_A" -c 1024 ubuntu : 通过 cpu share 可以设置容器使用 CPU 的优先级。    

**`--cpu`**    
* docker run --name container_my it -c 1024 progrium/stress --cpu 1 ： --cpu 用来设置工作线程的数量。当前 host 只有 1 颗 CPU，所以一个工作线程就能将 CPU 压满。如果 host 有多颗 CPU，则需要相应增加 --cpu 的数量。    

**`-h`**    
* docker run -h myhostname -it ubuntu :  默认情况下，容器的 hostname 是它的短ID，可以通过 -h 或 --hostname 参数设置。    

**`images`**    
* docker images : 已经下载到本地的所有镜像信息。
* docker images 容器标识 ： 查看此镜像的信息    

**`ps`**    
* docker ps 或者 docker container ls 显示容器正在运行。     
* docker ps -a ： 查看所有状态的容器。    

**`rename`**    
* docker rename 旧容器名字 新容器名字 ： 重命名容器。    

**`pull`**    
* docker pull 容器标识： 从 Docker Hub 下载相应容器。    

**`push`**    
* docker push 镜像标识 ： 将镜像上传到 Docker Hub。     

**`commit`**    
* docker commit 镜像标识 新的镜像名字 ： 命令将容器保存为新的镜像。    

**`build `**    
* docker build -t 新镜像名字 ： 利用Dockerfile构建新的镜像。    
* docker build -t 新镜像名字:版本号 ： 利用Dockerfile构建新的镜像以及其版本号。    

**`login`**    
* docker login -u 用户名 ： 在docker host登入Docker Hub。     

**`tag`**    
* docker tag 旧镜像名字 新镜像名字 ： 重新命名镜像。    

**`history`**    
* docker history 镜像标识 ： 显示镜像的构建历史，也就是 Dockerfile 的执行过程。     

**`rmi`**    
* docker rmi 镜像名字[：版本] ： 删除 host 上的镜像.    

**`rm`**    
* docker rm 镜像标识 【镜像标识】： 删除一个或多个容器。   
* docker rm -v $(docker ps -aq -f status=exited) ： 批量删除所有已经退出的容器。    
* docker rm -v : ocker rm 删除容器时可以带上 -v 参数，docker 会将容器使用到的 volume 一并删除，但前提是没有其他容器 mount 该 volume，目的是保护数据，非常合理。果删除容器时没有带 -v 呢？这样就会产生孤儿 volume，好在 docker 提供了 volume rm 子命令可以对 docker managed volume 进行维护。    

**`search`**    
* docker search 镜像名字 ： 在命令行中就可以搜索 Docker Hub 中的镜像。    

**`stop`**    
* docker stop 容器标识 : 停止一个容器，命令本质上是向该进程发送一个 SIGTERM 信号。   

**`kill`**    
* docker kill 容器标识 ： 停止一个容器，命令本质上是向该进程发送一个 SIGKILL 信号。   

**`attach`**    
* docker attach 容器长ID ： 直接进入容器 启动命令 的终端，不会启动新的进程。    

**`exec`**    
* docker exec -it 容器短ID bash : 在容器中打开新的终端，并且可以启动新的bash进程。     

**`logs`**    
* docker logs -f 容器短ID : 查看启动命令的输出, -f 的作用与 tail -f 类似，能够持续打印输出。    

**`create`**    
* docker create 镜像名字 ： 创建的容器处于 Created 状态。    

**`start`**    
* docker start 容器标识 ： 对于处于停止状态或创建状态的容器，可以通过 docker start 重新启动，且会保留容器的第一次启动时的所有参数。    

**`restart`**    
* docker restart 容器标识 ： 可以重启容器，其作用就是依次执行 docker stop 和docker start。    

**`pause`**    
* docker pause 容器标识 ： 使容器处于暂停状态，不会占用 CPU 资源。    

**`unpause`**    
* docker unpause 容器标识 : 处于暂停状态的容器不会占用 CPU 资源，直到通过 docker unpause 恢复运行。    

<br>
### 网络配置     
**`network ls`**    
* docker network ls : 在docker host 上查看创建的网络。  

**`network inspect bridge`**    
* docker docker network inspect bridge  : 看一下 bridge 网络的配置信息。       

**`network connect`**    
docker network connect my_net 2b668e52480e : 为 httpd 容器添加一块 net_my 的网卡。    

**`create --driver `**    
Docker 提供三种 user-defined 网络驱动：bridge, overlay 和 macvlan。overlay 和 macvlan 用于创建跨主机的网络，bridge类似于自带的桥模式。    
* docker network create --driver bridge my_net : 通过桥驱动，创建了一个属于自己的网络。  

**`--subnet 与 --gateway`**    
* docker network create --driver bridge --subnet 172.22.16.0/24 --gateway 172.22.16.1 my_net : 将自己创建的网络，设置其所属的网段、默认网关。    

**`--ip`**    
* docker run -it --network=my_net2 --ip 172.22.16.8 busybox : 容器运行时从 subnet 中指定一个静态 IP。    

**`--network=`**     
* docker run -it --network=none busybox : 容器创建时，可以通过 --network=none 指定使用 none 网络。  

**`--network=container:[web1]`**    
* docker run -it --network=container:web1 busybox : joined 容器是另一种实现容器间通信的方式。joined 容器非常特别，它可以使两个或多个容器共享一个网络栈，共享网卡和配置信息，joined 容器之间可以通过 127.0.0.1 直接通信。请看下面的例子：先创建一个 httpd 容器，名字为 web1。docker run -d -it --name=web1 httpd ，然后创建 busybox 容器并通过 --network=container:web1 指定 jointed 容器为 web1。    

**`-p`**    
* docker run -d -p 80 httpd : 将容器对外提供服务的端口映射到host主机上的随机端口。    
* docker run -d -p 8080:80 httpd : 将容器的 80 端口映射到 host 的 8080 端口。    
        

<br>
### 存储配置        
**`--blkio-weight`**    
改变容器 Block IO 的优先级，目前 Block IO 限额只对 direct IO（不使用文件缓存）有效。  
* docker run -it --name container_A --blkio-weight 600 ubuntu   : 设置的是相对权重值600，默认为 500。   

**`bps 或 iops`**     
bps 是 byte per second，每秒读写的数据量。
iops 是 io per second，每秒 IO 的次数。
--device-read-bps，限制读某个设备的 bps。
--device-write-bps，限制写某个设备的 bps。
--device-read-iops，限制读某个设备的 iops。
--device-write-iops，限制写某个设备的 iops。
docker run -it --device-write-bps /dev/sda:30MB ubuntu ： 限制容器写 /dev/sda 的速率为 30 MB/s。    
 
**`info`**   
* docker info : 查看系统默认的driver。

**`-v`**   
* docker run -d -v <host path>:<container path>:<权限> httpd : 将某一个host文件mount到容器中，容器中已存在的目录文件(container path)数据将会被隐藏，取而代之的是host path目录下的数据。<权限>用于指定容器对于挂在数据的读写权限，默认权限是可读可写。    
* docker run -d -v <host path/file>:<container path/file>:<权限> httpd : 除了 bind mount 目录，还可以单独指定一个文件。使用 bind mount 单个文件的场景是：只需要向容器添加文件，不希望覆盖整个目录，同时也保留了容器原有的数据。        
* docker run -d -v <host path/file> httpd :  创建docker managed volume，无控制，均为读写权限。创建过程：容器启动时，简单的告诉 docker "我需要一个 volume 存放数据，帮我 mount 到目录 /abc"。docker 在 /var/lib/docker/volumes 中生成一个随机目录作为 mount 源。如果 /abc 已经存在，则将数据复制到 mount 源。将 volume mount 到 /abc。    

**`cp`**    
* docker cp hostfile 容器短ID:容器目录的全路径 ： 可以在容器和 host 之间拷贝数据。    

**`--volumes-from`**    
* docker run --name web1 -d -p 80 --volumes-from vc_data httpd : 其他容器可以通过 --volumes-from 使用 vc_data 这个 volume container。    

**`volume ls`**    
* docker volume ls : 查看使用的 docker managed volume 。    

**`volume rm`**    
* docker volume rm [docker managed volumeID] : 删除容器中的docker managed volume。    

**`volume prune`**    
* docker volume prune : 用这个命令删除孤儿 volume。    

 

----
该总结到44... 

<br>
### 数据管理配置        

<br>
### 容器监控配置    

<br>
### 日志管理配置    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Docker命令小结](https://clodfisher.github.io/2018/03/DockerCommand/)   