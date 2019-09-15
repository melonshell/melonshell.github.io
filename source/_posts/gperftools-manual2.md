---
title: gperftools之heap profile
date: 2019-09-14 23:36:27
categories:
- 工具
- gperftools
tags:
- gperftools
- 内存
- heap profile
---

源文件leak.cc：  
```C
#include <stdlib.h>
int main() {
    const int array_count = 99; 
    int* p = new int[array_count];
    return 0;
}
```

# 对整个程序Profile
### 1 动态链接
heap profiler需要-ltcmalloc：  
g++ -g leak.cc -o leak -L./gperftools/libs -ltcmalloc -Wl,-rpath=/root/gperftools/gperftools/libs
**加上-g参数可以打印出行号等更多的调试信息**  

定义环境变量HEAPCHECK，执行：  
env HEAPCHECK=normal ./leak  
输出结果会显示内存泄露情况，同时**提示pprof**命令；  

定义环境变量HEAPPROFILE(heap profile导出文件名称)，执行：  
env HEAPPROFILE=d.prof ./leak  
会将内存信息写入d.prof.0001.heap文件；  

env HEAPPROFILE=d.prof HEAPCHECK=normal ./leak  
输出结果会显示内存泄露情况，且将内存信息写入d.prof.0001.heap文件；  

### 2 运行时LD_PRELOAD加载
编译:  
g++ -g leak.cc -o leak  

执行命令：  
env LD_PRELOAD="/root/gperftools/gperftools/libs/libtcmalloc.so" HEAPCHECK=normal HEAPPROFILE=n.prof ./leak  

# pprof查看profile结果
根据提示执行命令：  
**heap profiler:**  
pprof ./leak "d.prof.0001.heap" --pdf > d.pdf 
pprof ./leak "d.prof.0001.heap" --text   

**heap checker:**  
pprof ./leak "/tmp/leak.15356._main_-end.heap" --inuse_objects --lines --heapcheck  --edgefraction=1e-10 --nodefraction=1e-10 --pdf > d.pdf  
pprof ./leak "d.prof.0001.heap" --inuse_objects --lines --heapcheck  --edgefraction=1e-10 --nodefraction=1e-10 --pdf > d.pdf  
pprof ./leak "/tmp/leak.15700._main_-end.heap" --inuse_objects --lines --heapcheck  --edgefraction=1e-10 --nodefraction=1e-10 --pdf > n.pdf  
pprof ./leak "n.prof.0001.heap" --inuse_objects --lines --heapcheck  --edgefraction=1e-10 --nodefraction=1e-10 --pdf > n.pdf  

结果如下：  
<div style="height: 80%; width: 80%">![gperftools内存分析结果](/pic/gperftools2201909151028.png)</div>  


