---
title: dmesg命令
date: 2020-12-27 23:27:12
categories:
- 工具
tags:
- dmesg
---

# 1 介绍
显示内核信息。
dmesg is used to examine or control the kernel ring buffer.
dmesg打印和控制内核的输出信息，这些信息保存在ring buffer中。

# 2 查看OOM
AdsNewStatServer进程重启，但无panic文件
```shell
dmesg |grep -E 'kill|oom|out of memory'
--------------------------------------
[792059.380069] [ pid ]   uid  tgid total_vm      rss nr_ptes nr_pmds swapents oom_score_adj name
[792059.380384] Memory cgroup out of memory: Kill process 2178895 (AdsNewStatServe) score 1940 or sacrifice child
[792087.307426] ps invoked oom-killer: gfp_mask=0x14000c0(GFP_KERNEL), nodemask=(null),  order=0, oom_score_adj=979
[792087.315865]  oom_kill_process+0x22c/0x440
[792087.318529]  mem_cgroup_oom_synchronize+0x2f9/0x320
[792087.319517]  ? mem_cgroup_oom_unregister_event+0xb0/0xb0
[792087.336920] Task in /kubepods/burstable/pod536ef603-3c88-11eb-9ad9-e60de00e2576/b6b6b95624483b2e86fd1ecbc3de20a852ecb600009e06fa9fe969ff81122654 killed as a result of limit of /kubepods/burstable/pod536ef603-3c88-11eb-9ad9-e60de00e2576
```