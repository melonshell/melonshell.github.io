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
                 