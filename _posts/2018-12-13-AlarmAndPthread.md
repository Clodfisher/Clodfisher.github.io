---
layout: post    
title: 定时器与线程之间的关系    
date: 2018-10-21    
tags: linux系统编程           
---

<br>
### 前言    
本文档主要是针对工作中对于定时器与线程涉及到的知识点进行总结，以及对定时器与线程相互之间的执行过程验证。    
其具体验证问题如下所示：    
* 如何实现单定时器，当定时时间到时，执行一次相应的操作。          
* 如何实现循环定时，每过一个时间段，执行一次相应的操作。          
* 对于定时器，当执行的操作的过程中，定时器有一次触发，会不会中断当前正在执行的操作。    
* 对于定时器计时，是否包含执行操作的时间，即当定时器执行的操作，小于定时器的时间，定时器的时间，是从定时器执行操作后开始计时，还是定时执行操作之前计时。      
* 当主进程包含一个定时器和一个线程时，若线程和定时器处理函数同时具有一把锁，是否会出现死锁。   

<br>
### 背景知识        
#### 定时器基本知识   
alarm函数是当到达某一个时间时，发送一个信号，执行相应的处理函数，此函数只执行一次。要是想循环定时处理某操作，需要使用linux内置的3个定时器，其如下所示：    
1. TIMER_REAL：实时定时器，不管进程在何种模式下运行（甚至在进程被挂起时），它总在计数。定时到达，向进程发送SIGALRM信号。    
2. ITIMER_VIRTUAL：这个不是实时定时器，当进程在用户模式（即程序执行时）计算进程执行的时间。定时到达后向该进程发送SIGVTALRM信号。     
3.ITIMER_PROF：进程在用户模式（即程序执行时）和核心模式（即进程调度用时）均计数。定时到达产生SIGPROF信号。ITIMER_PROF记录的时间比ITIMER_VIRTUAL多了进程调度所花的时间。    

定时器在初始化是，被赋予一个初始值，随时间递减，递减至0后发出信号，同时恢复初始值。在任务中，我们可以一种或者全部三种定时器，但同一时刻同一类型的定时器只能使用一个。    

#### 信号处理基本知识    
内核如何实现信号捕捉如下所示：    
如果信号的处理动作是用户自定义函数，在信号递达时就调用这个函数，这称为捕捉信号。由于信号处理函数的代码是在用户空间的，处理过程比较复杂，举例如下：    

1. 用户程序注册了SIGQUIT信号的处理函数sighandler。     
2. 当前正在执行main函数，这时发生中断或异常切换到内核态。     
3. 在中断处理完毕后要返回用户态的main函数之前检查到有信号SIGQUIT递达。    
4. 内核决定返回用户态后不是恢复main函数的上下文继续执行，而是执行sighandler函数，sighandler和main函数使用不同的堆栈空间，它们之间不存在调用和被调用的关系，是两个独立的控制流程。    
5. sighandler函数返回后自动执行特殊的系统调用sigreturn再次进入内核态。    
6. 如果没有新的信号要递达，这次再返回用户态就是恢复main函数的上下文继续执行了。     
![信号的捕捉](/images/posts/2018-12-13-AlarmAndPthread/AlarmAndPthread0.jpg)          

<br>    
### 验证过程    
#### 单次执行一次定时器    
```
#include <stdio.h>
#include <unistd.h>
#include <sys/time.h>
#include <signal.h>

void func()
{
    printf("2 s reached.\n");
}

int main()
{
   signal(SIGALRM,func);
   alarm(2);
   int i = 0;
   for(i = 0; i < 5; i++)
   {
      printf("i = %d\n", i);
      sleep(1);
   }
   while(1);
   return 0;
}
```
输出
```
[root@localhost test]# ./a.out 
i = 0
i = 1
2 s reached.
i = 2
i = 3
i = 4
```
#### 循环定时器，处理函数大于定时器时间    
```
#include <stdio.h>
#include <signal.h>
#include <sys/time.h>
#include <stdlib.h>
#include <pthread.h>
#include <errno.h>

pthread_mutex_t mutex;
int limit = 10;
/* signal process */
void timeout_info(int signo)
{
    printf("time action ....\n");
    int i;
    for( i = 1; i < 5; i++)
    {
        printf(" time sleep %d ...\n", i);
        sleep(1);
    }

}

/* init sigaction */
void init_sigaction(void)
{
    struct sigaction act;

    act.sa_handler = timeout_info;
    act.sa_flags   = 0;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM, &act, NULL);
    //sigaction(SIGVTALRM, &act, NULL);
    //sigaction(SIGPROF, &act, NULL);
}

/* init */
void init_time(void)
{
    struct itimerval val;

    val.it_value.tv_sec = 2;
    val.it_value.tv_usec = 0;
    val.it_interval = val.it_value;
    setitimer(ITIMER_REAL, &val, NULL);
    //setitimer(ITIMER_VIRTUAL, &val, NULL);
    //setitimer(ITIMER_PROF, &val, NULL);
}

int main(void)
{
    int ret = 0;
    printf("main func start .... .\n");
    init_sigaction();
    init_time();

    int i;
    for( i = 1; i < 100; i++)
    {
        printf("sleep %d ...\n", i);
        sleep(1);
    }

   while(1);
   return 0;
}
```
输出：    
```
[root@localhost test]# ./a.out 
main func start .... .
sleep 1 ...
sleep 2 ...
time action ....
 time sleep 1 ...
 time sleep 2 ...
 time sleep 3 ...
 time sleep 4 ...
time action ....
 time sleep 1 ...
 time sleep 2 ...
 time sleep 3 ...
 time sleep 4 ...
time action ....
 time sleep 1 ...
 time sleep 2 ...
 time sleep 3 ...
 time sleep 4 ...
time action ....
 time sleep 1 ...
 time sleep 2 ...
 time sleep 3 ...
 time sleep 4 ...
time action ....
 time sleep 1 ...
 time sleep 2 ...
```    
结论：    
当定时器的处理函数大于定时器轮询时间，若新的定时器触发信号，信号将会再原来的处理函数后面排队，当上一个定时器执行函数，执行完后，才接着执行排队的定时器处理函数，而主进程函数将会得不到执行。    

#### 循环定时器，处理函数小于定时器时间    
```
将上面的验证代码的定时器2秒改为10秒
/* init */
void init_time(void)
{
    struct itimerval val;

    val.it_value.tv_sec = 10;
    val.it_value.tv_usec = 0;
    val.it_interval = val.it_value;
    setitimer(ITIMER_REAL, &val, NULL);
    //setitimer(ITIMER_VIRTUAL, &val, NULL);
    //setitimer(ITIMER_PROF, &val, NULL);
}
```
输出：    
```
[root@localhost test]# ./a.out 
main func start .... .
sleep 1 ...
sleep 2 ...
sleep 3 ...
sleep 4 ...
sleep 5 ...
sleep 6 ...
sleep 7 ...
sleep 8 ...
sleep 9 ...
sleep 10 ...
time action ....
 time sleep 1 ...
 time sleep 2 ...
 time sleep 3 ...
 time sleep 4 ...
sleep 11 ...
sleep 12 ...
sleep 13 ...
sleep 14 ...
sleep 15 ...
sleep 16 ...
time action ....
 time sleep 1 ...
 time sleep 2 ...
 time sleep 3 ...
```
结论：    
通过上面的数据，可以验证，定时器开始重新计时时间，是在上一次定时器结束后开始，即定时器处理函数开始时，重新计时。    

#### 线程和定时器处理函数同时具有一把锁    
```
#include <stdio.h>
#include <signal.h>
#include <sys/time.h>
#include <stdlib.h>
#include <pthread.h>
#include <errno.h>

pthread_mutex_t mutex;
int limit = 10;
/* signal process */
void timeout_info(int signo)
{
    printf("  time lock ....\n");
    pthread_mutex_lock(&mutex);
    printf("  time lock end....\n");

    printf("time action ....\n");

    printf("  time unlock ....\n");
    pthread_mutex_unlock(&mutex);
    printf("  time unlock end ....\n");

}

/* init sigaction */
void init_sigaction(void)
{
    struct sigaction act;

    act.sa_handler = timeout_info;
    act.sa_flags   = 0;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM, &act, NULL);
    //sigaction(SIGVTALRM, &act, NULL);
    //sigaction(SIGPROF, &act, NULL);
}

/* init */
void init_time(void)
{
    struct itimerval val;

    val.it_value.tv_sec = 2;
    val.it_value.tv_usec = 0;
    val.it_interval = val.it_value;
    setitimer(ITIMER_REAL, &val, NULL);
    //setitimer(ITIMER_VIRTUAL, &val, NULL);
    //setitimer(ITIMER_PROF, &val, NULL);
}

static void* pthread_func_1 (void* data)
{
   while(1)
 {
   int i = 0;
   printf("  pthread lock ....\n");
   pthread_mutex_lock(&mutex);
   printf("  pthread lock end....\n");
   for (i; i < 6; i++)
   {
     printf ("This is pthread1: i = %d\n", i);

     sleep (2);
   }
   printf("  pthread unlock ....\n");
   pthread_mutex_unlock(&mutex);
   printf("  pthread unlock end....\n");
  }
}

int main(void)
{
    int ret = 0;
    pthread_mutex_init(&mutex,NULL);
    printf("You have only 10 seconds for thinking.\n");
    init_sigaction();
    init_time();

    pthread_t pt_1 = 0;

    ret = pthread_create (&pt_1, NULL, pthread_func_1, NULL);
    if (ret != 0)
    {
       perror ("pthread_1_create");
    }

   while(1);
   return 0;
}
```
输出：    
```
[root@localhost test]# ./a.out 
You have only 10 seconds for thinking.
  pthread lock ....
  pthread lock end....
This is pthread1: i = 0
  time lock ....
This is pthread1: i = 1
  time lock ....

```   
结论：   
出现死锁    

<br>
### 结论总结                 
* 对于定时器，当执行的操作的过程中，定时器有一次触发，会不会中断当前正在执行的操作。    
答：当定时器的处理函数大于定时器轮询时间，若新的定时器触发信号，信号将会再原来的处理函数后面排队，当上一个定时器执行函数，执行完后，才接着执行排队的定时器处理函数，而主进程函数将会得不到执行。      

* 对于定时器计时，是否包含执行操作的时间，即当定时器执行的操作，小于定时器的时间，定时器的时间，是从定时器执行操作后开始计时，还是定时执行操作之前计时。  
答：通过上面的数据，可以验证，定时器开始重新计时时间，是在上一次定时器结束后开始，即定时器处理函数开始时，重新计时。         

* 当主进程包含一个定时器和一个线程时，若线程和定时器处理函数同时具有一把锁，是否会出现死锁。   
答：出现死锁。    

<br>
参考链接：    
[Linux定时器的使用](http://www.cnblogs.com/feisky/archive/2010/03/20/1690561.html)        
[linux系统编程之信号（四）：信号的捕捉与sigaction函数](https://blog.csdn.net/jnu_simba/article/details/8947410)        
[Linux 线程挂起与唤醒功能 实例](https://blog.csdn.net/waldmer/article/details/23422943)    
[linux多线程学习(二)——线程的创建和退出](https://blog.csdn.net/wtz1985/article/details/3792770)             
[小记——linux定时器之alarm](https://blog.csdn.net/u010155023/article/details/51984602)    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [向有序单向链表添加新的节点](https://clodfisher.github.io/2018/12/AlarmAndPthread/)            