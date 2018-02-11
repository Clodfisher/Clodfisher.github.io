---
layout: post
title: Git 获取远程分支
date: 2018-2-8 
tags: Git        
---
<br>    
另一哥们将分支push到库中，我怎么获取到他的分支信息呢？    
例如： 远程仓库这是有master分支和develop分支，若我们git clone的话，这时候只有master分支，那么怎样把远程develop分支一并拉到本地呢？    
方法一：（测试可用）    
1.先用 git branch -a 查看下所有分支，如下：    
```
*master
 remotes/origin/0.0.0.60
 remotes/origin/HEAD->origin/master
 remotes/origin/develop
 remotes/origin/develop_yht
 remotes/origin/master
2.git checkout remotes/origin/develop
3.git checkout -b develop
```
方法二：    
如果安装了git客户端，直接选择fetch一下，就可以获取到了。    
如果用命令行，运行 git fetch，可以将远程分支信息获取到本地，再运行 git checkout -b local-branchname origin/remote_branchname  就可以将远程分支映射到本地命名为local-branchname  的一分支。     

<br>
参考链接：    
[Git 获取远程分支](http://www.cnblogs.com/hanxianlong)             

<br>
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Git 获取远程分支](https://clodfisher.github.io/2018/02/GitRemoteBranch/)   

