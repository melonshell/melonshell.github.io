---
title: go empty struct和interface
date: 2020-07-01 19:48:32
categories:
- go语言
tags:
- go
---

# struct{}
struct{}类型的占用内存大小为0：
```go
unsafe.Sizeof(struct{}{})
//输出结果：0
```

* 用于控制而非数据信息：chan struct{}
* 实现C++的set：map[string]struct{}

# interface{}
空interface没有方法，故所有类型均实现了interface(包括值接收者和指针接收者);