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
-u:CPU使用统计  
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



