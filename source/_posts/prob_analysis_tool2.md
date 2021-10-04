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

# 2 ulimit
ulimit是shell层对进程的限制。
ulimit限制的是当前shell进程，及其派生的子进程。

# 3 rlimit
rlimit为程序中限制单进程的资源使用，结构如下：
```c
struct rlimit {
    rlim_t rlim_cur;    //Soft limit
    rlim_t rlim_max;    //Hard limit (ceiling for rlim_cur)
};
```
setrlimit/getrlimit操作rlimit结构。

# 4 查看进程占用的句柄数
lsof -n | awk '{print $2}' | sort | uniq -c | sort -nr | more
