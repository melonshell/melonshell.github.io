---
title: go语言之sync.Once
date: 2020-05-04 19:52:54
categories:
- go语言
tags:
- go
- sync.Once
---

# 1 sync.Once说明
sync.Once用于执行只需执行一次的函数功能。

# 2 例子
```go
func main() {
	o sync.Once

	for {
		o.Do(func() {
			fmt.Println("sync once")
		})

		time.Sleep(1 * time.Second)
		fmt.Println("not once")
	}
}
```
# 3 源码数据结构
```go
type Once struct {
	m    Mutex
	done uint32
}
```
done uint32，标记位，若已执行过，置为1；
m Mutex，锁

# 4 源码实现
```go
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 1 {
		return
	}
	// Slow-path.
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

* if atomic.LoadUint32(&o.done) == 1是if o.done == 1的线程安全写法，如果o.done等于0才会执行后续逻辑，atomic.LoadUint32是原子变量取值函数。
* defer atomic.StoreUint32(&o.done, 1)是o.done = 1的线程安全写法，当然取值的时候也要用atomic.LoadUint32才有效果。如果写的时候用atomic.StoreUint32，取值的时候直接用if o.done == 1，则无法保证原子性。

# 5 错误实现
Note: Here is an incorrect implementation of Do:
```go
if atomic.CompareAndSwapUint32(&o.done, 0, 1) {
    f()
}
```
Do guarantees that when it returns, f has finished. This implementation would not implement that guarantee:
given two simultaneous calls, the winner of the cas would call f, and the second would return immediately, without waiting for the first's call to f to complete. This is why the slow-path falls back to a mutex, and why the atomic.StoreUint32 must be delayed until after f returns. 此时如果第二个直接返回的协程立即访问到尚未初始化完成的变量(第一个协程调用f可能还未初始化完成)，则会出问题。
