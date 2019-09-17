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
* GOBIN：go install可执行文件存放路径，不允许设置多个，可以为空，为空时可执行文件放在各自GOPATH/bin文件夹中
* PATH：将%GOBIN%添加至PATH，方便命令行运行

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
