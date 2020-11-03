---
title: go map并发读写问题
date: 2020-11-01 10:24:41
categories:
- 问题解决
---

# 1 问题描述
用trpc重构看点精排后，灰度放量1%，测试用例一直告警，排查染色日志发现ip connection refused，登录粗排容器，发现进程刚重启，在日志目录发现有panic：map并发读写；

# 2 代码分析
接口请求结构体req *RoughRankAdReq中有成员AdsUserInfo，AdsUserInfo中扩展字段MExtendUserInfo map[string]string；

在召回游戏、合约、自营广告时并行起了多个goroutine，每个goroutine都调用自己RecallData的召回函数，但这些RecallData都有RoughRankAdReq成员，全部由请求结构体req *RoughRankAdReq赋值浅Copy；

在每个goroutine中：
* 创建一个RecallAdReq结构体变量RecallReq；
* 构造新的RoughRankAdReq结构体变量newReq，AdsUserInfo成员由RecallData的AdsUserInfo赋值浅Copy；
* RecallAdReq的成员AdsUserInfo由newReq的AdsUserInfo赋值浅Copy；

问题就出在对于IEG召回源：
```go
RecallReq.StAdsUserInfo.MExtendUserInfo["sysid"] = "kd"
```
导致在map MExtendUserInfo可能会并发读写，panic。

# 3 map结构
map的底层结构为：
```go
// A header for a Go map.
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}
```
go结构体之间相互赋值，本质上是浅Copy，Copy了结构体内部的所有字段。因此上面描述的代码各结构体的AdsUserInfo的MExtendUserInfo map成员，底层指向的是同一块内存，故panic。