---
title: go语言之channel
date: 2019-12-07 15:34:04
categories:
- go语言
tags:
- go
- channel
---

# 1 channel定义
channel是Go中的一个核心类型，是Goroutine之间的通信方式，操作符：<-；channel有三种类型定义(chan|chan <-|<-chan)，<-代表channel方向，如没有指定方向，则channel是双向的，既可以接收数据也可以发送数据：
```go
chan T   //可以接收和发送T类型数据
chan<- float64  //只可以发送float64类型的数据
<-chan int //只可以用来接收int类型的数据
```
<-总是优先和最左边的类型结合：
```go
chan<-chan int  //等价chan <- (chan int)
chan<- <-chan int // 等价 chan <- (<-chan int)
<-chan<-chan int  // 等价 <- chan (<-chan int)
```
# 2 channel创建
channel须先创建再使用：
```go
ch := make(chan int)
```
创建带Buffer的channel：
```go
ch := make(chan int, 100)
```
如果没有设置buffer数量，或者buffer为0，则channel没有缓存，此时只有sender和receiver都准备好了通信才会发生，否则阻塞；如果设置了buffer，则buffer满后send才会阻塞，buffer空后receive才会阻塞；

# 3 channel关闭
close方法可以关闭channel；
检查channel是否已经关闭的方法：
v, ok := <-ch
close调用之后继续从channel读取，如果buffer中还有数据未读完，则v返回数据，ok为true；如果buffer中数据已读完，则v返回零值，ok为false；

操作    | nil channel | closed channel | not-closed non-nil channel
:-:     |    :-:      |      :-:       |      :-:
close   |  panic      |  panic         | 成功close
写ch <- | 一直阻塞     |  panic         | 阻塞或成功写入数据
读<- ch | 一直阻塞     | 读取对应类型零值 |阻塞或成功读取数据

关闭channel会产生一个广播机制，所有从channel读取消息的goroutine都会收到消息
For a channel c, the built-in function close(c) records that no more values will be sent on the channel. It is an error if c is a receive-only channel. Sending to or closing a closed channel causes a run-time panic. Closing the nil channel also causes a run-time panic. After calling close, and after any previously sent values have been received, receive operations will return the zero value for the channel's type without blocking. The multi-valued receive operation returns a received value along with an indication of whether the channel is closed.

# 4 channel特性
1) 线程安全；
2) 可以作为一个FIFO队列，接收和发送的数据顺序一致；
3) 对nil channel读写会永久block；
4) 写closed channel会panic；
5) 从closed channel读会立即返回，接收完buffer数据后返回零值；

# 5 range遍历
```go
func main() {
	len := 10
	ch := make(chan int, 10)
	for i:=1; i<=len; i++{
		ch <- i
	}

	go func(ch chan int) {
		time.Sleep(time.Duration(5) * time.Second)
		fmt.Println("close channel")
		close(ch)
	}(ch)

	for v := range ch {
		fmt.Println("chan val:", v)
		time.Sleep(time.Duration(1) * time.Second)
	}

	fmt.Println("main goroutine end")
}
```
range产生的迭代值为channel中发送的数据，它会一直迭代直到channel关闭且没有数据可接收时(buffer中有未读数据，则会继续读取，直到读完)跳出循环；

# 6 select
参见文章go语言之select

# 7 timer和ticker
timer和ticker是两个关于时间的channel；
timer是一个定时器，它提供一个channel，在等待时间过期后，将当前时间发送到此channel，下面例子会阻塞2秒，直到timer过期：
```go
timer1 := time.NewTimer(time.Second * 2)
<-timer1.C
fmt.Println("Timer 1 expired")
```
如果只是单纯地等待，可以使用time.Sleep；
timer.Stop停止计时器，Stop并不close channel：
```go
func main() {
	t := time.NewTimer(time.Second * 10)
	go func() {
		<-t.C
		fmt.Println("t expired")
	}()

	s := t.Stop()
	if s {
		fmt.Println("t stopped")
	}
	time.Sleep(time.Second * 60)
}
```
ticker是一个定时触发的计时器，它会以一个时间间隔向channel发送一个事件(当前时间)，channel的接收者可以以固定的时间间隔读取事件，下面例子ticker每秒钟触发一次：
```go
func main() {
	ticker := time.NewTicker(time.Second * 1)
	go func() {
		for t := range ticker.C {
			fmt.Println("Tick at ", t)
		}
		fmt.Println("ticker stop, exit for loop")
	}()
	time.Sleep(time.Second * 60)
	fmt.Println("ticker stop")
	ticker.Stop()
	time.Sleep(time.Hour * 1)
}
```
ticker也可以通过Stop方法停止，停止后接收者将不再会从channel中接收数据，Stop并不close channel。

# 8 同步
channel可以用于goroutine间同步
```go
func worker(done chan bool) {
	time.Sleep(time.Second * 5)
	fmt.Println("send: done <- true")
	done <- true
}

func main() {
	done := make(chan bool, 1)
	go worker(done)
	<-done
	fmt.Println("done, exit")
}
```