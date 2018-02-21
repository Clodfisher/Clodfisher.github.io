---
layout: post
title: Go基本信息
date: 2018-2-21 
tags: Go        
---

<br>
### Go开发环境安装目录说明    
对于开发环境的安装配置，请参考如下链接：    
[Go语言环境搭建详解](http://www.flysnow.org/2017/01/05/install-golang.html)    
安装之后目录结构如下：    
![](/images/posts/2018-2-21-GoBaseInfo/2018-2-21-GoBaseInfo1.jpg)     
主要文件夹功能如下所示：   
**-api**:用来存放依照Go版本顺序的API增量列表文件。这里的API包括公开的变量、常量、函数等。    
**-bin**:用于存放主要的标准命令文件，包括go、godoc和gofmt。    
**-blog**:用于存放官方博客中的所有文章。    
**-doc**:用于存放标准库的HTML格式的程序文档。可以通过godoc命令启动一个Web程序展现这些文档，godoc - http-:8080,直接本地访问即可。    
**-lib**:用于存放一些特殊的库文件。    
**-misc**:用于存放一些辅助类的说明和工具。    
**-pkg**:用于存放安装Go标准库后的所有归档文件。    
**-src**:用于存放Go自身，Go标准工具以及标准库的所有源码文件。    
**-test**:存放用来测试和验证Go本身的所有相关文件。    

<br>
### Go命令    
在命令行或终端输入go即可查看所有支持的命令。    
> go get: 获取远程包(需要提前安装git或hg)    
> go run: 直接运行程序       
> go build: 测试编译，检测是否有编译错误    
> go fmt: 格式化源码(部分IDE在保存时自动调用)    
> go install: 先编译包文件，随后编译整个程序    
> go test: 运行测试文件    
> go doc: 查看文档    
> go env: 查看Go语言相关的环境信息    
> go clean: 清除因执行其它Go命令而遗留下来的临时目录和文件。

<br>
### 工程结构    
go的工程目录结构主要是源代码相应地资源文件存放目录结构。其主要存放在工作区，工作区(GOPATH）其实就是一个对应于特定工程的目录，编译源代码等生成的文件都会放到这个目录下，它包含三个子目录：src目录，pkg目录，bin目录等，如下所示：
```
bin/   
    maphapp.exe    
pkp/    
    平台名/ 如：windows_amd64、linux_amd64    
          mymath.a    
          github.com/    
                    astaxie/    
                           beedb.a    
src/    
   maphapp/    
          main.go    
   mymath/    
         sqrt.go    
   github.com/    
             astaxie/    
                    beedb/    
                         beedb.go    
                         util.go          
```

* bin: 文化话存放`go install`命令生成的可执行文件。`go install`主要安装的是命令源码文件。   
* pkg: 存放通过`go install`命令安装后的代码包归档文件。`go install`主要安装的是库源码包文件。    
* src: 用于存放go源代码，不同工程项目的代码以包名区分。   

 通过上面可知，源码包都是存放到GOPATH的src目录下，它们都是通过包名来区分不同项目结构。有过java开发的都知道，使用包进行组织代码，包以网站域名开头就不会用重复，比如现在流行使用的托管平台github.com，因为每个人都是唯一的，所以不会用重复。它里面由是一github用户名命名的文件夹(astaxie),再向下还有项目名beedb,在向下为编写的go源代码。   

**怎样将远程包或项目添加到src目录**

go提供了一个获取远程包的工具go get,他需要一个完整的包名作为参数，只要这个完成的包名是可访问的，就可以被获取到，比如我们获取一个CLI的开源库：    
```
go get github.com/astaxie/beedb
```

就可以下载这个库到我们$GOPATH/src目录下了，这样我们就可以像导入其他包一样import了。    
那么我们如何引用一个包呢，也就是go里面的import。其实非常简单，通过包路径，包路径就是从src目录开始，逐级文件夹的名字用/连起来就是我们需要的包名，比如：    
```
import (
	"github.com/spf13/hugo/commands"
)
```

如果我们使用的远程包有更新，我们可以使用如下命令进行更新,多了一个-u标识。    
```
go get -u -v github.com/spf13/cobra/cobra
```
<br>
### 命令源码文件与库源码文件区别    
**命令源码文件**，就是声明属于main代码包并且包含无参数声明和结果声明的main函数的源码文件。这类源码文件是程序的入口，它们可以独立运行（使用go run命令），也可以通过go build或go install命令的到相应的可执行文件。    
**库源码文件**，则是指存在于某个代码包中的普通源码文件。     

<br>
参考链接：    
[Go语言环境搭建详解](http://www.flysnow.org/2017/01/05/install-golang.html)       
[Go编程基础](https://github.com/Unknwon/go-fundamental-programming)       
[Go并发编程实战](https://github.com/gopcp/example.v2/)       



<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Go基本信息](https://clodfisher.github.io/2018/02/GoBaseInfo/)   
  

