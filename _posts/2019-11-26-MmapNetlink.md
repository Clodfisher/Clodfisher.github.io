---
layout: post    
title: 修改nfqueue支持零拷贝    
date: 2019-11-26    
tags: 性能优化              
---
<br> 
### 解决问题描述                 
产品中的snort采用nfqueue从内核层获取数据包到应用层，因为nfqueue是采用netlink实现，其中涉及到数据包从内核层到应用层传输时进行数据包拷。从应用层下发抉择数据到内核层也需要进行数据包的拷贝，导致性能比较差，从而将从netlink方面进行性能优化。采用的优化方式将snort中daq中获取数据包的源码nfqueue部分改为支持netlink mmap的方式。                           

<br>
### 研究netlink相关内容        
netlink相关的内容讲解网上有很多，此处就不多说，研究netlink相关的内容，是为了修改nfqueue中netlink相关部分，采用netlink mmap实现。此部分研究的策略为，先实现一个简单的netlink样例，再将此样例改为支持netlink mmap的方式。      

#### 一个简单的netlink样例    
##### 内核空间代码实现             
netlink_test_kernel.c    
```
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/skbuff.h>
#include <linux/init.h>
#include <linux/ip.h>
#include <linux/types.h>
#include <linux/sched.h>
#include <net/sock.h>
#include <linux/netlink.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("hunter");

#define MAX_MSGSIZE 125

#define NETLINK_TEST 30

struct sock *nl_sk = NULL;

//向用户空间发送消息的接口
int sendnlmsg(char *message,int pid)
{
    struct sk_buff *skb;
    struct nlmsghdr *nlh;

    int slen = 0;

    if(!message || !nl_sk){
        return -1;
    }

    slen = strlen(message);

    // 为新的 sk_buffer申请空间
    skb = nlmsg_new(slen, GFP_ATOMIC);
    if(!skb){
        printk(KERN_ERR "my_net_link: alloc_skb Error./n");
        return -2;
    }

    //用nlmsg_put()来设置netlink消息头部
    nlh = nlmsg_put(skb, 0, 0, NETLINK_TEST, slen, 0);
    if(nlh == NULL){
        printk("nlmsg_put failauer \n");
        nlmsg_free(skb);
        return -1;
    }

    memcpy(nlmsg_data(nlh), message, slen);

    //通过netlink_unicast()将消息发送用户空间由pid所指定了进程号的进程
    netlink_unicast(nl_sk, skb, pid, MSG_DONTWAIT);
    printk("send OK!\n");

    return 0;
}

static void nl_data_ready(struct sk_buff *skb)
{
    struct nlmsghdr *nlh = NULL;
    char *umsg = NULL;
    char kmsg[] = "hello users!!!";

    if(skb->len >= nlmsg_total_size(0))
    {
        nlh = nlmsg_hdr(skb);
        umsg = NLMSG_DATA(nlh);
        if(umsg)
        {
            printk("kernel recv from user: %s\n", umsg);
            sendnlmsg (kmsg, nlh->nlmsg_pid);
        }
    }
}

struct netlink_kernel_cfg cfg = {
    .input = nl_data_ready, /* set recv callback */
};

int myinit_module(void)
{
    printk("my netlink in\n");
    nl_sk = netlink_kernel_create(&init_net, NETLINK_TEST, &cfg);
    if(nl_sk == NULL)
        printk("kernel_create error\n");
    return 0;
}

void mycleanup_module(void)
{
    printk("my netlink out!\n");
    sock_release(nl_sk->sk_socket);
    netlink_kernel_release(nl_sk);
}

module_init(myinit_module);
module_exit(mycleanup_module);
```

##### 用户空间代码实现             
netlink_test_user.c    
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>


/* /usr/include/linux/netlink.h */
#include <linux/netlink.h>

#define NETLINK_TEST 30
int main()
{
    /* 建立socket */
    int skfd = socket(AF_NETLINK, SOCK_RAW, NETLINK_TEST);
    if(skfd == -1){
        perror("create socket error\n");
        return -1;
    }

    /* netlink socket */
    struct sockaddr_nl nladdr;

    /*init netlink socket */
    nladdr.nl_family = AF_NETLINK;  /* AF_NETLINK or PE_NETLINK */
    nladdr.nl_pad = 0;              /* not use */
    nladdr.nl_pid = 0;              /* 传送到内核 */
    nladdr.nl_groups = 0;           /* 单播*/

    /* bing 绑定netlink socket 与socket */
    if( 0 != bind(skfd, (struct sockaddr *)&nladdr, sizeof(struct sockaddr_nl))){
        perror("bind() error\n");
        close(skfd);
        return -1;
    }

    /* 构造msg消息*/
    struct msghdr msg;
    memset(&msg, 0, sizeof(msg));
    msg.msg_name = (void *)&(nladdr);
    msg.msg_namelen = sizeof(nladdr);

    #define MAX_MSGSIZE 1024
    char buffer[] = "hello kernel!!!";

    /* netlink 消息头 */
    struct nlmsghdr *nlhdr;
    nlhdr = (struct nlmsghdr *)malloc(NLMSG_SPACE(MAX_MSGSIZE));

    strcpy(NLMSG_DATA(nlhdr),buffer);

    nlhdr->nlmsg_len = NLMSG_LENGTH(strlen(buffer));
    nlhdr->nlmsg_pid = getpid();  /* self pid */
    nlhdr->nlmsg_flags = 0;


    struct iovec iov;

    iov.iov_base = (void *)nlhdr;
    iov.iov_len = nlhdr->nlmsg_len;
    msg.msg_iov = &iov;
    msg.msg_iovlen = 1;

    sendmsg(skfd, &msg, 0);

    /* recv */
    //memset((char *)NLMSG_DATA(nlhdr), 0, 1024);
    //recvmsg(skfd, &msg, 0);
    int rv=0;
    while ((rv = recvmsg(skfd, &msg, 0)) && rv >= 0) {
       printf("while .....\n");
       printf("kernel: %s\n", NLMSG_DATA(nlhdr));
    }

    //printf("kernel: %s\n", nlmsg_data(nlhdr));

    close(skfd);
    free(nlhdr);

    return 0;
}
```

##### Makefile相关文档        
```
CONFIG_MODULE_SIG=n
MODULE_NAME := netlink_test_kernel
obj-m :=$(MODULE_NAME).o
KERNEL_VER := $(shell uname -r)
KERNEL_DIR := /lib/modules/$(KERNEL_VER)/build
PWD := $(shell pwd)

all:
	$(MAKE) -C $(KERNEL_DIR) M=$(PWD)
	gcc -o netlink_test_user netlink_test_user.c
clean:
	rm -fr *.ko *.o *.cmd netlink_test_user $(MODULE_NAME).mod.c
```

##### 编译运行    
运行Makefile会生成相应的ko文件和相应的应用可执行文件    
首先将内核文件插入内核，为了能够看到内核调试信息，先用`dmesg -C`清空内核原来的信息，随后再插入内核ko模块，执行`dmesg`查看消息。     
```
root netlink$ dmesg -C
root netlink$ insmod netlink_test_kernel.ko 
root netlink$ dmesg 
[122839.783666] my netlink in
root netlink$ 
root netlink$ 
root netlink$ dmesg 
[122839.783666] my netlink in
[122852.543073] kernel recv from user: hello kernel!!!
[122852.543076] send OK!
root netlink$ 
```   

应用层模块执行相应的二进制文件。    
```
root netlink$ ./netlink_test_user 
while .....
kernel: hello users!!!

```     

#### 修改为支持netlink_mmap的netlink样例        
##### 内核空间代码实现                          
netlink_test_kernel.c     
```
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/skbuff.h>
#include <linux/init.h>
#include <linux/ip.h>
#include <linux/types.h>
#include <linux/sched.h>
#include <net/sock.h>
#include <linux/netlink.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("hunter");

#define MAX_MSGSIZE 125
#define NL_FR_SZ    16384
#define TDB_NLMSG_MAXSZ		(NL_FR_SZ / 2 - NLMSG_HDRLEN - MAX_MSGSIZE \
				 - MAX_MSGSIZE)

#define NETLINK_TEST 30

struct sock *nl_sk = NULL;

static int tdb_if_info(struct sk_buff *skb, struct netlink_callback *cb)
{
	struct nlmsghdr *nlh;

	nlh = nlmsg_put(skb, NETLINK_CB(cb->skb).portid, cb->nlh->nlmsg_seq,
			cb->nlh->nlmsg_type, TDB_NLMSG_MAXSZ, 0);
	if (!nlh)
		return -EMSGSIZE;

	nlmsg_data(nlh);

	/* Fill in response. */
        char kmsg[] = "hello users!!!";
        int slen = strlen(kmsg);
        memcpy(nlmsg_data(nlh), kmsg, slen);
	return 0;
}

static int tdb_if_proc_msg(struct sk_buff *skb, struct nlmsghdr *nlh)
{
    char kmsg[] = "hello users!!!";
    char *m = NULL;
    m = nlmsg_data(nlh);
    if(m)
    {
        printk("kernel recv from user: %s\n", m);
    }
    struct netlink_dump_control c = {
        .dump = tdb_if_info,
        //.data = m,
        .min_dump_alloc = NL_FR_SZ / 2,
    };
    return netlink_dump_start(nl_sk, skb, nlh, &c);
}

static void tdb_if_rcv(struct sk_buff *skb)
{
	/* TODO remove the mutex for concurrent user-space updates. */
        struct nlmsghdr *nlh = NULL;
	nlh = nlmsg_hdr(skb);
        tdb_if_proc_msg(skb, nlh);
}

struct netlink_kernel_cfg cfg = {
    .input = tdb_if_rcv, /* set recv callback */
};

int myinit_module(void)
{
    printk("my netlink in\n");
    nl_sk = netlink_kernel_create(&init_net, NETLINK_TEST, &cfg);
    if(nl_sk == NULL)
        printk("kernel_create error\n");
    return 0;
}

void mycleanup_module(void)
{
    printk("my netlink out!\n");
    sock_release(nl_sk->sk_socket);
    netlink_kernel_release(nl_sk);
}

module_init(myinit_module);
module_exit(mycleanup_module);
```

##### 用户空间代码实现             
netlink_test_user.c    
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <linux/socket.h>
#include <sys/mman.h>
#include <poll.h>
#include <errno.h>

/* /usr/include/linux/netlink.h */
#include <linux/netlink.h>

#define NETLINK_TEST 30

#ifndef SOL_NETLINK
#define SOL_NETLINK 270
#endif

int  g_skfd;
unsigned int g_ring_size;
void *g_rx_ring;
void *g_tx_ring;
unsigned int g_tx_fr_off;
unsigned int g_rx_fr_off;

int main()
{
    printf("run....\n");
    /* 建立socket */
    g_skfd = socket(AF_NETLINK, SOCK_DGRAM, NETLINK_TEST);
    if(g_skfd == -1){
        perror("create socket error\n");
        return -1;
    }

    struct sockaddr_nl addr = {
		.nl_family	= AF_NETLINK,
	};
    addr.nl_pid = getpid();
    /* bing 绑定netlink socket 与socket */
    if( 0 != bind(g_skfd, (struct sockaddr *)&addr, sizeof(addr))){
        perror("bind() error\n");
        close(g_skfd);
        return -1;
    }
    
    //Ring setup
    unsigned int block_size = 16 * getpagesize();
    printf("block_size=%d\n", block_size);
    struct nl_mmap_req req = {
    	.nm_block_size		= block_size,
    	.nm_block_nr		= 64,
    	.nm_frame_size		= 16384,
    	.nm_frame_nr		= 64 * block_size / 16384,
    };

    //void *rx_ring, *tx_ring;
    /* Configure ring parameters */
    if (setsockopt(g_skfd, SOL_NETLINK, NETLINK_RX_RING, &req, sizeof(req)) < 0)
    {
        printf("set rx_ring erro=%d!!!\n", errno);
        exit(1);
    }
    if (setsockopt(g_skfd, SOL_NETLINK, NETLINK_TX_RING, &req, sizeof(req)) < 0)
    {
        printf("set tx_ring erro!!!\n");
        exit(1);
    }
    /* Calculate size of each invididual ring */
    g_ring_size = req.nm_block_nr * req.nm_block_size;
    /* Map RX/TX rings. The TX ring is located after the RX ring */
    g_rx_ring = mmap(NULL, 2 * g_ring_size, PROT_READ | PROT_WRITE,
    	       MAP_SHARED, g_skfd, 0);
    if ((long)g_rx_ring == -1L)
    	exit(1);
    g_tx_ring = g_rx_ring + g_ring_size;

    //Message transmission
    struct nl_mmap_hdr *hdr;
    struct nlmsghdr *nlh_tx;
    
    hdr = g_tx_ring + g_tx_fr_off;
    if (hdr->nm_status != NL_MMAP_STATUS_UNUSED)
    	/* No frame available. Use poll() to avoid. */
    	exit(1);
    
    nlh_tx = (void *)hdr + NL_MMAP_HDRLEN;

    /* 构造msg消息*/
    struct msghdr msg;
    memset(&msg, 0, sizeof(msg));
    msg.msg_name = (void *)&(addr);
    msg.msg_namelen = sizeof(addr);

    #define MAX_MSGSIZE 1024
    char buffer[] = "hello kernel!!!";

    
    /* netlink 消息头 */
    //struct nlmsghdr *nlhdr;
    //nlhdr = (struct nlmsghdr *)malloc(NLMSG_SPACE(MAX_MSGSIZE));

    strcpy(NLMSG_DATA(nlh_tx),buffer);

    //nlh->nlmsg_len = NLMSG_LENGTH(strlen(buffer));
    nlh_tx->nlmsg_len = sizeof(*nlh_tx) + sizeof(buffer);
    nlh_tx->nlmsg_pid = getpid();  /* self pid */
    nlh_tx->nlmsg_flags = 0;

    printf("sendto.....\n");
    /* Fill frame header: length and status need to be set */
    hdr->nm_len	= nlh_tx->nlmsg_len;
    hdr->nm_status	= NL_MMAP_STATUS_VALID;
    struct sockaddr_nl nladdr = {
		.nl_family	= AF_NETLINK,
	};
    if (sendto(g_skfd, NULL, 0, 0, &nladdr, sizeof(nladdr)) < 0)
        exit(1);

    g_tx_fr_off = (g_tx_fr_off + req.nm_frame_size) % g_ring_size;

    struct nl_mmap_hdr *hdr_rx;
    struct nlmsghdr *nlh_rx;
    unsigned char buf[16384];
    ssize_t len;
    struct pollfd pfds[1];

    pfds[0].fd = g_skfd;
    pfds[0].events = POLLIN | POLLERR;
    pfds[0].revents = 0;
   
    if (poll(pfds, 1, -1) < 0 && errno != -EINTR)
   	exit(1);
   
   /* Check for errors. Error handling omitted */
   if (pfds[0].revents & POLLERR)
   	exit(1);
   
   /* If no new messages, poll again */
   if (!(pfds[0].revents & POLLIN))
   	exit(1);
   
   /* Process all frames */
   while (1) {
   	/* Get next frame header */
        printf("recev ....\n");
   	hdr_rx = g_rx_ring + g_rx_fr_off;
   
   	if (hdr_rx->nm_status == NL_MMAP_STATUS_VALID)
              {
                      printf("Regular memory mapped frame.\n");
   		/* Regular memory mapped frame */
   		nlh_rx = (void *)hdr_rx + NL_MMAP_HDRLEN;
   		len = hdr_rx->nm_len;
   
   		/* Release empty message immediately. May happen
   		 * on error during message construction.
   		 				 */
   		if (len == 0)
   			goto release;
   	} else if (hdr_rx->nm_status == NL_MMAP_STATUS_COPY) {
                      printf(" Frame queued to socket receive queue.\n");
   		/* Frame queued to socket receive queue */
   		len = recv(g_skfd, buf, sizeof(buf), MSG_DONTWAIT);
   		if (len <= 0)
   			break;
   		nlh_rx = buf;
   	} else
   		/* No more messages to process, continue polling */
   		break;

   	printf("kernel: %s\n", NLMSG_DATA(nlh_rx));
   
release:
		/* Release frame back to the kernel */
		hdr_rx->nm_status = NL_MMAP_STATUS_UNUSED;

		/* Advance frame offset to next frame */
		g_rx_fr_off = (g_rx_fr_off + req.nm_frame_size) % g_ring_size;
	}

    close(g_skfd);

    return 0;
}
```

##### 编译运行    
运行Makefile会生成相应的ko文件和相应的应用可执行文件    
首先将内核文件插入内核，为了能够看到内核调试信息，先用`dmesg -C`清空内核原来的信息，随后再插入内核ko模块，执行`dmesg`查看消息。     
```
root netlink$ dmesg -C
root netlink$ insmod netlink_test_kernel.ko 
root netlink$ dmesg 
[122839.783666] my netlink in
root netlink$ 
root netlink$ 
root netlink$ dmesg 
[122839.783666] my netlink in
[122852.543073] kernel recv from user: hello kernel!!!
```   

应用层模块执行相应的二进制文件。    
```
root netlink$ ./netlink_test_user 
run....
block_size=65536
sendto.....
recev ....
Regular memory mapped frame.
kernel: hello users!!!
recev ....
``` 

<br>
### 研究nfqueue相关内容        
经过上面的研究，如何将netlink改为支持零拷贝已实现。下一步研究nfqueue相关内容即可，nfqueue相关的内容讲解网上有很多，此处就不多说，研究nfqueue相关的内容，是为了修改nfqueue相关部分，采用netlink mmap实现。此部分研究的策略为，先实现一个简单的nfqueue样例，再将此样例改为支持netlink mmap的方式。      

#### 一个简单的nfqueue样例      
##### nfqueue_example.c    
```
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <errno.h>
#include <string.h>
#include <netinet/in.h>
#include <linux/ip.h>
#include <linux/udp.h>
#include <linux/tcp.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <linux/netfilter.h>
#include <libnetfilter_queue/libnetfilter_queue.h>
#include <libnetfilter_queue/libnetfilter_queue_udp.h>
#include <libnetfilter_queue/libnetfilter_queue_ipv4.h>

//#define QUEUE_NUM 10010

static uint16_t checksum(uint32_t sum, uint16_t *buf, int size)
{
    while (size > 1) {
        sum += *buf++;
        size -= sizeof(uint16_t);
    }
    if (size)
        sum += *(uint8_t *)buf;

    sum = (sum >> 16) + (sum & 0xffff);
    sum += (sum >>16);

    return (uint16_t)(~sum);
}

static uint16_t checksum_tcpudp_ipv4(struct iphdr *iph)
{
    uint32_t sum = 0;
    uint32_t iph_len = iph->ihl*4;
    uint32_t len = ntohs(iph->tot_len) - iph_len;
    uint8_t *payload = (uint8_t *)iph + iph_len;

    sum += (iph->saddr >> 16) & 0xFFFF;
    sum += (iph->saddr) & 0xFFFF;
    sum += (iph->daddr >> 16) & 0xFFFF;
    sum += (iph->daddr) & 0xFFFF;
    sum += htons(iph->protocol);
    // 这是一个bug
    //sum += htons(IPPROTO_TCP);
    sum += htons(len);

    return checksum(sum, (uint16_t *)payload, len);
}
static void udp_compute_checksum_ipv4(struct udphdr *udph, struct iphdr *iph)
{
    /* checksum field in header needs to be zero for calculation. */
    udph->check = 0;
    udph->check = checksum_tcpudp_ipv4(iph);
}


int cb(struct nfq_q_handle *qh, struct nfgenmsg *nfmsg, struct nfq_data *nfad, void *data) {
    uint32_t id;
    struct nfqnl_msg_packet_hdr *ph;
    unsigned char *payload;
    int r;
    struct iphdr    *iph;
    struct udphdr   *udph;
    struct tcphdr   *tcph;
    char saddr_str[16];
    char daddr_str[16];
    struct in_addr tmp_in_addr;
    
    ph = nfq_get_msg_packet_hdr(nfad);
    if (ph) {
        id = ntohl(ph->packet_id);
        r = nfq_get_payload(nfad, &payload);
        if (r >= sizeof(*iph)) {
            iph = (struct iphdr *)payload;
            tmp_in_addr.s_addr = iph->saddr;
            strcpy(saddr_str, inet_ntoa(tmp_in_addr));
            tmp_in_addr.s_addr = iph->daddr;
            strcpy(daddr_str, inet_ntoa(tmp_in_addr));
            if (iph->protocol == IPPROTO_UDP) {
                if (iph->ihl * 4 + sizeof(*udph) <= r) {//UDP数据包
                    udph = (struct udphdr *)(payload + iph->ihl * 4);
                    if (ntohs(udph->dest) == 10010) {//DUP数据包，端口为10010数据全部修改为字符'h'并以'\n'结尾后返回修改后的数据及ACCEPT裁决
                        if (iph->ihl * 4 + sizeof(*udph) < r && ntohs(iph->tot_len) - iph->ihl * 4 == ntohs(udph->len) && ntohs(udph->len) - sizeof(*udph) > 0) {
                            int offset = iph->ihl * 4 + sizeof(*udph);
                            memset(payload + offset, 'h', ntohs(udph->len) - sizeof(*udph));
                            memset(payload + iph->ihl * 4 + ntohs(udph->len) - 1, '\n', 1);
                            // 由于修改了udp数据包内容，因此需要重新计算udp校验和。
                            // libnetfilter_queue在udp校验和计算时有bug，因此计算校验和代码拷贝到本文件后修复使用。
                            //nfq_udp_compute_checksum_ipv4(udph, iph);
                            udp_compute_checksum_ipv4(udph, iph);
                            printf("ACCEPT & modified  protocol:udp %s:%u -> %s:%u\n", saddr_str, ntohs(udph->source), daddr_str, ntohs(udph->dest));
                        } else {
                            printf("ACCEPT & !modified  protocol:udp %s:%u -> %s:%u\n", saddr_str, ntohs(udph->source), daddr_str, ntohs(udph->dest));
                        }
                        nfq_set_verdict(qh, id, NF_ACCEPT, r, payload);
                    } else if (ntohs(udph->dest) == 10086) {//udp目的端口10086的数据包做DROP裁决
                        printf("DROP    protocol:udp %s:%u -> %s:%u\n", saddr_str, ntohs(udph->source), daddr_str, ntohs(udph->dest));
                        nfq_set_verdict(qh, id, NF_DROP, 0, NULL);
                    } else {
                        printf("ACCEPT  protocol:udp %s:%u -> %s:%u\n", saddr_str, ntohs(udph->source), daddr_str, ntohs(udph->dest));
                        nfq_set_verdict(qh, id, NF_ACCEPT, 0, NULL);
                    }
                } else {//IP数据包，但不是UDP
                    printf("ACCEPT  protocol:udp %s -> %s\n", saddr_str, daddr_str);
                    nfq_set_verdict(qh, id, NF_ACCEPT, 0, NULL);
                }
            } else if (iph->protocol == IPPROTO_TCP) {
                if (iph->ihl * 4 + sizeof(*tcph) <= r) {//TCP数据包，含有TCP头
                    tcph = (struct tcphdr *)(payload + iph->ihl *4);
                    printf("ACCEPT  protocol:tcp %s:%u -> %s:%u\n", saddr_str, ntohs(tcph->source), daddr_str, ntohs(tcph->dest));
                    nfq_set_verdict(qh, id, NF_ACCEPT, 0, NULL);
                } else {//TCP数据包，但是不含有TCP数据包头
                    printf("ACCEPT  protocol:tcp %s -> %s\n", saddr_str, daddr_str);
                    nfq_set_verdict(qh, id, NF_ACCEPT, 0, NULL);
                }
            } else {//IP数据包，但不是TCP
                printf("ACCEPT  protocol:%d %s -> %s\n", iph->protocol, saddr_str, daddr_str);
                nfq_set_verdict(qh, id, NF_ACCEPT, 0, NULL);
            }
        } else {//非IP数据包
            printf("ACCEPT  unknown protocol\n");
            nfq_set_verdict(qh, id, NF_ACCEPT, 0, NULL);
        }
    }

    return 0;
}

int main(int argc, char *argv[])  {
    struct nfq_handle *h;
    struct nfq_q_handle *qh;
    int r;

    char buf[10240];

    unsigned int portid, queue_num;

    if (argc != 2) {
    	printf("Usage: %s [queue_num]\n", argv[0]);
    	exit(EXIT_FAILURE);
    }
    queue_num = atoi(argv[1]);

    h = nfq_open();
    if (h == NULL) {
        perror("nfq_open error");
        goto end;
    }

    if (nfq_unbind_pf(h, AF_INET) != 0) {
        perror("nfq_unbind_pf error");
        goto end;
    }
    if (nfq_bind_pf(h, AF_INET) != 0) {
        perror("nfq_bind_pf error");
        goto end;
    }

    qh = nfq_create_queue(h, queue_num, &cb, NULL);
    if (qh == NULL) {
        perror("nfq_create_queue error");
        goto end;
    }

    if (nfq_set_mode(qh, NFQNL_COPY_PACKET, 0xffff) != 0) {
        perror("nfq_set_mod error");
        goto end;
    }

    while(1) {
        r = recv(nfq_fd(h), buf, sizeof(buf), 0);
        if (r == 0) {
            printf("recv return 0. exit");
            break;
        } else if (r < 0) {
            perror("recv error");
            break;
        } else {
            nfq_handle_packet(h, buf, r);
        }
    }

end:
    if (qh) nfq_destroy_queue(qh);
    if (h) nfq_close(h);
    return 0;
}
```

##### 编译样例    
```
gcc -l netfilter_queue nfqueue_example.c 
```

##### 部署测试环境    
```
//清空所有规则
iptables -F

//先iptables的INPU链挂载一个测试规则
iptables -I INPUT -p icmp -j QUEUE --queue-num 1

//查看规则，可以看出没有任何数据包命中此条规则 
iptables -L -n -v
Chain INPUT (policy ACCEPT 12 packets, 792 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 NFQUEUE    icmp --  *      *       0.0.0.0/0            0.0.0.0/0            NFQUEUE num 1

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 8 packets, 1040 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

##### 运行可执行程序
```
//此时有个中断一直ping这去往INPUT的数据包
root nfqueue$ ./a.out 
Usage: ./a.out [queue_num]
root nfqueue$ ./a.out 1
ACCEPT  protocol:1 192.168.96.1 -> 192.168.96.154
ACCEPT  protocol:1 192.168.96.1 -> 192.168.96.154
ACCEPT  protocol:1 192.168.96.1 -> 192.168.96.154
ACCEPT  protocol:1 192.168.96.1 -> 192.168.96.154

//查看规则，命中4条
iptables -L -n -v
Chain INPUT (policy ACCEPT 97 packets, 6972 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    4   240 NFQUEUE    icmp --  *      *       0.0.0.0/0            0.0.0.0/0            NFQUEUE num 1

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 70 packets, 7492 bytes)
 pkts bytes target     prot opt in     out     source               destination 
```
    

<br> 
转载请注明：[HunterYuan的博客](https://clodfisher.github.io/) » [修改nfqueue支持零拷贝](https://clodfisher.github.io/2019/11/MmapNetlink/)           
