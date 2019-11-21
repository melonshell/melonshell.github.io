---
title: go语言之指针和值接收者
date: 2019-11-21 00:28:31
categories:
- go语言
tags:
- go
- 接收者
---

# 1 struct值和指针接收者调用
```
type bird struct{
}

func (b *bird) run() {
}

func (b bird)  speak() {
}

func main() {
	var bv bird = bird{}
	var bp *bird = &bird{}
	bv.speak() //ok
	bv.run()   //调用指针ok
	bp.run()   //ok
	bp.speak() //ok
}
```
编译器会做隐式类型转换，bv.run()转换为(&bv).run()，bp.run()转化为(*bp).run()，但是对于interface，情况会有些许不同。

# 2 interface值和指针接收者调用
```go
type animal interface {
	run()
	speak()
}

type bird struct{
}

func (b *bird) run() {
}

func (b bird)  speak() {
}

func main() {
	var b bird = bird{}
	var a animal = b   //error
	//var a animal = &b   //ok
}
```
报错：Cannot use 'b' (type bird) as type animal in assignment. Type does not implement 'animal' as 'run' method has a pointer method.

从这个例子可以看到：指针接收者方法的值接收者函数是非法的，因为指针接收者方法不在值接收者的方法集中； 值接收者方法的指针接收者函数是合法的，因为值接收者方法在指针接收者方法集中。具体原因参考go说明文档[https://golang.org/ref/spec](https://golang.org/ref/spec):

The final case, a value-receiver function for a pointer-receiver method, is illegal because pointer-receiver methods are not in the method set of the value type.

**Method sets**
A type may have a method set associated with it. The method set of an interface type is its interface. The method set of any other type T consists of all methods declared with receiver type T. The method set of the corresponding pointer type *T is the set of all methods declared with receiver *T or T (that is, it also contains the method set of T). Further rules apply to structs containing embedded fields, as described in the section on struct types. Any other type has an empty method set. In a method set, each method must have a unique non-blank method name.

The method set of a type determines the interfaces that the type implements and the methods that can be called using a receiver of that type.

接口类型的方法集是其接口，**其他类型T的方法集包括以值接收者T声明的所有方法；对应指针类型\*T的方法集是所有以接收者\*T或T声明的方法集合(即包含T的方法集)**。