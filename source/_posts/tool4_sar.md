---
title: sar查看内存、CPU、IO
date: 2019-10-30 08:02:03
categories:
- 工具
- 性能
tags:
- sar
- 内存
- CPU
- IO
---

# 1 内存使用情况统计
-r:输出物理内存和虚拟内存的统计信息
```shell
[root@VM_100_12_centos ~]# sar -r 1 4
Linux 3.10.0-957.1.3.el7.x86_64 (VM_100_12_centos) 	10/30/2019 	_x86_64_	(1 CPU)

08:09:26 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
08:09:27 AM     74960    939932     92.61    137240    553912    345240     34.02    367848    416976       112
08:09:28 AM     72680    942212     92.84    137240    553916    348496     34.34    369440    416984       220
08:09:29 AM     74556    940336     92.65    137240    553920    345240     34.02    367928    416940       296
08:09:30 AM     73696    941196     92.74    137244    553916    346732     34.16    368912    416936       296
Average:        73973    940919     92.71    137241    553916    346427     34.13    368532    416959       231
```
参数：
* kbmemfree：剩余可用内存，单位K  
* kbmemused：已用内存，以K为单位，该值不考虑内核自身所使用的内存  
* %memused：已用内存百分比  
* kbbuffers：free命令中的buufer
* kbcached：free命令中的cache  
* kbcommit：保证当前系统所需要的内存，即为了确保不溢出而需要的内存(RAM+swap)
* %commit：kbcommit与内存总量(包括swap)的百分比
* kbactive：活动内存量(内存最近被使用过，并且不会被回收)，单位K
* kbinact：不活动内存量(最近未被使用的内存，很符合回收策略的内存)，单位K
* kbdirty：等待写入磁盘的内存量，单位K

-B:分页统计
```shell
[root@VM_100_12_centos ~]# sar -B 1 5
Linux 3.10.0-957.1.3.el7.x86_64 (VM_100_12_centos) 	10/30/2019 	_x86_64_	(1 CPU)

08:34:52 AM  pgpgin/s pgpgout/s   fault/s  majflt/s  pgfree/s pgscank/s pgscand/s pgsteal/s    %vmeff
08:34:53 AM      4.08      0.00   2200.00      0.00   1200.00      0.00      0.00      0.00      0.00
08:34:54 AM      0.00      0.00    119.00      0.00     97.00      0.00      0.00      0.00      0.00
08:34:55 AM      0.00      0.00     62.00      0.00     79.00      0.00      0.00      0.00      0.00
08:34:56 AM      0.00     76.00   2285.00      0.00    654.00      0.00      0.00      0.00      0.00
08:34:57 AM      0.00      0.00     76.77      0.00     85.86      0.00      0.00      0.00      0.00
Average:         0.80     15.29    945.27      0.00    420.72      0.00      0.00      0.00      0.00
```
参数：
* pgpgin/s：每秒从磁盘或SWAP置换到内存的字节数(KB)  
* pgpgout/s：每秒从内存置换到磁盘或SWAP的字节数(KB)  
* fault/s：每秒系统产生的缺页数，即主缺页与次缺页之和(major + minor)  
* majflt/s：每秒产生的主缺页数
* pgfree/s：每秒被放入空闲队列的页数
* pgscank/s：每秒被kswapd扫描的页数
* pgscand/s：每秒直接被扫描的页数
* pgsteal/s：每秒从cache中被清除来满足内存需要的页数
* %vmeff：每秒清除的页(pgsteal)占总扫描页(pgscank+pgscand)的百分比

# 2 CPU统计
-u：CPU使用统计
```shell
[root@VM_100_12_centos ~]# sar -u 1 5
Linux 3.10.0-957.1.3.el7.x86_64 (VM_100_12_centos) 	10/30/2019 	_x86_64_	(1 CPU)

01:10:07 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
01:10:08 PM     all      0.00      0.00      0.99      3.96      0.00     95.05
01:10:09 PM     all      1.01      0.00      1.01     14.14      0.00     83.84
01:10:10 PM     all      1.00      0.00      1.00     10.00      0.00     88.00
01:10:11 PM     all      1.01      0.00      0.00     13.13      0.00     85.86
01:10:12 PM     all      1.00      0.00      2.00      5.00      0.00     92.00
Average:        all      0.80      0.00      1.00      9.22      0.00     88.98
```
参数：
* %user：用户级别(application)运行使用的CPU总时间的百分比
* %nice：用户级别，用于nice操作，所占用CPU总时间的百分比
* %system：kernel运行所使用CPU总时间的百分比
* %iowait：用于等待I/O操作占用CPU总时间的百分比
* %idle：CPU空闲时间占用CPU总时间的百分比

1）若%iowait值过高，表示硬盘存在I/O瓶颈
2）若%idle值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量
3）若%idle的值持续低于1，则系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU

nice表示以nice优先级运行的进程用户态时间，即低优先级(正nice值)任务运行的用户态CPU时间，nice值范围[-20, 19]，-20最高优先级，19最低优先级

iowait表示I/O等待时间，大部分人对此有误解，第一个误解是认为CPU此时不能工作。其实CPU在等待I/O时，可以切换到其他就绪任务执行，只是当时刚好没有就绪任务可以执行，准确地讲，iowait是CPU空闲且系统有I/O请求未完成的时间。
另一个误解是iowait升高便认为系统存在I/O瓶颈，同种I/O条件下，如果系统还有其他计算密集型任务，iowait将明显降低。
idle和iowait都说明CPU很空闲，iowait还表明系统有未完成的I/O请求

# 3 IO
## 3.1 磁盘IO
-b：IO统计
```shell
[root@0ec6c1a1049e /home/go/AdsStatConsumer]# sar -b 1 5
Linux 3.10.107-1-tlinux2_kvm_guest-0049 (0ec6c1a1049e)  11/18/2019      _x86_64_        (8 CPU)

01:15:45 PM       tps      rtps      wtps   bread/s   bwrtn/s
01:15:46 PM      0.00      0.00      0.00      0.00      0.00
01:15:47 PM      5.00      5.00      0.00    104.00      0.00
01:15:48 PM      3.00      0.00      3.00      0.00     24.00
01:15:49 PM      0.00      0.00      0.00      0.00      0.00
01:15:50 PM      0.00      0.00      0.00      0.00      0.00
Average:         1.60      1.00      0.60     20.80      4.80
```
参数：
* tps：每秒钟磁盘的I/O请求次数
* rtps：每秒钟磁盘的读请求次数
* wtps：每秒钟磁盘的写请求次数
* bread/s：每秒钟从磁盘读出的数据量，单位block/s
* bwrtn/s：每秒钟向磁盘写入的数据量，单位block/s

## 3.2 网络IO
用法：sar -n ｛keyword | ALL｝，keyword包括：
DEV，EDEV，NFS，NFSD，SOCK，IP，EIP，ICMP，EICMP，TCP，ETCP，UDP，SOCK6，IP6，EIP6，ICMP6，EICMP6和UDP6
```shell
[mqq@9-146-128-151 /usr/local/trpc/bin]$ sar -n DEV 1 2
Linux 3.10.107-1-tlinux2_kvm_guest-0050 (9-146-128-151)         11/21/2019      _x86_64_        (64 CPU)

05:32:46 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
05:32:47 PM      cbr0     29.00     20.00      6.81      7.21      0.00      0.00      0.00
05:32:47 PM veth19f87654     29.00     20.00      7.20      7.21      0.00      0.00      0.00
05:32:47 PM      eth1    686.00    701.00    266.45    125.05      0.00      0.00      0.00
05:32:47 PM        lo     17.00     17.00      9.54      9.54      0.00      0.00      0.00

05:32:47 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
05:32:48 PM      cbr0     26.00     20.00      6.61      7.22      0.00      0.00      0.00
05:32:48 PM veth19f87654     26.00     20.00      6.97      7.22      0.00      0.00      0.00
05:32:48 PM      eth1    726.00    920.00    243.36    565.16      0.00      0.00      0.00
05:32:48 PM        lo     12.00     12.00      5.44      5.44      0.00      0.00      0.00

Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
Average:         cbr0     27.50     20.00      6.71      7.21      0.00      0.00      0.00
Average:    veth19f87654     27.50     20.00      7.09      7.21      0.00      0.00      0.00
Average:         eth1    706.00    810.50    254.90    345.10      0.00      0.00      0.00
Average:           lo     14.50     14.50      7.49      7.49      0.00      0.00      0.00
```
参数：
* rxpck/s：每秒接收数据包的数量
* txpck/s：每秒发送数据包的数量
* rxkb/s：每秒接收的数据大小，单位kb
* txkb/s：每秒发送的数据大小，单位kb
* rxcmp/s：每秒接收的压缩包的数量
* txcmp/s：每秒发送的压缩包的数量
* rxmcst/s：每秒接收的多播数据包数量

# 4 其他常用命令
1) -o选项可以把sar统计信息保存到指定文件，对于保存的日志，可以用-f选项读取：
   sar -n DEV 1 10 -o sar.out
   sar -d 1 10 -f sar.out
   sar -u 1 10 -f sar.out

2) 统计网络设备通信失败信息：
   sar -n EDEV 1 10

3) 统计socket连接信息：
   sar -n SOCK 1 10

4) TCP连接统计：
   sar -n TCP 1 10
参数：
   active/s：主动连接次数。The number of times TCP connections have made a direct transition to  the  SYN-SENT state from the CLOSED state per second [tcpActiveOpens].
   passive/s：被动连接次数。The number of times TCP connections have made a direct transition to the SYN-RCVD state from the LISTEN state per second [tcpPassiveOpens].
   iseg/s：当前已建立的连接每秒接收的段数量。The total number of segments received per second, including those  received  in error [tcpInSegs]. This count includes segments received on currently established connections.
   oseg/s：当前连接每秒发送的段数量，但不包括仅包含重传字节的段。The total number of segments sent per second, including those on current connections but excluding those containing only retransmitted octets [tcpOutSegs].

1) IP数据报统计：
   sar -n IP 1 10

2) IP错误统计：
   sar -n EIP 1 10