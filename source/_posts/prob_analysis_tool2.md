---
title: 性能分析命令
date: 2019-10-10 23:40:47
categories:
- 性能分析
tags:
- 性能分析
---

# 1 查看TCP连接的各状态的数量
```shell
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

output:
TIME_WAIT 9822
CLOSE_WAIT 129
SYN_SENT 25
FIN_WAIT1 2468
FIN_WAIT2 748
ESTABLISHED 19344
SYN_RECV 942
CLOSING 204
LAST_ACK 4552
```