---
title: go mod基本命令
date: 2020-06-19 10:50:34
categories:
- go语言
tags:
- go
---

# 清理缓存
go clean --modcache

# 增加缺失的包，移除没用的包
go mod tidy

# go mod使用master分支
在go mod目录执行:
go get git.code.oa.com/nfa/nfalib@master 或 go get git.code.oa.com/nfa/nfalib@commitid
go get会自动更新依赖文件(go.mod, go.sum)

# 依赖关系
go list -m all显示依赖关系
go list -m -json all显示详细依赖关系
go mod graph 打印模块依赖

# 依赖不同版本
```go
module github.com/Xuanwo/go-mod-intro

go 1.12

require (
        github.com/google/go-github/v24 v24.0.1
        github.com/google/go-github/v25 v25.0.4
)
```
可以在同一个文件中引用同一个模块的不同大版本，它们的导入路径不同，所以被看作两个不同的模块

# 例子
原文地址：https://xuanwo.io/2019/05/27/go-modules/
```go
package main

import (
	v24 "github.com/google/go-github/v24/github"
	v25 "github.com/google/go-github/v25/github"
    "golang.org/x/text/width"

)

var (
	_ = v24.Tag{}
	_ = v25.Tag{}
	_ = width.EastAsianAmbiguous
)

func main() {
	return
}
```
go.mod中增加一行：golang.org/x/text v0.3.0
使用go list -m all查看当前模块所有的依赖：
```shell
go list -m all
github.com/Xuanwo/go-mod-intro
github.com/golang/protobuf v1.2.0
github.com/google/go-github v17.0.0+incompatible
github.com/google/go-github/v24 v24.0.1
github.com/google/go-github/v25 v25.0.4
github.com/google/go-querystring v1.0.0
golang.org/x/crypto v0.0.0-20190308221718-c2843e01d9a2
golang.org/x/net v0.0.0-20190311183353-d8887717615a
golang.org/x/oauth2 v0.0.0-20180821212333-d2e6202438be
golang.org/x/sync v0.0.0-20190227155943-e225da77a7e6
golang.org/x/sys v0.0.0-20190215142949-d0b11bdaac8a
golang.org/x/text v0.3.0
google.golang.org/appengine v1.1.0
```
下面我们把golang.org/x/text依赖的v0.3.0修改成v0.2.0，然后重新执行go list -m all查看最终选择的版本：
```shell
go list -m all
go: finding golang.org/x/text v0.2.0
github.com/Xuanwo/go-mod-intro
...
golang.org/x/text v0.3.0
```
可以发现go在查找了golang.org/x/text v0.2.0之后实际选择的还是v0.3.0，我们可以用go mod graph | grep text来看看谁在依赖这个模块：
```shell
go mod graph | grep text
github.com/Xuanwo/go-mod-intro golang.org/x/text@v0.3.0
golang.org/x/net@v0.0.0-20190311183353-d8887717615a golang.org/x/text@v0.3.0
```
因为golang.org/x/net在依赖golang.org/x/text@v0.3.0，所以即使我们在go.mod中强行指定了v0.2.0，最后还是会选择v0.3.0来进行构建，不仅如此，我们的go.mod文件中依赖也被修改成了v0.3.0，因为这才是我们依赖的最终版本。

下面我们来试一下如果指定成v0.3.2会如何：
```shell
go list -m all
go: finding golang.org/x/text v0.3.2
github.com/Xuanwo/go-mod-intro
...
golang.org/x/text v0.3.2
```
显然的，v0.3.2 > v0.3.0，所以最后选择了 v0.3.2

# go mod依赖多个版本
最小版本选择算法:https://zhuanlan.zhihu.com/p/59549613

# replace
replace directives provide additional control in the top-level go.mod for what is actually used to satisfy a dependency found in the Go source or go.mod files, while replace directives in modules other than the main module are ignored when building the main module.

KdAdsNewRecallServer(go.mod)依赖nfalib(go.mod)，nfalib依赖attaapi_go(go.mod)，因此KdAdsNewRecallServer间接依赖attaapi_go。在KdAdsNewRecallServer(go.mod) require中不需要引入attaapi_go，对attaapi_go进行replace：
```shell
replace git.code.oa.com/atta/attaapi_go => ../attaapi_go
```
然后go list -m all | grep atta：
```shell
go list -m all | grep atta
输出结果：
git.code.oa.com/atta/attaapi_go v1.6.5 => ../attaapi_go
```