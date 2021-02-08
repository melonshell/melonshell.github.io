---
title: go方法集
date: 2021-02-08 00:35:11
categories:
- go语言
tags:
- go
- 方法集
- 接口
---

# 1 方法集的作用
The method set of a type determines the interfaces that the type implements and the methods that can be called using a receiver of that type.

方法集的概念是对接口来说的，⽤实例value和pointer调⽤⽅法(含匿名字段)不受方法集约束，编译器总是查找全部⽅法，并⾃动转换receiver实参。

# 2 方法集定义
* The method set of an interface type is its interface.
* The method set of any other type T consists of all methods declared with receiver type T.
* The method set of the corresponding pointer type *T is the set of all methods declared with receiver *T or T (that is, it also contains the method set of T).

# 3 嵌入结构体方法集
Given a struct type S and a defined type T(anonymous field), promoted methods are included in the method set of the struct as follows:
* If S contains an embedded field T, the method sets of S and *S both include promoted methods with receiver T. The method set of *S also includes promoted methods with receiver *T.
* If S contains an embedded field *T, the method sets of S and *S both include promoted methods with receiver T or *T.

# 4 总结
|         sceneario         | object value | object pointer |
|           ----            |     ----     |      ----      |
|      pointer method       |      yes     |       yes      |
|      value   method       |      yes     |       yes      |
| interface pointer method  |      no      |       yes      |
| interface value   method  |      yes     |       yes      |