---
title: go tool cgo入门
date: 2019-10-03 09:30:07
categories:
- go语言
tags:
- go
---

go详细命令介绍参见[go命令](https://github.com/hyper0x/go_command_tutorial)。  

# 1 Go代码调用C函数
go语言可以通过cgo工具调用C代码，C代码被封装在虚拟包package "C"中，你可以访问C实现的类型C.size_t，变量C.stdout和方法C.putchar，即使首字母是小写。  

启用CGO： **import "C"**，import "C"需单独一行，不能与其他包一同import。  

在import "C"之前有注释(和注释之间没有空行)，称之为序文。序文可以包含C头文件和C代码，可以在C代码中定义变量和函数，然后在Go代码中通过C来引用，C代码中的静态变量不能在Go中使用，但是静态函数可以。  

需要注意的是，Go是强类型语言，所以CGO中传递的参数类型必须与声明的类型完全一致，传递前必须用"C"中的转化函数转换为对应的C类型，不能直接传入Go类型变量。 虚拟C包导入的C语言符号并不需要大写首字母，它们不受Go语言的导出规则约束。下面看一个例子cgo.go：  

```
package main
/*
#include "stdio.h"
void test(int n) {
	char dummy[10240];
	printf("in c test func iterator %d\n", n);
	if(n <= 0) {
	return;
	}
	dummy[n] = '\a';
	test(n-1);
}
#cgo CFLAGS: -g
*/
import "C"
func main() {
	C.test(C.int(2))
}
```

对于C语言的原生类型，CGO都会将其映射为Go语言中的类型：C.char和C.schar（对应于C语言中的signed char），C.uchar（对应于C语言中的unsigned char），C.short和C.ushort
（对应于unsigned short），C.int和C.uint（对应于unsigned int），C.long和C.ulong（对应于unsigned long），C.longlong（对应于C语言中的long long），C.ulonglong（对
应于C语言中的unsigned long long类型）以及C.float和C.double，C语言中的void*指针类型在Go语言中则用特殊的unsafe.Pointer类型来对应。  

C语言中的struct、union和enum类型，对应到Go语言中都会变成带这样前缀的类型名称：struct_、union_和enum_，比如一个在C语言中叫做person的struct会被CGO翻译为C.struct_person。如果C语言中的类型名称或变量名称与Go语言的关键字相同，CGO会自动给这些名字加上下划线前缀；但是如果有两个成员，一个以Go语言关键字命名，另一个刚好是以下划线和Go语言关键字命名，那么以Go语言关键字命名的成员将无法访问(被屏蔽)。  

访问C类型T的size用C.sizeof_T，如C.sizeof_struct_stat；调用C的函数可以进行多值赋值，一个值作为返回值，一个作为errno。  

# 2 编译
运行go tool cgo t.go，会在本地生成一个_obj文件夹，实际开发中，我们不会直接调用cgo工具，因为go build会自动完成这一切go build t.go：
```
[root@VM_100_12_centos test]# ls _obj/
_cgo_export.c  _cgo_export.h  _cgo_flags  _cgo_gotypes.go  _cgo_main.c  _cgo_.o  t.cgo1.go  t.cgo2.c
```
每个cgo文件会展开为一个Go文件和C文件，分别以.cgo1.go和.cgo2.c为后缀名，大多数作用都是包装现有的函数或者声明：  
_cgo_.o：编译器编译C文件后生成的目标文件；  
t.cgo2.c：生成_cgo_3cf70ea79b6d_Cfunc_test来包装test函数
```
void
_cgo_3cf70ea79b6d_Cfunc_test(void *v)
{
        struct {
                int p0;
                char __pad4[4];
        } __attribute__((__packed__, __gcc_struct__)) *_cgo_a = v;
        _cgo_tsan_acquire();
        test(_cgo_a->p0);
        _cgo_tsan_release();
}
```
t.cgo1.go：包含main函数，它调用C函数test
```
import _ "unsafe"
func main() {
    ( /*line :17:2*/_Cfunc_test /*line :17:7*/)( /*line :17:9*/_Ctype_int /*line :17:14*/(2))
}
```
_cgo_gotypes.go：生成_Cfunc_test函数，它调用_cgo_3cf70ea79b6d_Cfunc_test。对应C导入到Go语言中相关函数或变量的桥接代码。  
```
//go:cgo_unsafe_args
func _Cfunc_test(p0 _Ctype_int) (r1 _Ctype_void) {
    _cgo_runtime_cgocall(_cgo_3cf70ea79b6d_Cfunc_test, uintptr(unsafe.Pointer(&p0)))
    if _Cgo_always_false {
        _Cgo_use(p0)
    }   
    return
}
```
_cgo_export.h：对应导出的Go函数和类型  
_cgo_export.c：对应相关包装代码的实现  

cgo封装的原因来自两个方面：一方面是Go运行时调用cgo代码需要做特殊处理，比如_cgo_runtime_cgocall；另一方面是由于Go和C使用的命名空间不一样，需要加一层转换。 
 
cgo会识别任意的C.XXX关键字，使用gcc找到XXX定义，所有出现的C.XXX类型会被转换为_Ctype_XXX，C.XXX类型不能跨越多个包，因此不同包之间的C.int并不是相同的类型。cgo转换中最重要的部分是函数，如果XXX是一个C函数，那么cgo会重写C.XXX为一个新的函数_Cfunc_XXX，此函数会在一个标准pthread中调用C的XXX，另外还负责进行参数转换，转换输入参数，调用XXX，然后转换返回值。  

# 3 #cgo指令
可以用#cgo指令为C/C++编译器提供CFLAGS，CPPFLAGS，CXXFLAGS和LDFLAGS设置，同时也可以提供一些编译的约束，比如特定的平台参数：  
```
// #cgo CFLAGS: -DPNG_DEBUG=1
// #cgo amd64 386 CFLAGS: -DX86=1
// #cgo LDFLAGS: -lpng
// #include <png.h>
import "C"
```
CPPFLAGS和LDFLAGS可以通过pkg-config工具获取：  
```
// #cgo pkg-config: png cairo
// #include <png.h>
import "C"
```
变量${SRCDIR}用来指定当前源文件所在的目录的绝对路径，这允许你将预先编译好的静态库放在本地文件夹，让编译器可找到这些库正确链接，如包foo在文件夹/go/src/foo下：  
```
// #cgo LDFLAGS: -L${SRCDIR}/libs -lfoo
```
上述指令等价于：  
```
// #cgo LDFLAGS: -L/go/src/foo/libs -lfoo
```
下面给出一个计算圆周率1000位的例子：  
pi.c  
```
#include <stdio.h>
int a=10000, b, c=2800, d, e, f[2801], g,i;
char r[1000];
char* pr = r;
char* calc() {
    for(;b-c;)
            f[b++]=a/5;
    for(;d=0,g=c*2;c-=14,sprintf(pr,"%.4d",e+d/a),pr +=4,e=d%a)
    for(b=c;d+=f[b]*a,f[b]=d%--g,d/=g--,--b;d*=b);
    return r;
}
```
pi.h  
```
char* calc();
```

编译成动态库：  
```
gcc -o libpi.so -fPIC -shared pi.c
```

用Go代码使用这个库：  
和C文件同一个目录pi.go  
```
package main
/*
#cgo CFLAGS: -I${SRCDIR}
#cgo LDFLAGS: -L${SRCDIR} -lpi

#include "pi.h"
 */
import "C" 
import "fmt"

func main()  {
    fmt.Println("pi:")
    v := C.GoString(C.calc())
    fmt.Println(v)
}
```
Go代码编译：  
```
go build -o pi cgo3.go
```

执行命令：  
```
env LD_LIBRARY_PATH=./ ./pi
```

或者cgo中直接指定-Wl,-rpath：  
```
package main
/*
#cgo CFLAGS: -I${SRCDIR}
#cgo LDFLAGS: -L${SRCDIR} -lpi
#cgo LDFLAGS: -Wl,-rpath=./

#include "pi.h"
 */
import "C" 
import "fmt"

func main()  {
    fmt.Println("pi:")
    v := C.GoString(C.calc())
    fmt.Println(v)
}
```

现在再将pi.c编译成静态库：  
```
gcc -c pi.c
ar -r libpi.a pi.o
```
Go代码编译：  
```
package main

/*
#cgo CFLAGS: -I${SRCDIR}
#cgo LDFLAGS: -L${SRCDIR} -lpi
#include "pi.h"
 */
import "C" 
import "fmt"

func main()  {
    fmt.Println("pi:")
    v := C.GoString(C.calc())
    fmt.Println(v)
}

go build -o pi pi.go
./pi
```

# 4 CGO内存模型
## 4.1 Go访问C内存
C语言空间的内存是稳定的，只要不是被人为提前释放，Go可以放心使用，Go语言方位C语言内存是最简单的情形。  
因为Go语言实现的限制，我们无法在Go中创建大于2GB内存的切片，不过借助CGO技术，可以实现：  
```
package main

/*
#include <stdlib.h>

void* makeslice(size_t memsize) {
    return malloc(memsize);
}
*/
import "C"
import "unsafe"

func makeByteSlize(n int) []byte {
    p := C.makeslice(C.size_t(n))
    return ((*[1 << 32 + 1]byte)(p))[0:n:n]
}

func freeByteSlice(p []byte) {
    C.free(unsafe.Pointer(&p[0]))
}

func main() {
    s := makeByteSlize(1<<32 + 1)
    s[len(s)-1] = 255
    print(s[len(s)-1])
    freeByteSlice(s)
}
```

## 4.2 C临时访问传入的Go内存
C/C++很多库需要通过指针直接处理传入的内存数据，因此CGO有很多需要将Go内存传入C函数的场景。假设一个极端场景：将一块位于某goroutine的栈上的Go内存传入C函数后，在此C函数执行期间，goroutine的栈因为空间不足发生了扩容，即导致原来的Go内存被转移到新位置，但是C函数并不知道内存已经移动了位置，仍然用之前的地址来操作内存，这将导致内存越界，就是说C访问传入的Go内存可能是不安全的。  

一种方式是通过完全传值处理，借助C语言内存稳定的特性，在C语言空间先开辟同样大小的内存，然后将Go内存填充到C内存，返回的内存同样处理，下面看一个例子：  
```
/*
#include "stdio.h"
#include "stdlib.h"
void printString(const char* s) {
	printf("c str:%s\n", s);
}
*/
import "C"
import (
	"fmt"
	"unsafe"
)

func printString(s string) {
	cs := C.CString(s)
	defer C.free(unsafe.Pointer(cs))
	fmt.Println(cs)
	C.printString(cs)
}

func main()  {
	s := "hello"
	printString(s)
}
```
将Go字符串传入C函数时，先通过C.CString()将Go语言字符串对应的内存数据复制到新创建的C内存空间，上面例子处理思路是安全的，但是效率较低(需要多次分配内存并复制元素)。另外C函数printf是一个行缓冲函数，会先将输出写到缓冲区；行缓冲在缓冲区满或输入和输出遇到换行符时，执行真正的I/O操作，典型代表是标准输入(stdin)和标准输出(stdout)。标准错误不带缓冲。  

为优化这种Go内存传入C函数的问题，CGO制定了规则：在CGO调用的C函数返回之前，CGO保证传入的Go语言内存在此期间不会发生移动。  
```
/*
#include <stdio.h>
void printString(const char* s, int n) {
	int i;
    for(i = 0; i < n; i++) {
    	putchar(s[i]);
	}
	putchar('\n');
}
 */
import "C"
import (
	"reflect"
	"unsafe"
)

func printString(s string) {
	p := (*reflect.StringHeader)(unsafe.Pointer(&s))
	C.printString((*C.char)(unsafe.Pointer(p.Data)), C.int(len(s)))
}

func main()  {
	s := "hello"
	printString(s)
}
```

此法避免了额外的内存分配，但是如果C函数需要运行较长时间，那么将导致Go内存在C函数返回前不能移动，从而可能导致这个Go内存栈对应的goroutine不能动态伸缩内存栈，即可能导致goroutine阻塞。因此，运行较长时间的C函数需要谨慎传入Go内存。  

特别注意的是，在取得Go内存后需要马上传入C函数，不能保存到临时变量后再间接传入C函数，因为CGO只能保证在C函数调用之后被传入的Go内存不会发生移动，并不能保证在传入C函数之前内存不发生变化。以下代码是错误的：  
```
// 错误的代码
tmp := uintptr(unsafe.Pointer(&x))
pb := (*int16)(unsafe.Pointer(tmp))
*pb = 42
```
tmp并不是指针类型，在它获取到Go内存地址后x对象可能会被移动，但是因为不是指针类型，所以不会被Go语言运行时更新为新内存地址，在非指针类型的tmp保持Go对象的地址，和在C环境保持Go对象地址的效果是一样的：如果原始Go对象内存发生了移动，Go语言运行时并不会同步更新它们。  

