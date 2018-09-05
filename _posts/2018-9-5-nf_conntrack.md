---
layout: post    
title: Iptables之nf_conntrack模块    
date: 2018-8-26    
tags: 网络安全 iptables           
---

<br>
### nf_conntrack模块     
nf_conntrack(在老版本的 Linux 内核中叫 ip_conntrack)是一个内核模块,用于跟踪一个连接的状态的。连接状态跟踪可以供其他模块使用,最常见的两个使用场景是 iptables 的 nat 的 state 模块。
iptables 的 nat 通过规则来修改目的/源地址,但光修改地址不行,我们还需要能让回来的包能路由到最初的来源主机。这就需要借助 nf_conntrack 来找到原来那个连接的记录才行。而 state 模块则是直接使用 nf_conntrack 里记录的连接的状态来匹配用户定义的相关规则。例如下面这条 INPUT 规则用于放行 80 端口上的状态为 NEW 的连接上的包。    

`iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT。`    

<br>
### nf_conntrack模块常用命令    

```
查看nf_conntrack表
cat /proc/net/nf_conntrack

通过conntrack命令行工具查看conntrack的内容
yum install -y conntrack  
conntrack -L  

加载对应跟踪模块
[root@plop ~]# modprobe /proc/net/nf_conntrack_ipv4    
[root@plop ~]# lsmod | grep nf_conntrack    
nf_conntrack_ipv4       9506  0    
nf_defrag_ipv4          1483  1 nf_conntrack_ipv4    
nf_conntrack_ipv6       8748  2    
nf_defrag_ipv6         11182  1 nf_conntrack_ipv6    
nf_conntrack           79758  3 nf_conntrack_ipv4,nf_conntrack_ipv6,xt_state    
ipv6                  317340  28 sctp,ip6t_REJECT,nf_conntrack_ipv6,nf_defrag_ipv6  

移除 nf_conntrack 模块
$ sudo modprobe -r xt_NOTRACK nf_conntrack_netbios_ns nf_conntrack_ipv4 xt_state
$ sudo modprobe -r nf_conntrack

查看当前的连接数:
grep nf_conntrack /proc/slabinfo

查出目前 nf_conntrack 的排名:
cat /proc/net/nf_conntrack | cut -d ' ' -f 10 | cut -d '=' -f 2 | sort | uniq -c | sort -nr | head -n 10
```

<br>   
### nf_conntrack会话表的内容解释    
#### **会话表样例**    
通过`conntrack -L`与`/proc/net/nf_conntrack`是完全一样的，除了少了前面的两列。    
下面以`cat /proc/net/nf_conntrack`为例进行说明：    
```
ipv4  2  tcp  6  25   SYN_SENT     src=182.168.77.7  dst=42.236.9.57     sport=57430  dport=443  [UNREPLIED]       src=42.236.9.57     dst=182.168.77.7  sport=443    dport=57430  mark=0  secctx=system_u:object_r:unlabeled_t:s0  zone=0  use=2
ipv4  2  tcp  6  299  ESTABLISHED  src=172.18.15.56  dst=172.18.15.96    sport=40248  dport=22   src=172.18.15.96  dst=172.18.15.56    sport=22          dport=40248  [ASSURED]    mark=0  secctx=system_u:object_r:unlabeled_t:s0  zone=0  use=2
ipv4  2  tcp  6  5    SYN_SENT     src=182.168.77.7  dst=221.181.72.250  sport=57428  dport=443  [UNREPLIED]       src=221.181.72.250  dst=182.168.77.7  sport=443    dport=57428  mark=0  secctx=system_u:object_r:unlabeled_t:s0  zone=0  use=2
ipv4  2  tcp  6  1    SYN_SENT     src=182.168.77.7  dst=221.181.72.250  sport=57427  dport=80   [UNREPLIED]       src=221.181.72.250  dst=182.168.77.7  sport=80     dport=57427  mark=0  secctx=system_u:object_r:unlabeled_t:s0  zone=0  use=2
```
#### **每一列表达的意思**    
* 第一列：网络层协议名字。    
* 第二列：网络层协议号。    
* 第三列：传输层协议名字。    
* 第四列：传输层协议号。    
* 第五列：无后续包进入时无效的秒数，即老化时间。    
* 第六列：不是所有协议都有，连接状态。    
* 其它的列都是通过名字的方式（key与value对)表述，或和呈现标识（[UNREPLIED], [ASSURED], ...）。一行的不同列可能包含相同的名字（例如src和dst），第一个表示请求方，第二个表示应答方。    

#### **呈现标识含义**    
* `[ASSURED]`: 在两个方面（即请求和响应）方向都看到了流量。    
* `[UNREPLIED]`: 尚未在响应方向上看到流量。如果连接跟踪缓存溢出，则首先删除这些连接。  

请注意，某些列名仅出现在特定协议中（例如，TCP和UDP的sport和dport，ICMP的type和code）。
仅当内核使用特定选项构建时，才会显示其他列名称（例如mark）。  

#### **举例说明**    
* `ipv4 2 tcp 6 300 ESTABLISHED src=1.1.1.2 dst=2.2.2.2 sport=2000 dport=80 src=2.2.2.2 dst=1.1.1.1 sport=80 dport=12000 [ASSURED] mark=0 use=2`    
属于从主机1.1.1.2，端口2000到主机2.2.2.2，端口80的已建立的TCP连接，从中将响应发送到主机1.1.1.2，端口2000，在五分钟内超时。对于此连接，已在两个方向上看到数据包。    

* `ipv4 2 icmp 1 3 src=1.1.1.2 dst=1.1.1.1 type=8 code=0 id=32354 src=1.1.1.1 dst=1.1.1.2 type=0 code=0 id=32354 mark=0 use=2`    
属于从主机1.1.1.2到主机1.1.1.1的ICMP回应请求数据包，具有从主机1.1.1.1到主机1.1.1.2的预期回应应答数据包，在三秒内超时。响应目标主机不一定与请求源主机相同，因为请求源地址可能已被响应目标主机伪装。    

#### **主要标识**   
请注意，以下信息可能不是最新信息！    

Fields available for all entries:    
> * bytes (if accounting is enabled, request and response)    
> * delta-time (if CONFIG_NF_CONNTRACK_TIMESTAMP is enabled)    
> * dst (request and response)    
> * mark (if CONFIG_NF_CONNTRACK_MARK is enabled)    
> * packets (if accounting is enabled, request and response)    
> * secctx (if CONFIG_NF_CONNTRACK_SECMARK is enabled)    
> * src (request and response)    
> * use    
> * zone (if CONFIG_NF_CONNTRACK_ZONES is enabled)    

Fields available for dccp, sctp, tcp, udp and udplite transmission layer protocols:    
> * dport (request and response)    
> * sport (request and response)    

Fields available for icmp transmission layer protocol:    
> * code (request and response)    
> * id (request and response)    
> * type (request and response)    

Fields available for gre transmission layer protocol:    
> * dstkey (request and response)    
> * srckey (request and response)    
> * stream_timeout    
> * timeout    

Allowed values for the sixth field:        
* dccp transmission layer protocol     
> CLOSEREQ    
> CLOSING    
> IGNORE     
> INVALID     
> NONE    
> OPEN    
> PARTOPEN    
> REQUEST    
> RESPOND    
> TIME_WAIT        

* sctp transmission layer protocol        
> CLOSED    
> COOKIE_ECHOED    
> COOKIE_WAIT    
> ESTABLISHED    
> NONE    
> SHUTDOWN_ACK_SENT    
> SHUTDOWN_RECD    
> SHUTDOWN_SENT    

* tcp transmission layer protocol    
> CLOSE    
> CLOSE_WAIT    
> ESTABLISHED    
> FIN_WAIT    
> LAST_ACK    
> NONE    
> SYN_RECV    
> SYN_SENT    
> SYN_SENT2    
> TIME_WAIT    

<br>
### nf_conntrack相关内核参数和解释    
参考内核帮助文档`/usr/share/doc/kernel-doc-3.10.0/Documentation/networking/nf_conntrack-sysctl.txt`    
/proc/sys/net/netfilter/nf_conntrack_*:    

* nf_conntrack_acct    
值类型：BOOLEAN      
0 - disabled (default)  
not 0 - enabled      
启用连接跟踪流记帐。64位字节和数据包每个流量的计数器被添加。     

* nf_conntrack_buckets    
值类型：INTEGER (**read-only**)    
哈希表的大小。 如果在模块加载期间未指定为参数，则通过将总内存除以16384来计算默认大小以确定存储区的数量，但是哈希表将永远不会少于32并且限制为16384个存储区。 对于内存超过4GB的系统，它将是65536个桶。        

* nf_conntrack_checksum    
值类型：BOOLEAN       
0 - disabled      
not 0 - enabled (default)     
验证传入数据包的校验和。 具有错误校验和的数据包处于INVALID状态。 如果启用此选项，则不会考虑此类数据包进行连接跟踪。          

* nf_conntrack_count    
值类型：INTEGER (**read-only**)     
当前分配的流条目数。          
   
* nf_conntrack_events    
值类型：BOOLEAN     
0 - disabled      
not 0 - enabled (default)     
如果启用此选项，则连接跟踪代码将通过ctnetlink为用户空间提供连接跟踪事件。    

* nf_conntrack_events_retry_timeout    
值类型：INTEGER (seconds)     
default 15    
此选项仅在使用“可靠连接跟踪事件”时才相关。 通常，ctnetlink是“有损”的，也就是说，当用户空间监听器无法跟上时，事件通常会被丢弃。    
用户空间可以请求“可靠的事件模式”。 当此模式处于活动状态时，conntrack将仅在事件发布后销毁。 如果事件传递失败，则内核会定期重新尝试将事件发送到用户空间。    
这是内核在重新尝试传递destroy事件时应使用的最大间隔。    
数字越大意味着交付重试次数越少，处理待办事项的时间就越长。    

* nf_conntrack_expect_max    
值类型：INTEGER     
期望表的最大大小。 默认值为nf_conntrack_buckets / 256.最小值为1。     

* nf_conntrack_frag6_high_thresh    
值类型：INTEGER    
用于重组IPv6片段的最大内存。 当为达到重组目标时分配nf_conntrack_frag6_high_thresh字节的内存时，若超出此值片段处理程序将抛出数据包。     

* nf_conntrack_frag6_timeout    
值类型：INTEGER (seconds)      
default 60      
ipv6分片在内存中保存的老化时间。    

* nf_conntrack_generic_timeout    
值类型：INTEGER (seconds)    
default 600    
通用超时的默认值。 这指的是第4层未知/不支持的协议。    

* nf_conntrack_helper    
值类型：BOOLEAN    
0 - disabled      
not 0 - enabled (default)      
启用自动conntrack帮助程序分配。    

* nf_conntrack_icmp_timeout    
值类型：INTEGER (seconds)      
default 30      
ICMP连接状态超时的默认值。    

* nf_conntrack_icmpv6_timeout    
值类型：INTEGER (seconds)      
default 30      
ICMP6连接状态超时的默认值。    

* nf_conntrack_log_invalid     
值类型：INTEGER     
0   - disable (default)       
1   - log ICMP packets      
6   - log TCP packets      
17  - log UDP packets      
33  - log DCCP packets     
41  - log ICMPv6 packets     
136 - log UDPLITE packets      
255 - log packets of any protocol    
根据值类型记录指定类型无效数据包。    

* nf_conntrack_max    
值类型：INTEGER     
连接跟踪表的大小。 默认值为nf_conntrack_buckets值* 4。    

* nf_conntrack_tcp_be_liberal    
值类型：BOOLEAN    
0 - disabled (default)      
not 0 - enabled     
Be conservative in what you do, be liberal in what you accept from others.If it's non-zero, we mark only out of window RST segments as INVALID.     

* nf_conntrack_tcp_loose    
值类型：BOOLEAN      
0 - disabled     
not 0 - enabled (default)      
如果它设置为零，我们将禁用拾取已建立的连接。     

* nf_conntrack_tcp_max_retrans    
值类型：INTEGER      
default 3    
在未收到来自目标的（可接受）ACK的情况下可以重新传输的最大数据包数。 如果达到此数字，将启动更短的计时器。    

* nf_conntrack_tcp_timeout_close    
值类型：INTEGER (seconds)      
default 10       
  
* nf_conntrack_tcp_timeout_close_wait    
值类型：INTEGER (seconds)       
default 60       
  
* nf_conntrack_tcp_timeout_established    
值类型：INTEGER (seconds)       
default 432000 (5 days)    
默认是432000=3600*24*5即5天的超时时间，超时后清空对应的那条记录。         
  
* nf_conntrack_tcp_timeout_fin_wait     
值类型：INTEGER (seconds)       
default 120      
  
* nf_conntrack_tcp_timeout_last_ack     
值类型：INTEGER (seconds)      
default 30      
  
* nf_conntrack_tcp_timeout_max_retrans     
值类型：INTEGER (seconds)      
default 300       
  
* nf_conntrack_tcp_timeout_syn_recv    
值类型：INTEGER (seconds)      
default 60      
  
* nf_conntrack_tcp_timeout_syn_sent    
值类型：INTEGER (seconds)      
default 120      
  
* nf_conntrack_tcp_timeout_time_wait     
值类型：INTEGER (seconds)      
default 120       
  
* nf_conntrack_tcp_timeout_unacknowledged     
值类型：INTEGER (seconds)       
default 300     

* nf_conntrack_timestamp      
值类型：BOOLEAN       
0 - disabled (default)      
not 0 - enabled     
启用连接跟踪流时间戳。    

* nf_conntrack_udp_timeout     
值类型：INTEGER (seconds)      
default 30          
  
* nf_conntrack_udp_timeout_stream2     
值类型：INTEGER (seconds)      
default 180      
如果检测到UDP流，将使用此扩展超时。    

还有多少秒这条会话信息会从跟踪表清除，取决于超时参数的配置，以及是否有包传输，有包传输时，这个时间会重置为超时时间。  
  
<br>
### 如何判断会话表是否满    
当会话表中的记录大于内核设置nf_conntrack_max的值时，会导致会话表满。    
```
nf_conntrack_max - INTEGER  
        Size of connection tracking table.  Default value is  
        nf_conntrack_buckets value * 4.  
```

错误例子：
**less /var/log/messages**      
``` 
Nov  3 23:30:27 digoal_host kernel: : [63500383.870591] nf_conntrack: table full, dropping packet.  
Nov  3 23:30:27 digoal_host kernel: : [63500383.962423] nf_conntrack: table full, dropping packet.  
Nov  3 23:30:27 digoal_host kernel: : [63500384.060399] nf_conntrack: table full, dropping packet.  
```
<br>
### 会话表满的解决办法     
  
nf_conntrack table full的问题，会导致丢包，影响网络质量，严重时甚至导致网络不可用。

解决方法举例：

1、排查是否DDoS攻击，如果是，从预防攻击层面解决问题。

2、清空会话表。

重启iptables，会自动清空nf_conntrack table。注意，重启前先保存当前iptables配置(iptables-save > /etc/sysconfig/iptables ; service iptables restart)。

3、应用程序正常关闭会话

设计应用时，正常关闭会话很重要。

4、加大表的上限（需要考虑内存的消耗）

`sysctl -w net.nf_conntrack_max = 10240000`
  
永久生效
```
vi /etc/sysctl.conf  
net.nf_conntrack_max = 10240000  
```
<br>
### 计算公式    
可以增大 conntrack 的条目(sessions, connection tracking entries) CONNTRACK_MAX 或者增加存储 conntrack 条目哈希表的大小 HASHSIZE    
默认情况下，CONNTRACK_MAX 和 HASHSIZE 会根据系统内存大小计算出一个比较合理的值：    
对于 CONNTRACK_MAX，其计算公式：    
CONNTRACK_MAX = RAMSIZE (in bytes) / 16384 / (ARCH / 32)    
比如一个 64 位 48G 的机器可以同时处理 48*1024^3/16384/2 = 1572864 条 netfilter 连接。对于大于 1G 内存的系统，默认的 CONNTRACK_MAX 是 65535。    

对于 HASHSIZE，默认的有这样的转换关系：    
CONNTRACK_MAX = HASHSIZE * 8    
这表示每个链接列表里面平均有 8 个 conntrack 条目。其真正的计算公式如下：    
HASHSIZE = CONNTRACK_MAX / 8 = RAMSIZE (in bytes) / 131072 / (ARCH / 32)    
比如一个 64 位 48G 的机器可以存储 48*1024^3/131072/2 = 196608 的buckets(连接列表)。对于大于 1G 内存的系统，默认的 HASHSIZE 是 8192。    

可以通过 echo 直接修改目前系统 CONNTRACK_MAX 以及 HASHSIZE 的值：     
$ `echo 100000 > /proc/sys/net/netfilter/nf_conntrack_max`        
$ `echo 50000 > /proc/sys/net/netfilter/nf_conntrack_buckets`       

还可以缩短 timeout 的值：    
$`echo 600 > /proc/sys/net/netfilter/  nf_conntrack_tcp_timeout_established`       

<br>
参考链接：    
[解决 nf_conntrack: table full, dropping packet 的几种思路](http://jaseywang.me/2012/08/16/%E8%A7%A3%E5%86%B3-nf_conntrack-table-full-dropping-packet-%E7%9A%84%E5%87%A0%E7%A7%8D%E6%80%9D%E8%B7%AF/)  

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [Iptables之nf_conntrack模块](https://clodfisher.github.io/2018/09/nf_conntrack/)              