---
layout: post
title: C中使用的类型是否必须声明
date: 2018-2-1 
tags: C    
---
<br>    
### 思考此问题原因   

由于看linux协议栈内核源码使遇到了如下情况：   
```
#ifndef _NET_NEIGHBOUR_H  
#define _NET_NEIGHBOUR_H  
  
#include <linux/neighbour.h>  
  
/* 
 *  Generic neighbour manipulation 
 * 
 *  Authors: 
 *  Pedro Roque     <roque@di.fc.ul.pt> 
 *  Alexey Kuznetsov    <kuznet@ms2.inr.ac.ru> 
 * 
 *  Changes: 
 * 
 *  Harald Welte:       <laforge@gnumonks.org> 
 *      - Add neighbour cache statistics like rtstat 
 */  
  
#include <asm/atomic.h>  
#include <linux/netdevice.h>  
#include <linux/skbuff.h>  
#include <linux/rcupdate.h>  
#include <linux/seq_file.h>  
  
#include <linux/err.h>  
#include <linux/sysctl.h>  
#include <linux/workqueue.h>  
#include <net/rtnetlink.h>  
  
/* 
 * NUD stands for "neighbor unreachability detection" 
 */  
  
#define NUD_IN_TIMER    (NUD_INCOMPLETE|NUD_REACHABLE|NUD_DELAY|NUD_PROBE)  
#define NUD_VALID   (NUD_PERMANENT|NUD_NOARP|NUD_REACHABLE|NUD_PROBE|NUD_STALE|NUD_DELAY)  
#define NUD_CONNECTED   (NUD_PERMANENT|NUD_NOARP|NUD_REACHABLE)  
  
struct neighbour;  
  
struct neigh_parms  
{  
#ifdef CONFIG_NET_NS  
    struct net *net;  
#endif  
    struct net_device *dev;  
    struct neigh_parms *next;  
    int (*neigh_setup)(struct neighbour *);  
    void    (*neigh_cleanup)(struct neighbour *);  
    struct neigh_table *tbl;  
  
    void    *sysctl_table;  
  
    int dead;  
    atomic_t refcnt;  
    struct rcu_head rcu_head;  
  
    int base_reachable_time;  
    int retrans_time;  
    int gc_staletime;  
    int reachable_time;  
    int delay_probe_time;  
  
    int queue_len;  
    int ucast_probes;  
    int app_probes;  
    int mcast_probes;  
    int anycast_delay;  
    int proxy_delay;  
    int proxy_qlen;  
    int locktime;  
};  
``` 
neigh_parms中用的struct neighbour类型在使用前声明了下，而struct neigh_table类型却没有进行任何声明，这样难道不会出错吗？    

<br>
### 对问题验证    

**验证一：**    

```
#include <stdio.h>  
  
struct str2;  
  
struct str1  
{  
    struct str2 st;  
    int i1;  
};  
  
struct str2  
{  
    int i2;  
};  
  
int main()  
{  
    struct str1 stest1;  
    stest1.st.i2 = 2;  
    stest1.i1 = 1;  
      
    printf("%d----%d\n", stest1.st.i2, stest1.i1);  
  
    return 0;  
}  
``` 
运行结果为：    
define1.c:7:17: 错误：字段‘st’的类型不完全    
     struct str2 st;    
                 ^    

**验证二：**    
```
#include <stdio.h>  
  
//typedef struct str2* ST;  
//struct str2;  
  
struct str1  
{  
//  ST  pst;  
    struct str2 * pst;  
    int i1;  
};  
  
struct str2  
{  
    int i2;  
};  
  
  
int main()  
{  
    struct str1 stest1;  
    struct str2 stest2;  
    stest1.pst = &stest2;  
    stest1.pst->i2 = 2;  
    stest1.i1 = 1;  
  
    printf("%d-----%d\n", stest1.pst->i2, stest1.i1);  
  
    return 0;  
}  
```

运行结果：    
$ gcc define.c    
$ ./a.out     
2-----1  

<br>
### 问题结论
用的是指针类型无妨，若是实体类型将会报错，究竟为什么怎样，是由于编译器决定的，本身不管是什么指针类型的字节数都是一样的         

转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [C中使用的类型是否必须声明](https://clodfisher.github.io/2018/02/CPointTypeStatement/)   