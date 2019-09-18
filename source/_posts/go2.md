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
编译源码文件，代码包以及依赖包。  
go build后不跟任何代码包，那么命令将编译当前目录所对应的代码包。  

go有三类源码文件：  
* 命令源码文件，声明属于main包，且包含一个无参数声明无返回值声明的main函数。命令源码文件是程序的运行入口，是每个独立运行程序必须拥有的。
* 库源码文件，
* 测试源码文件，


<font size=5>3 go get</font>  

<font size=5>4 go clean</font>  

<font size=5>5 go doc和godoc</font>  

<font size=5>6 go run</font>  

<font size=5>7 go test</font>  

<font size=5>8 go install</font>  

<font size=5>9 go tool pprof</font>  

<font size=5>10 go tool cgo</font>  

<font size=5>11 go vendor</font>  

<font size=5>12 go module</font>  