---
title: go语言之select
date: 2019-12-05 16:45:42
categories:
- go语言
tags:
- go
- select
---

# 1 官方描述
A "select" statements chooses which of a set of possible send or receive operations will proceed. It looks similar to a "switch" statement but with the cases all referring to communication operatinos.

# 2 基本用法
select用来监听和channel有关的IO操作，基本用法:
```go
select {
	case <- chan1:
	// 如果chan1成功读到数据，则执行该case处理语句
	case chan2 <- 1:
	// 如果成功向chan2写入数据，则执行该case处理语句
	default:
	// 如果上面都没有成功，则执行default处理
}
```
## 2.1 执行流程
如果有一个或多个IO操作可以完成，Go运行时会随机选择一个执行；
如果没有IO操作可以完成，若有default分支，则执行default分支语句；若没有default分支，则select语句将一直阻塞，直到至少一个IO操作可以执行。

## 2.2 计算规则
所有channel表达式都会被求值，所有被发送的表达式都会被求值，求值顺序：自上而下，从左到右
```go
func c1() <-chan int {
	fmt.Println("c1 channel receive##1")
	return make(<-chan int)
}

func c2() chan <- int {
	fmt.Println("c2 channel send##2")
	return make(chan <- int)
}

func getNum() int {
	fmt.Println("getNum##3")
	return 9
}

func main() {
	select {
	case <-c1():
		fmt.Println("c1 return")
	case c2() <- getNum():
		fmt.Println("c2 return")
	default:
		fmt.Println("this is default")
	}
}

执行结果：
c1 channel receive##1
c2 channel send##2
getNum##3
this is default
```

## 2.3 结束select
```go
func c1() <-chan int {
	fmt.Println("c1 channel receive##1")
	c := make(chan int, 1)
	c <- 1
	return c
}

func c2() chan <- int {
	fmt.Println("c2 channel send##2")
	return make(chan <- int, 1)
}

func getNum() int {
	fmt.Println("getNum##3")
	return 9
}

func main() {
	select {
	case <-c1():
		fmt.Println("c1 return")
		break
		fmt.Println("c1 return 2")
	case c2() <- getNum():
		fmt.Println("c2 return")
		fmt.Println("c2 return 2")
	}
}

输出结果：
c1 channel receive##1
c2 channel send##2
getNum##3
c2 return
c2 return 2
-------------
c1 receive##1
c2 send##2
getNum##3
c1 return
```
可以看到所有channel表达式都会被求值，所有被发送表达式都会被求值；在执行case <-c1()时，由于break关键字结束了select，只打印了c1 return。

## 2.4 nil channel和select
```go
func main() {
	var c chan int

	select {
	case <- c:
		fmt.Println("nil receive")
	case c <- 1:
		fmt.Println("nil send")
	}
}
```
对nil channel读写永久阻塞