---
title: go环境搭建
date: 2019-09-16 23:52:09
categories:
- go语言
tags:
- go
---

# 下载
安装包下载地址：https://golang.org/dl/  
如果打不开，则改用地址：https://golang.google.cn/dl/  

# windows安装
默认情况下.msi文件会安装在C:\Go目录，配置环境变量：  
* GOROOT：Go安装的根目录，如C:\Go
* GOPATH：工作目录，允许多个，多目录分隔符为;，如D:\Go
* GOBIN：go install生成的可执行文件存放路径，为空时默认为GOPATH/bin
* PATH：将GOROOT/bin、%GOBIN%和GOPATH/bin添加至PATH，方便命令行运行

# linux安装
1. 解压安装包至/usr/local：  
```shell
tar -C /usr/local -xvzf go1.13.linux-amd64.tar.gz
```
2. 将/usr/local/go/bin添加至环境变量： 
vim .bashrc 
```shell
export GOROOT=/usr/local/go
export GOPATH=/root/go
export PATH=$PATH:$GOROOT/bin:${GOPATH//://bin:}/bin
```
其中export PATH=$PATH:${GOPATH//://bin:}/bin将每个GOPATH下的bin都加入PATH。  

# 工作目录
$GOPATH有三个子目录：  
* src存放源代码
* pkg存放编译后生成的.a文件
* bin存放go install生成的可执行文件(为了方便，可以把此目录加入到$PATH变量中，如果有多个gopath，那么使用${GOPATH//://bin:}/bin添加所有的bin目录）

# 运行
1. 检查go是否安装成功：  
go env  
go env命令用于查看go环境的安装配置变量。  

2. 测试代码 
test.go 
```go
package main

import "fmt"

func main() {
   fmt.Println("Hello, World!")
}
```

执行：go run test.go  
输出：Hello, World!
