---
title: watch和dmesg
date: 2019-10-10 23:40:47
categories:
- 性能分析
tags:
- 性能
- 工具
---

# 1 watch
**监测命令的运行结果**
-d, --differences[=cumulative]   highlight changes between updates (cumulative means highlighting is cumulative)
-n, --interval=<seconds>, seconds to wait between updates
**每隔1秒显示网络连接数**
watch -n 1 -d netstat -ant
watch -n 1 ss -s
**实时查看client建立的连接数**
watch -n 1 -d 'netstat -an | egrep "192.168.25.100"| wc -l'

# 2 dmesg
dmesg command is used to display the kernel related messages on Unix like systems. dmesg command retrieve its data by reading the kernel ring buffer.
```shell
dmsg

output:
[2195497.136762] nf_conntrack: table full, dropping packet
[2195497.136800] nf_conntrack: table full, dropping packet
[2195497.140649] nf_conntrack: table full, dropping packet
[2195497.140741] nf_conntrack: table full, dropping packet
[2195497.140766] nf_conntrack: table full, dropping packet
[2195497.424427] nf_conntrack: table full, dropping packet
[2195497.424458] nf_conntrack: table full, dropping packet
[2195497.424497] nf_conntrack: table full, dropping packet
```
**Display messages related to RAM, Hard disk, USB drives and Serial ports**
```shell
dmesg | grep -i memory
dmesg | grep -i dma
dmesg | grep -i usb
dmesg | grep -i tty
```
**Read and Clear dmesg logs using (-c) option**
```shell
dmesg -c
```