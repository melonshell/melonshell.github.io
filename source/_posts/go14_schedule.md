---
title: go调度
date: 2020-06-06 22:31:15
categories:
- go语言
tags:
- go
---

# 系统调用
sysmon线程会监控所有执行syscall的线程M，一旦超过某个时间阈值，就将该M与对应的P剥离；

# cgo
cgo的m，cgo的调用实际上使用了lockedm和syscall；
g陷入cgocall: lockedm加上syscall的处理逻辑；

m陷入cgo和syscall时，p的状态会被设置为_Psyscall，sysmon周期性地检查并retake p，如果发现p处于这个状态且超过10ms就会强制性收回p，m在cgo和syscall返回后会重新尝试拿p，进入调度循环；
