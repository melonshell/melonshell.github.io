---
title: go数组作为函数形参
date: 2021-02-10 21:02:43
categories:
- go语言
tags:
- go
- 数组
- 函数形参
---

# 1 数组作为函数形参
Go中数组作为函数的形参，实际传递的是数组的拷贝，而非数组的指针（区别于C/C++）。
```go
    func sum(a [4]int){
        a[0]=100
        fmt.Printf("a:%p\n", &a[0])
    }

    func main(){
        a := [4]int{1,1,1,1}
        sum(a)
        fmt.Println(a[0])
        fmt.Printf("a:%p\n", &a[0])
    }
```
结果如下：
a:0xc000014040
1
a:0xc000014020

# 2 数组的地址
如果需要传递数组本身而非拷贝，可传递数组的地址：
```go
    func sum(a *[4]int){
        a[0]=100
        fmt.Printf("a:%p\n", &a[0])
    }

    func main(){
        a := [4]int{1,1,1,1}
        sum(&a)
        fmt.Println(a[0])
        fmt.Printf("a:%p\n", &a[0])
    }
```
结果如下：
a:0xc000014020
100
a:0xc000014020

# 3 传递切片