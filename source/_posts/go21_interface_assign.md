---
title: go接口赋值
date: 2021-02-09 14:35:11
categories:
- go语言
tags:
- go
- 赋值
- 接口
---

在做项目的过程中，经常用atomic.Value来实现配置文件的无锁加载。出于C语言的认知，每次都Store的是结构体指针，类似于如下代码：
```go
    //成员/全局变量
    var cfg atomic.Value

    //定时任务
    func LoadCfg() {
        tmpCfg := &BaseConfig{}
        parseCfg(tmpCfg)
        cfg.Store(tmpCfg)
    }
```
当时心中留下一个疑问：如果atomic.Value接口变量Store的是结构体本身而非指针会怎样？

# 1 接口底层实现
go描述interface的底层结构体为iface和eface，其中iface描述非空方法集接口，eface描述空方法集接口，即interface{}。
iface结构体如下：
```go
    type iface struct {
        tab  *itab
        data unsafe.Pointer
    }
    type itab struct {
        inter  *interfacetype
        _type  *_type
        link   *itab
        hash   uint32 // copy of _type.hash. Used for type switches.
        bad    bool   // type does not implement interface
        inhash bool   // has this itab been added to hash?
        unused [2]byte
        fun    [1]uintptr // variable sized
    }

    type interfacetype struct {
        typ     _type
        pkgpath name
        mhdr    []imethod
    }
```
iface有两个指针成员，tab指向itab结构体。itab包含静态类型信息*interfacetype，如io.Reader，以及动态（具体）类型信息*_type，还有具体类型的方法集fun，fun存储的为第一个方法的函数指针，多个方法内存连续存储，按照函数名字的字典序排列。
data为指向具体数据的指针。

eface结构体如下：
```go
    type eface struct {
        _type *_type
        data  unsafe.Pointer
    }
```
空接口的静态类型只有一种interface{}，且无方法集，故无需itab结构体，只需保存动态（具体）类型信息的*_type，data为指向具体数据的指针。
   ![接口底层原理](/pic/go_interface_ds.png)

# 2 接口底层数据查看
了解了接口的底层数据结构，我们就可以将iface/eface的成员data打印出来：
```go
    type Person interface {
        Walk()
    }

    type Man struct {
        Name string
        Age  int32
    }

    func (m Man) Walk() {
        fmt.Println("man walk")
    }

    // 检查Person接口对应iface结构体的data成员
    func PrintData(p *Person) {
        pd := (**Man)(unsafe.Pointer(uintptr(unsafe.Pointer(p)) + uintptr(unsafe.Sizeof(p))))
        fmt.Printf("iface &data:%p, data:%p, *data:%+v\n", pd, *pd, **pd)
    }
```
首先定义接口Person，然后定义struct Man实现该接口，PrintData用于打印Person接口对应的iface的data成员。unsafe.Pointer(p)指向iface/eface第一个指针成员，由于结构体内存连续，指针大小可通过unsafe.Sizeof(p)计算，因此data成员的地址为uintptr(unsafe.Pointer(p)) + uintptr(unsafe.Sizeof(p))，再类型转换为**Man即可。

或者采用如下方式：
```go
    type ifaceWords struct {
        typ unsafe.Pointer
        data unsafe.Pointer
    }

    func PrintData(p *Person) {
        d := (*Man)(((*ifaceWords)(unsafe.Pointer(p))).data)
        fmt.Printf("iface &data:%p, data:%p, *data:%+v\n", &d, d, *d)
    }
```

# 3 对象值赋值给接口变量
```go
   var person Person
   man := Man{"Jeff", 33}
   person = man

   fmt.Printf("man addr:%p\n", &man)
   PrintData(&person)
```
结果如下：
man addr:0xc00009c040
iface &data:0xc0000841e8, data:0xc00009c060, *data:{Name:Jeff Age:33}
可以发现man addr和data并不相同，接口Copy了一份man。
   ![接口赋值](/pic/go_interface_value.png)

# 4 对象指针赋值给接口变量
```go
   var person Person
   man := &Man{"Jeff", 33}
   person = man

   fmt.Printf("man addr:%p\n", man)
   PrintData(&person)
```
结果如下：
man addr:0xc00010c000
iface &data:0xc00010a048, data:0xc00010c000, *data:{Name:Jeff Age:33}
可以发现man addr和data值相同，指向同一个对象。
   ![接口赋指针](/pic/go_interface_pointer.png)

# 5 接口变量赋值给接口变量
## 5.1 接口变量保存值
```go
   var person, person1 Person
   man := Man{"Jeff", 33}
   person = man

   fmt.Printf("man addr:%p\n", &man)
   PrintData(&person)
   person1 = person
   PrintData(&person1)
```
结果如下：
man addr:0xc00010c000
iface &data:0xc00010a048, data:0xc00010c020, *data:{Name:Jeff Age:33}
iface &data:0xc00010a058, data:0xc00010c020, *data:{Name:Jeff Age:33}
可以发现person拷贝了一份man，但是person和person1的data相同。
可以发现man addr和data值相同，指向同一个对象。
   ![接口赋值](/pic/go_value_interface_assign.png)

正如所期望的一样，person和person1的iface的tab指针也相同：
d  := ((*ifaceWords)(unsafe.Pointer(&person))).typ
d1 := ((*ifaceWords)(unsafe.Pointer(&person1))).typ
fmt.Printf("d:%p d1:%p", d, d1)
结果如下：
d:0x4de560 d1:0x4de560

## 5.2 接口变量保存指针
```go
   var person, person1 Person
   man := &Man{"Jeff", 33}
   person = man

   fmt.Printf("man addr:%p\n", man)
   PrintData(&person)
   person1 = person
   PrintData(&person1)
```
结果如下：
man addr:0xc0000ae040
iface &data:0xc0000961e8, data:0xc0000ae040, *data:{Name:Jeff Age:33}
iface &data:0xc0000961f8, data:0xc0000ae040, *data:{Name:Jeff Age:33}
可以发现man、person iface.data、person1 iface.data值相同。
   ![接口赋值](/pic/go_pointer_interface_assign.png)
