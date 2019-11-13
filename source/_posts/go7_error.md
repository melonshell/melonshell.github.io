---
title: go语言之Error处理推荐方案
date: 2019-11-09 09:04:12
categories:
- go语言
tags:
- go
- error
---

# 1 error接口
error是一个内置接口，定义如下：  
```go
// The error built-in interface type is the conventional interface for
// representing an error condition, with the nil value representing no error.
type error interface {
	Error() string
}
```

# 2 自定义error
error接口只有一个方法Error，只要实现了这个方法，就实现了error，现在我们自定义一个error：  
```go
type fileError struct {
}

func(fe *fileError) Error() string {
	return "file error";
}
```
测试效果如下：  
```go
func openFile()([]byte, error) {
	return nil, &fileError{}
}
func main() {
	content, err := openFile()
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(string(content))
	}
}
```
运行代码可以看到file error的错误通知。  

# 3 优化
实际项目中会遇到很多错误，他们的区别就是错误信息不一样，如果对每种错误都定义一个错误类型，那这样非常麻烦。Error返回的其实是一个字符串，我们可以修改下，让这个字符串可以设置：  
```go
type fileError struct {
	s string
}

func(fe *fileError) Error() string {
	return fe.s;
}

func openFile()([]byte, error) {
	return nil, &fileError{s:"file error"}
}
```
如此一来就达到了我们的目的。但我们希望更加通用，比如修改fileError的名字，再创建一个辅助函数，用于创建不同的错误类型：  
```go
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}

func New(txt string) error {
	return &errorString{txt}
}
```
我们就可以用New函数创建不同的错误了，这其实就是经常用到的errors.New函数。  

# 4 存在问题
对开发者来说，Go的错误设计虽然简洁但是存在很多不足，比如更多的错误信息，在什么文件，哪一行代码，只有这些才更容易定位问题，还有我们希望对返回的error附加更多的信息，以上述例子来说，我们只能先通过Error方法取出原来的错误信息，然后拼接，再使用errors.New函数生成新错误返回。如果我们以前做过java开发，就会知道Java的异常是可以嵌套的，也就是说，通过异常很容易知道错误的根本原因，因为Java的异常，是一层层的嵌套返回，不管中间经历了多少包装，我们可以通过cause找到根本错误的原因。  

要解决以上问题，必须扩充errorString，再增加一些字段来存储更多的信息，比如记录调用堆栈：  
```go
type stack []uintptr

type errorString struct {
	s string
	*stack
}
```
有了存储堆栈的stack字段，我们在生成错误的时候，就可以把调用的堆栈信息存储在此字段：  
```
type stack []uintptr

type errorString struct {
	s string
	*stack
}

func callers() *stack {
	const depth = 32
	var pcs [depth]uintptr
	n := runtime.Callers(, pcs[:])
	var st stack = pcs[0:n]
	return &st
}

func (e *errorString) Error() string {

	var m string = e.s + "\ncall stack:\n"

	fmt.Println("len stack", len(*e.stack))
	for i := 0; i < len(*e.stack); i++ {
		f := runtime.FuncForPC((*e.stack)[i])
		file, line := f.FileLine((*e.stack)[i])
		m += file + " " + fmt.Sprintf("%d", line) + " " + f.Name() + "\n"
	}

	return m
}

func New(txt string) error {
	return &errorString{
		s : txt,
		stack : callers(),
	}
}
```
再对现有错误附加一些信息:  
```go
type withMessage struct {
	cause error
	msg   string
}

func WithMessage(err error, message string) error {
	if err == nil {
		return nil
	}
	return &withMessage{
		cause: err,
		msg:   message,
	}
}
```

# 5 推荐方案

以上我们解决问题采取的方法是不是比较熟悉？其实就是github.com/pkg/errors错误处理库的源代码。因为Go语言提供的错误太简单无法为我们提供更有用的信息，所以诞生了很多错误处理库，github.com/pkg/errors比较简洁，且功能强大。它的使用非常简单，如果我们要新生成一个错误，可以使用New函数,生成的错误，自带调用堆栈信息：  
```go
func New(message string) error
```
如果有一个现成的error，我们需要对他进行再次包装处理，这时候有三个函数可以选择:   
```go
//只附加新的信息
func WithMessage(err error, message string) error

//只附加调用堆栈信息
func WithStack(err error) error

//同时附加堆栈和信息
func Wrap(err error, message string) error
```
上面的包装很类似于Java的异常包装，被包装的error，其实就是Cause，在前面的章节提到错误的根本原因就是这个Cause，所以这个错误处理库为我们提供了Cause函数让我们可以获得最根本的错误原因：  
```go
func Cause(err error) error {
	type causer interface {
		Cause() error
	}

	for err != nil {
		cause, ok := err.(causer)
		if !ok {
			break
		}
		err = cause.Cause()
	}
	return err
}
```
使用for循环一直找到最底层的那个error。

以上的错误我们都包装和收集完毕，那么怎么把他们里面存储的堆栈、错误原因等这些信息打印出来呢？其实，这个错误处理库的错误类型都实现了Formatter接口，我们可以通过fmt.Printf函数输出对应的错误信息：  

* %s,%v //功能一样，输出错误信息，不包含堆栈
* %q //输出的错误信息带引号，不包含堆栈
* %+v //输出错误信息和堆栈

以上如果有循环包装错误类型的话，会递归的把这些错误输出。  
