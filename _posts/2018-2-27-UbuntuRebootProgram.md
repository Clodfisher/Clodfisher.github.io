---
layout: post
title: Ubuntu下添加开机启动项方法
date: 2018-2-27 
tags: linux        
---
<br>
### 解决问题    
在Ubuntu服务启动时，自动启动snort+barnyard2程序，用于开机时自动启动入侵检测系统，用一下方法多次尝试，最终还是不能够实现，通过查看/var/log/syslog文件，发现是由于barnyard2启动限于mysql数据库导致的，随后在barnyard2启动前加上了启动延，才得以解决，具体代码如下：    
```
#!/bin/sh
#启动snort脚本，当snort进程已启动时先要杀死，再启动
 
SnortId=$(ps -e | grep -w snort | awk '{print $1}')

if [ "$SnortId" != "" ]
then
    kill -9 $SnortId
fi
 
snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth0 -D
 
if [ $? = 0 ]
then
    exit 0
else
    exit 2
fi  
```

```
#!/bin/sh
#启动barnyard2程序之前，先杀了barnyard2程序，再启动

Barnyard2=$(ps -e | grep -w barnyard2 | awk '{print $1}')
if [ "$Barnyard2" != "" ]
then 
    kill -9 $Barnyard2
fi

/usr/local/bin/barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -w /var/log/snort/barnyard2.waldo -g snort -u snort -D

if [ $? = 0 ]
then
    exit 0
else
    exit 1
fi
```

```
#!/bin/sh
#用于放入/etc/init.d/目录下进行开机启动，一定要使用start、stop、restart的格式

    ICMS_DIR="/home/yht/snort_src/yht/shell"

    case "$1" in
        start)
            echo "Start ICMS"

            sh $ICMS_DIR/run_snort.sh
			sleep 50
            sh $ICMS_DIR/run_barnyard2.sh
            echo "done."
            ;;
        stop)
            ;;
        restart)
            $0 stop
            $0 start
            ;;
        *)
            echo "Usage: /etc/init.d/S99icms {start|stop|restart}"
            ;;
    esac
```
随后执行命令` sudo update-rc.d S99icms defaults 95` 即可。     

<br>
### 添加开机启动项的2种方法

对于以下两种方法：    
方法一，只能够使用当前用户，也就是或不能够执行sudo的一些命名或脚本。    
方法二，其放入/etc/init.d/目录下的文件，一定要有固定的格式，随后用命令向rc0.d、rc1.d、rc*.d等建立一些链接。    
#### **方法一，编辑rc.loacl脚本**    
Ubuntu开机之后会执行/etc/rc.local文件中的脚本，所以我们可以直接在/etc/rc.local中添加启动脚本。当然要添加到语句：exit 0 前面才行。    
如：    
```
sudo vi /etc/rc.local
```
然后在 exit 0 前面添加好脚本代码。    

#### **方法二，添加一个Ubuntu的开机启动服务。**    

如果要添加为开机启动执行的脚本文件，可先将脚本复制或者软连接到/etc/init.d/目录下，然后用：`update-rc.d xxx defaults NN` 命令(NN为启动顺序)，将脚本添加到初始化执行的队列中去。    
注意如果脚本需要用到网络，则NN需设置一个比较大的数字，如99。    
1) 将你的启动脚本复制到 /etc/init.d目录下,以下假设你的脚本文件名为 test。        
2) 设置脚本文件的权限。     
```
$ sudo chmod 755 /etc/init.d/test
```
3) 执行如下命令将脚本放到启动脚本中去：    
```
$ cd /etc/init.d
$ sudo update-rc.d test defaults 95
```
注：其中数字95是脚本启动的顺序号，按照自己的需要相应修改即可。在你有多个启动脚本，而它们之间又有先后启动的依赖关系时你就知道这个数字的具体作用了。该命令的输出信息参考如下：     
```
update-rc.d: warning: /etc/init.d/test missing LSB information
update-rc.d: see <http://wiki.debian.org/LSBInitScripts>
Adding system startup for /etc/init.d/test ...
/etc/rc0.d/K95test -> ../init.d/test
/etc/rc1.d/K95test -> ../init.d/test
/etc/rc6.d/K95test -> ../init.d/test
/etc/rc2.d/S95test -> ../init.d/test
/etc/rc3.d/S95test -> ../init.d/test
/etc/rc4.d/S95test -> ../init.d/test
/etc/rc5.d/S95test -> ../init.d/test
```  

卸载启动脚本的方法：    
```
$ cd /etc/init.d
$ sudo update-rc.d -f test remove
```

命令输出的信息参考如下：    
```
Removing any system startup links for /etc/init.d/test ...
/etc/rc0.d/K95test
/etc/rc1.d/K95test
/etc/rc2.d/S95test
/etc/rc3.d/S95test
/etc/rc4.d/S95test
/etc/rc5.d/S95test
/etc/rc6.d/K95test
```

<br>
参考链接：    
[Ubuntu下添加开机启动项的2种方法](http://www.jb51.net/os/Ubuntu/181138.html)       



<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Ubuntu下添加开机启动项方法](https://clodfisher.github.io/2018/02/UbuntuRebootProgram/)   

