---
title: go for range遍历
date: 2020-11-15 10:40:11
categories:
- go语言
tags:
- go
---

# range表达式

The range expression x is evaluated once before beginning the loop, with one exception: if at most one iteration variable is present and len(x) is constant, the range expression is not evaluated.

备注： 至多只有一个迭代变量，且len(x)是常量，见go说明文档<https://golang.org/ref/spec#Length_and_capacity>

The expression len(s) is constant if s is a string constant. The expressions len(s) and cap(s) are constants if the type of s is an array or pointer to an array and the expression s does not contain channel receives or (non-constant) function calls; in this case s is not evaluated. Otherwise, invocations of len and cap are not constant and s is evaluated.

# 表达式左边
Function calls on the left are evaluated once per iteration. For each iteration, iteration values are produced as follows if the respective iteration variables are present:
<https://golang.org/ref/spec>  **For statements with range clause**
注意string的遍历， 例子：
```go
s := "he你好"
for idx, val := range s {
   fmt.Println("idx:", idx, ",val:", string(val))
}

output:
idx: 0, val: h
idx: 1, val: e
idx: 2, val: 你
idx: 5, val: 好
```