---
layout: post    
title: Shell命令EOF使用    
date: 2018-8-15    
tags: shell           
---

### 使用背景        
执行脚本的时候，需要往一个文件里自动输入N行内容。如果是少数的几行内容，还可以用echo追加方式，但如果是很多行，那么单纯用echo追加的方式显得不那么优雅！    
这个时候，就可以使用EOF结合cat命令进行行内容的追加。    


### 知识点梳理    
EOF是END Of File的缩写,表示自定义终止符.既然自定义,那么EOF就不是固定的,可以随意设置别名,在linux按ctrl-d就代表EOF.EOF一般会配合cat能够多行文本输出.其用法如下:    
```
<<EOF        
//开始....
EOF            //结束
```

还可以自定义，比如自定义：
```
<<BBB        //开始
....
BBB              //结束
```

通过cat配合重定向能够生成文件并追加操作,在它之前先熟悉几个特殊符号:    
```
< :输入重定向
> :输出重定向
>> :输出重定向,进行追加,不会覆盖之前内容
<< :标准输入来自命令行的一对分隔号的中间内容.
```
### 原理总结    
Shell中通常将EOF与 << 结合使用，表示后续的输入作为子命令或子Shell的输入，直到遇到EOF为止，再返回到主调Shell。    
可以把EOF替换成其他东西，意思是把内容当作标准输入传给程序。    
回顾一下< <的用法。当shell看到< <的时候，它就会知道下一个词是一个分界符。在该分界符以后的内容都被当作输入，直到shell又看到该分界符(位于单独的一行)。这个分界符可以是你所定义的任何字符串。   

### 简单应用    
1, 向文件test.sh里输入内容。
```
[root@slave-server opt]# cat << EOF >test.sh 
> 123123123
> 3452354345
> asdfasdfs
> EOF
[root@slave-server opt]# cat test.sh 
123123123
3452354345
asdfasdfs
```

2, 进行mysql数据库数据的读取：    
```
 #执行一段sql命令
 sql()
 {
    mysql -h 127.0.0.1 -N -u "$USER" --password="$PASSWORD" << EOF
	use project
	$*
	quit
	"
 EOF
 }

 TEST=$(sql)
 if [ $? -ne 0 ];then
	#读取数据库错误
	echo read mysql err.
	exit 1
 fi

 #通过DESC字段查找到所使用的时间节点的id
 RESULT=$(sql "select node_id from time_object where id=$DESC;")
``` 

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Shell命令EOF使用](https://clodfisher.github.io/2018/08/ShellEOF/)      
