---
title: defer深入
date: 2021-02-11 11:02:43
categories:
- go语言
tags:
- go
- defer
- 函数return
---

defer表达式必须是函数/方法调用。

# 1 deferred函数执行顺序
如果有多个defer表达式，调用顺序类似于栈，越后面的defer表达式越先被调用，后进先出。
deferred functions are invoked immediately before the surrounding function returns, in the reverse order they were deferred.
```go
    func main() {
        i := 0
        defer func() {
            i++
            fmt.Println("func 1:", i)
        }()

        defer func() {
            i++
            fmt.Println("func 2:", i)
        }()
    }
}
```
结果如下：
func 2: 1
func 1: 2

# 2 defer语句函数实参
Each time a "defer" statement executes, the function value and parameters to the call are evaluated as usual and saved anew but the actual function is not invoked.
defer语句执行时，函数参数会保存当前时刻变量值的副本，但函数不会被调用。
```go
    func main() {
        i := 10
        defer func(i int) {
            fmt.Println("defer i:", i)
        }(i)

        i++
        fmt.Println("main i:", i)
    }
```
结果如下：
main i: 11
defer i: 10

# 3 deferred函数执行时机
if the surrounding function returns through an explicit return statement, deferred functions are executed after any result parameters are set by that return statement but before the function returns to its caller.
**return xxx这一条语句并不是一条原子指令:** 先给返回值赋值，再调用defer表达式，最后返回到主调函数。
对于命名返回值的情况，需要注意：
例1:
```go
    func f() (result int) {
        defer func() {
            result++
        }()
        return 0
    }
    //output: 1
```

例2:
```go
    func f() (r int) {
        t := 5
        defer func() {
        t = t + 5
        }()
        return t
    }
    //output: 5
```

例3:
```go
    func f() (r int) {
        defer func(r int) {
            fmt.Println("f():", r)
            r = r + 5
        }(r)
        return 1
    }
    //output: f(): 0
    //        1
```

例4:
```go
   func f() int {
        r := 0
        defer func(r int) {
            r = r + 5
        }(r)
        return r
    }
    //output: 0
```

例5:
```go
    func main() {
        fmt.Printf("x: %d\n", Parser())
    }
    func Parser() (x int) {
        x = 1
        defer func() {
            x = 2
        }()
        return x
    }
    //output: 2
```
**返回值赋值：返回值变量已命名，即为x本身，无需赋值**

对于匿名返回值：
例6:
```go
    func main() {
        fmt.Printf("x: %d\n", Parser())
    }
    func Parser() int {
        x := 1
        defer func() {
            x = 2
        }()
        return x
    }
    //output: 1
```
**返回值赋值：返回值没有命名，会将x赋值给匿名变量**。

//x赋值到r0寄存器 -- 匿名返回值的赋值操作
0x005b 00091 (/Users/mapleyin/test/t1.go:15)	MOVQ	"".x+16(SP), AX
0x0060 00096 (/Users/mapleyin/test/t1.go:15)	MOVQ	AX, "".~r0+56(SP)

汇编分析参考文章：http://404-notfound.cn/golang-return-defer/
