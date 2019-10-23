---
title: gdb调试go
date: 2019-10-20 09:07:31
categories:
- go语言
tags:
- go
- gdb
---

# 1 gdb调试go

golang这类静态型语言调试工具必不可少，gdb支持调试golang程序，GDB版本必须大于7.1，go build需要带上-gcflags "-N -l"参数，关闭内联优化。  

配置GDB：  
* 打开gdb初始化配置文件：vim ~/.gdbinit
* 添加配置： add-auto-load-safe-path /usr/local/go/src/runtime/runtime-gdb.py

如果不能加载，可在GDB中手动加载：  
source /usr/local/go/src/runtime/runtime-gdb.py

**如果gdb调试出现"Loading Go Runtime support"，则表明gdb支持golang。**  

# 2 例子
```go
package main

import "fmt"
func main(){
	c:=make(map[string]interface{})
	fmt.Println(c)
}
```

编译链接：  
```shell
go build -gcflags "-N -l" t2.go   
```

gdb调试，**-tui**在运行时同时显示代码，以TUI模式运行GDB，或者在处于非TUI模式时在GDB中使用Ctrl+X+A组合键，切换至TUI模式，如果已经处于TUI模式，则Ctrl+X+A退出TUI模式：    
```shell
gdb -tui t2
(gdb) b t2.go:6
(gdb) layout
```
