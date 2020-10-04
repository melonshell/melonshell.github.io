---
title: go变量分配
date: 2020-10-04 11:31:07
categories:
- go语言
tags:
- go
---

# 局部变量分配
1) GO编译器尽可能将变量分配在栈上
2) 以下两种情况，变量会分配在堆上：
* 如果一个变量被取地址(has its address taken)，并且被逃逸分析(escape analysis)识别为"逃逸到堆"(escapes to heap)
* 如果变量很大(very large)

# 逃逸分析
以下面例子来说，github bradfitz iter源码：
```go
func N(n int) []struct{} {
	return make([]struct{}, n)  //It does not cause any allocations
}
```
调用如下：
```go
package main

import "github.com/bradfitz/iter"

func main() {
	for range iter.N(4) {}
}
```
逃逸分析： **go build -gcflags='-m -m' escape.go**
```shell
$ go build -gcflags='-m -m' escape.go
./escape.go:5:6: cannot inline main: unhandled op RANGE
./escape.go:6:18: inlining call to iter.N func(int) []struct {} { return make([]struct {}, iter.n) }
./escape.go:6:18: make([]struct {}, iter.n) escapes to heap:
./escape.go:6:18:   flow: {heap} = &{storage for make([]struct {}, iter.n)}:
./escape.go:6:18:     from make([]struct {}, iter.n) (non-constant size) at ./escape.go:6:18
./escape.go:6:18: make([]struct {}, iter.n) escapes to heap
```
从"make([]struct {}, iter.n) escapes to heap"可以推断：make([]struct {}, iter.n)被分配在堆上。

# 内存分配器追踪
除了逃逸分析，Go还提供了一种叫内存分配器追踪（Memory Allocator Trace）的方法，用于细粒度地分析由程序引发的所有堆分配和释放操作：
```shell
$ GODEBUG=allocfreetrace=1 go run escape.go 2>&1 | grep -C 10 t.log
```
因为进行内存分配器追踪时，很多由runtime引发的分配信息也会被打印出来，所以我们用grep进行过滤，只显示由用户代码(user code)引发的分配信息，然而这里的输出结果为空，表明make([]struct {}, iter.n)没有引发任何堆分配。

内存分配器追踪的结论与逃逸分析的结论截然相反，那到底哪个结论正确？
runtime.makeslice的源码：
```go
func makeslice(et *_type, len, cap int) slice {
	...
	p := mallocgc(et.size*uintptr(cap), et, true)
	return slice{p, len, cap}
}
```
mallocgc源码：
```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
	...
	if size == 0 {
		return unsafe.Pointer(&zerobase)
	}
	...
	if debug.allocfreetrace != 0 {
		tracealloc(x, size, typ)
	}
	...
}
```
原因是mallocgc函数中，**因为空结构体的size为0，所以mallocgc并没有实际进行堆分配，也没有执行tracealloc，故内存分配器追踪不会采集到相关的分配信息**，makeslice函数中，切片slice本身是以结构体的形式返回，只会被分配在栈上。

# 总结
* make([]struct{}, n)只会被分配在栈上，而不会被分配在堆上；
* 注释It does not cause any allocations是对的，意思是"不会引发堆分配"；
* 逃逸分析识别出escapes to heap，并不一定就是堆分配，也可能是栈分配；
* 进行内存分配器追踪时，如果采集不到堆分配信息，那一定只有栈分配；

**如何确定一个变量被分配在哪里？**
1) 先对代码作逃逸分析:
* 如果该变量被识别为escapes to heap，那么它大概率是被分配在堆上；
* 如果该变量被识别为does not escape，或者没有与之相关的分析结果，那么它一定是被分配在栈上；

2) 如果对escapes to heap心存疑惑，就对代码做**内存分配器追踪**:
* 如果有采集到与该变量相关的分配信息，那么它一定是被分配在堆上；
* 否则，该变量一定是被分配在栈上；

3) 此外，如果想知道Go编译器是如何将变量分配在堆或者栈上，可以分析Go汇编以及runtime源码：
**go tool compile -I $GOPATH/pkg/mod -S XXX.go**
```shell
$ go tool compile -I $GOPATH/pkg/mod -S escape.go
...
0x001d 00029 (./escape.go:6)        LEAQ    type.struct {}(SB), AX
0x0024 00036 (./escape.go:6)        PCDATA  $2, $0
0x0024 00036 (./escape.go:6)        MOVQ    AX, (SP)
0x0028 00040 (./escape.go:6)        MOVQ    $4, 8(SP)
0x0031 00049 (./escape.go:6)        MOVQ    $4, 16(SP)
0x003a 00058 (./escape.go:6)        CALL    runtime.makeslice(SB)
...
```
