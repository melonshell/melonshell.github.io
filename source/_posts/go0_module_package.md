---
title: go基本概念
date: 2020-05-28 00:12:34
categories:
- go语言
tags:
- go
---

# module
A module is a collection of Go packages stored in a file tree with a go.mod file at its root. The go.mod file defines the module’s module path, which is also the import path used for the root directory, and its dependency requirements, which are the other modules needed for a successful build.

The module path is a prefix of the package import path, for every package in that module.

# package
import目录，非package名，但建议目录名和package名相同

同层级的目录，只能有一个package，比如mytest目录下的go文件package均应该相同，否则报错；mytest/comm目录下可以有其他的package

# go test
若go.mod module的目录为main，则go test报错：/tmp/go-build369883236/b001/_testmain.go:12:2: cannot import "main"，原因如下：
The test driver (_testmain.go) needs to import the package and if the import path is "main", gc will refuse to import it. The reason is simple, the packages in Go program must have unique import paths, but as the main package must have the import path "main", you can't import another "main" package.
