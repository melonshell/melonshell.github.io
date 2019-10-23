---
title: go之nil
date: 2019-10-13 21:43:24
categories:
- go语言
tags:
- go
- nil
---

# 1 nil介绍
## 1.1 从错误处理说起
```go
if err != nil ｛
	//do sth
｝
```
相信大家对这段代码已经很熟悉了：如果err != nil，表明出了错误；如果 err == nil，说明执行正常。Go语言中，如果你声明了一个变量但是没有赋值，那么此变量会有个对应类型的默认零值，每种类型对应的零值如下：  
```go
bool         false
number       0
string       ""
pointer      nil
slice        nil
map          nil
channel      nil
function     nil
interface    nil         
```
举个栗子，当你定义了一个struct：  
```go
type Person struct {
	age int
    name string
    friends []Person
}
var p Person //Person{0, "", nil}
```
变量p只声明但没有赋值，所有p的所有字段都是对应的零值。**nil到底是什么？**，Go文档提到，nil是预定义的标识符，代表指针、通道、函数、接口、映射或切片的零值，即预定义好的一个标量：  
```go
type Type int
var nil Type
```
nil并不是Go的关键字，你甚至可以改变nil的值：
```go
var nil = errors.New("hi")
```
当然不建议这么做。  

# 2 nil的用法
## 2.1 Pointer
```go
var p *int
p == nil //true
*p       //panic:invalid memory address or nil pointer dereference
```
对nil指针解引用会panic，nil指针有什么用了？先看一个计算二叉树和的例子：  
```go
type tree struct {
	v int
	l *tree
	r *tree
}

//first solution
func (t *tree) Sum() int {
	sum := t.v
	if t.l != nil {
		sum += t.l.Sum()
	}

	if t.r != nil {
		sum += t.r.Sum()
	}

	return sum
}
```
上述代码有两个问题，一个是t为nil的时候回panic：  
```go
var t *tree
sum := t.Sum() //panic : invalid memory address or nil pointer dereference
```
另一个问题是代码重复：
```go
if v != nil {
	v.m()
}
```
怎么解决上述问题了？先看一个指针接收器的例子：  
```go
type person struct {}
func sayHi(p *person) { fmt.Println("hi") }
func (p *person) sayHi() { fmt.Println("hi") }
var p *person
p.sayHi() // hi
```
对**指针对象的方法，即使指针的值为nil也可以调用；值对象的方法，nil调用会panic**，基于此，我们对刚才二叉树和的例子重新改造一下：  
```go
func(t *tree) Sum() int {
  if t == nil {
    return 0
  }
  return t.v + t.l.Sum() + t.r.Sum()
}
```
只需在方法开始判断一下是否为nil，无需重复判断。对于打印二叉树的值或者查找二叉树的思路是类似的：  
```go
func (t* tree) Print() string {
	if t == nil {
		return ""
	}
	return fmt.Sprint("%d ", t.v) + t.l.Print() + t.r.Print()
}

func (t* tree) Find(v int) bool {
	if t == nil {
		return false
	}

	return t.v == v || t.l.Find(v) || t.r.Find(v)
}
```
所以如果不是必要的话，不用NewX()去初始化值，而是使用它们的默认值。另外，对于**C++的空指针也可以调用成员方法，前提是成员方法没有引用成员变量**。  

## 2.2 slice
```go
var s []slice   //nil
len(s)          //0
cap(s)          //0
for range s     //iterates zero times
s[i]            //panic: index out of range
```
一个nil的slice，除了不能索引外，其他的操作均是可以的，**当用append函数填值的时候，slice会自动扩充**，nil的slice底层结构是怎样的？根据官方文档，slice有三个元素：长度、容量、指向数组的指针：  
  ![nil slice底层结构](/pic/go4201910140000.png)
当有元素的时候：  
  ![非nil slice底层结构](/pic/go4201910140002.png)


## 2.3 map
Go语言中，map/function/channel都是特殊的指针，指向各自特定的实现。  
```go
// nil maps
var m map[t]u
len(m)  // 0
for range m // iterates zero times
v, ok := m[i] // zero(v), false
m[i] = x // panic: assignment to entry in nil map
```
对于nil map，可以简单把它看成一个只读的map，不能进行写操作，否则会panic。那么nil map有什么用了？请看如下例子：  
```go
func NewGet(url string, headers map[string]string) (*http.Request, error) {
  req, err := http.NewRequest(http.MethodGet, url, nil)
  if err != nil {
    return nil, err
  }

  for k, v := range headers {
    req.Header.Set(k, v)
  }
  return req, nil
}
```
对于NetGet来说，我们需要传入一个类型为map的参数，且这个函数只是对参数进行读取，如传入一个非空值：  
```go
NewGet("http://www.google.com", map[string]string{"USER_AGENT":"golang/gopher",},)
```
或者这样传：  
```go
NewGet("http://google.com", map[string]string{})
```
前面提到，map的零值是nil，所以header为空的时候，我们可以直接传入一个nil:  
```go
NewGet("http://google.com", nil)
```
这样就简洁很多，把nil map作为一个只读的空map吧。  

## 2.4 channel
```go
// nil channels
var c chan t 
<- c      // blocks forever
c <- x    // blocks forever
close(c)  // panic: close of nil channel
```
channel关闭后，仍然可以从channel中读取剩余的数据，直到数据全部读取完成，如果还继续读数据，得到的是零值。向已关闭的channel发送数据，会引起panic。
```go
c := make(chan int, 10)
close(c)
v, ok := <-c
fmt.Printf("%d, %t", v, ok) //0, false
```
channel关闭后，**select一直可读**:  
```go
c := make(chan int, 3)
go func() {
	c <- 3
	c <- 4
	c <- 5

	close(c)
	fmt.Println("chan is closed")
	time.Sleep(5 * time.Second)
}()

time.Sleep(1 * time.Second)
fmt.Println("start read")

for i := 0; i < 10; i++ {
	select {
		case v, ok := <-c:
			fmt.Printf("i:%d, v:%d, ok:%t\n", i, v, ok)
	}
}

Output:
chan is closed
start read
i:0, v:3, ok:true
i:1, v:4, ok:true
i:2, v:5, ok:true
i:3, v:0, ok:false
i:4, v:0, ok:false
i:5, v:0, ok:false
i:6, v:0, ok:false
i:7, v:0, ok:false
i:8, v:0, ok:false
i:9, v:0, ok:false
```

关闭nil的channel会导致panic，假如有两个channel负责输入，一个channel负责汇总，代码如下：  
```go
func merge(out chan<- int, a, b <-chan int) {
  for {
    select {
      case v := <-a:
        out <- v
      case v := <- b:
        out <- v
    }
  }
}
```
如果外部调用中关闭了a或者b，那么就会不断地从a或者b中读出0，这和我们想要的不一样，我们希望关闭a和b就停止汇总，修改一下代码:
```go
func merge(out chan<- int, a, b <-chan int) {
  for a != nil || b != nil {
    select {
      case v, ok := <-a:
          if !ok {
            a = nil
            fmt.Println("a is nil")
            continue
          }
          out <- v
      case v, ok := <-b:
          if !ok {
            b = nil
            fmt.Println("b is nil")
            continue
          }
          out <- v
    }
  }
  fmt.Println("close out")
  close(out)
}
```
在知道channel关闭后，将channel的值设置为nil，这就相当于将select case子句停用了，**因为nil的channel是永远阻塞的**。  

## 2.5 interface
interface{}类型**的底层实现包含两个指针：类型和值，类似于(*Type, *Value)，只有类型和值都为nil的时候，才等于nil**，看如下代码：  
```go
func main() {
    var val interface{} = nil
    if val == nil {
        fmt.Println("val is nil")
    } else {
        fmt.Println("val is not nil")
    }
}
```
变量val是interface类型，故底层结构为(*Type, *Value)。nil是untyped(无类型)，将nil赋值给val，val实际上存储的是(nil, nil)，所以val == nil为true。  
```go
import (
   "fmt"
   "reflect"
)

type ErrorImpl struct {}
func (e *ErrorImpl) Error() string {
   return ""
}

var ei *ErrorImpl
var e error

func ErrorImplFun() error {
   return ei
}

func main() {
   f := ErrorImplFun()
   fmt.Println(f == nil)
   fmt.Println(reflect.TypeOf(f), " ", reflect.ValueOf(f))
}
输出结果如下：
false
*main.ErrorImpl   (*main.ErrorImpl, nil)
```