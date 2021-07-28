---
title: go常用命令
date: 2019-09-17 22:46:33
categories:
- go语言
tags:
- go
---

<font size=5>1 go env</font>  
显示Go语言的环境信息。  

<font size=5>2 go build</font>  
编译代码，以及依赖包。  
 
* 对于普通包，go build不会产生任何文件，go install会在$GOPATH/pkg生成相应的文件；
* 对于main包，go build会在当前目录生成可执行文件，go install/go build -o path/a.exe会在$GOPATH/bin下生成相应的文件；
* go build默认会编译当前目录下所有的go文件，可在go build后加上文件名，如go build a.go；
* 可以指定编译输出的文件名，go build -o yourname.exe，默认情况是package名（非main包），或第一个源文件的文件名（main包）；
* go build会忽略目录下以"_"或"."开头的go文件；
* 如果源码针对不同的操作系统需要不同的处理，那么可以根据不同的操作系统后缀命名文件，如有针对不同操作系统读取数组的程序：  
  array_linux.go array_darwin.go  array_windows.go array_freebsd.go  
  go build会选择性地编译以系统名结尾的文件（linux/darwin/windows/freebsd），如linux下只会编译array_linux.go文件；

go有三类源码文件：  
* 命令源码文件，声明属于main包，且包含一个无参数声明无返回值声明的main函数。命令源码文件是程序的运行入口，是每个独立运行程序必须拥有的。
* 库源码文件，不能直接运行，仅用于存放程序实体，这些程序实体可被其他代码使用
* 测试源码文件，名称以_test.go为后缀的代码文件，且必须包含Test或Benchmark为名称前缀的函数：  
  func TestXXX(t *testing.T) {}  
  以Test为名称前缀的函数，只能接受*testing.T的参数，这种测试函数是功能测试函数。  
  func BenchmarkXXX(b *testing.B) {}  
  以Benchmark为名称前缀的函数，只能接受*testing.B的参数，这种测试函数是性能测试函数。  

<font size=5>3 go get</font>  
动态获取远程代码包，目前支持BitBucket/GitHub/Google Code/Launchpad。  
此命令内部实际有两步:  
第一步：下载源码包  
第二步：执行go install  
参数介绍：  
* -d 只下载不安装
* -u 强制使用网络去更新包和它的依赖包

<font size=5>4 go clean</font>  
清理当前源码包，及其依赖包里面编译生成的文件；  

<font size=5>5 go doc和godoc</font>  
go doc工具会从Go程序和包文件中提取顶级声明的首行注释以及每个对象的相关注释，并生成文档，它也可以作为一个提供在线文档浏览的web服务器，http://golang.org就是通过这种形式实现。  

godoc安装：go get golang.org/x/tools/cmd/godoc  
下面以http子包为例：  
包文档查看：go doc net或者子包net/http  
包函数/结构体查看：go doc net/http SetCookie  
包函数代码查看：go doc -src net/http SetCookie  

在命令行执行godoc -http=:端口号，比如godoc -http=:8090，然后在浏览器打开127.0.0.1:8090，将会看到一个golang.org的本地Copy版本，可以查询pkg文档等内容。  

<font size=5>6 go run</font>  
编译并直接运行程序，它会产生一个临时文件(但不会生成exe文件)，直接在命令行输出执行结果，方便调试；  

<font size=5>7 go test</font>  
go test会自动测试每一个指定的代码包，多个代码包之间空格分隔：go test basic cnet/ctcp pkgtool。  
go test自动读取源码目录下名为*_test.go的文件，生成并运行测试用的可执行文件；默认不需要任何参数，会把源码包下所有的test文件测试完毕，也可以指定测试源码文件：go test envir_test.go  
常用参数：  
* -bench regexp 执行相应的benchmarks，例如-bench=.
* -run regexp 只运行regexp匹配的函数，例如-run=Array，则只执行Array开头的函数
* -cover 开启测试覆盖率

<font size=5>8 go install</font>  
第一步：编译生成结果文件(可执行文件或.a包)  
第二部：将生成结果移到$GOPATH/pkg或$GOPATH/bin  
参数支持go build的编译参数。  

<font size=5>9 go list</font>  
列出当前全部安装的package  

<font size=5>10 go version</font>  
查看go当前的版本  

<font size=5>11 go fmt</font>  
go代码格式化，go fmt是gofmt的一个包装命令  