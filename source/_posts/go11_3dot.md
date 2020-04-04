---
title: go语言之三个点用法
date: 2020-03-27 15:42:15
categories:
- go语言
tags:
- go
- 三个点
---

# 1 函数可变数量参数
示例：
```go
package main

import "fmt"

func main() {
    //multiParam 可以接受可变数量的参数
    multiParam("jerry", "herry")
    names := []string{"jerry", "herry"}
    multiParam(names...)
}
func multiParam(args ...string) {
    //接受的参数放在args数组中
    for _, e := range args {
        fmt.Println(e)
    }
}
```
**可变参数是函数最右边的参数**，普通参数放在左侧，如：
```go
package main

import "fmt"

func main() {
   //multiParam 可以接受可变数量的参数
   multiParam("jerry", 1)
   multiParam("php", 1, 2)
}
func multiParam(name string, args ...int) {
   fmt.Println(name)
   //参数放在args切片中
   for _, e := range args {
      fmt.Println(e)
   }
}
```

# 2 展开slice
如上例中将names打散展开，还有种情况就是通过append合并两个slice:
```go
stooges := []string{"Moe", "Larry", "Curly"}
lang := []string{"php", "golang", "java"}
stooges = append(stooges, lang...)
fmt.Println(stooges)
```

# 3 数组元素数量
如果忽略数组[]中的数字不设置大小，...指定的长度等于数组中元素的数量，Go语言会根据元素的个数设置数组的大小。
```go
stooges := [...]string{"Moe", "Larry", "Curly"} //等价于stooges := [3]string{"Moe", "Larry", "Curly"}
arr := [...]int{1, 2, 3}
fmt.Println(len(stooges))
fmt.Println(len(arr))
```

# 4 go命令
go描述软件包列表时，命令使用三个点作为通配符。
此命令测试当前目录及其子目录的所有软件包：
```go
go test ./...
```