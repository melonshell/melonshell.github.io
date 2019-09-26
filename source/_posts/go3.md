---
title: go pprof性能分析
date: 2019-09-25 16:41:07
categories:
- go语言
tags:
- go
---

# 1 pprof数据采样
pprof采样数据主要有三种获取方式：  
* runtime/pprof，手动调用runtime.StartCPUProfile runtime.StopCPUProfile等API生成和写入采样文件，灵活性高
* net/http/pprof，通过http服务获取Profile采样文件，适用于对应用程序的整理监控，通过runtime/pprof实现
* go test，go test -bench . -cpuprofile prof.cpu生成采样文件，适用于对函数进行针对性测试

## 1.1 runtime/pprof
runtime/pprof提供各种相对底层的API用于生成采样数据，一般应用程序更推荐使用net/http/pprof，runtime/pprof的API参考[runtime/pprof](https://golang.org/pkg/runtime/pprof/)或[http pprof实现](https://github.com/golang/go/blob/release-branch.go1.9/src/net/http/pprof/pprof.go)。  

我们可以使用gotest的命令行参数-cpuprofile和-memprofile，导入runtime/pprof，在[Hundt's benchmark programs](https://github.com/hundt98847/multi-language-bench)添加如下代码：  
```go
var cpuprofile = flag.String("cpuprofile", "", "write cpu profile to file")

func main() {
    flag.Parse()
    if *cpuprofile != "" {
        f, err := os.Create(*cpuprofile)
        if err != nil {
            log.Fatal(err)
        }   
        pprof.StartCPUProfile(f)
        defer pprof.StopCPUProfile()
    }
    ...
```
如设置了cpuprofile参数，CPU profile则会将Profile结果保存至此文件；最后需要调用StopCPUProfile，确保在程序退出前，所有数据都写入文件。  

./havlak1 -cpuprofile=havlak1.prof  

**go tool pprof havlak1 havlak1.prof**  
**(pprof) top10**  
输出结果如下：  

Showing nodes accounting for 39.93s, 64.17% of 62.23s total
Dropped 154 nodes (cum <= 0.31s)
Showing top 10 nodes out of 83
      flat  flat%   sum%        cum   cum%
     8.50s 13.66% 13.66%      9.99s 16.05%  runtime.findObject
     7.41s 11.91% 25.57%     13.93s 22.38%  runtime.scanobject
     5.38s  8.65% 34.21%      6.27s 10.08%  runtime.mapaccess1_fast64
     4.87s  7.83% 42.04%     44.51s 71.52%  havlakloopfinder.FindLoops
     3.40s  5.46% 47.50%     16.68s 26.80%  runtime.mallocgc
     2.43s  3.90% 51.41%      3.21s  5.16%  runtime.heapBitsSetType
     2.35s  3.78% 55.18%     11.03s 17.72%  runtime.gcWriteBarrier
     2.20s  3.54% 58.72%      7.86s 12.63%  runtime.mapassign_fast64ptr
     1.89s  3.04% 61.75%      9.29s 14.93%  runtime.wbBufFlush1
     1.50s  2.41% 64.17%      2.08s  3.34%  runtime.spanO 

开启CPU Profile，Go程序每10ms会记录一个当前运行goroutine栈的计数器组成的样本，输出各列含义如下：  
* flag： 采样时，该函数正在运行的次数 * 采样频率(10ms)，即估算的函数运行时间，此时间不包括等待被调函数返回；
* flag%：flag / 总采样时间
* sum%：前面所有行flat%的累加值，如第二行sum% = 25.57% = 13.66% + 11.91%
* cum：采样时，该函数出现在调用栈的采样时间，包括等待被调函数返回
* cum%：cum / 总采样时间


使用-cum参数按照第四第五列排序：  
**(pprof) top5 -cum**  
Showing nodes accounting for 4.93s, 7.92% of 62.23s total
Dropped 154 nodes (cum <= 0.31s)
Showing top 5 nodes out of 83
      flat  flat%   sum%        cum   cum%
         0     0%     0%     44.99s 72.30%  main.main
         0     0%     0%     44.99s 72.30%  runtime.main
         0     0%     0%     44.51s 71.52%  havlakloopfinder.FindHavlakLoops
     4.87s  7.83%  7.83%     44.51s 71.52%  havlakloopfinder.FindLoops
     0.06s 0.096%  7.92%     24.92s 40.04%  runtime.systemstack 

事实上，main.main和havlakloopfinder.FindLoops的总数应该是100%，但是每个栈样本仅包含栈底100帧，约30%的样本，递归函数havlakloopfinder.DFS深度（相比main.main）超过100帧，因此完整的trace被截断。  

如果安装了[graphviz](http://www.graphviz.org/)，**web**命令可以生成SVG格式的图形Profile，并且用浏览器打开：  
**(pprof) web**  
也可以生成pdf格式的Profile：  
**(pprof) pdf**  

也可以不用交互式：  
**go tool pprof -pdf ./havlak1 ./havlak1.prof > cpu.pdf**  

另外list FuncName可以显示函数名以及每行代码的采样分析：  
**(pprof) list havlakloopfinder.DFS **  
ROUTINE ======================== havlakloopfinder.DFS in /root/go/src/havlakloopfinder/havlakloopfinder.go
     710ms     11.37s (flat, cum) 18.27% of Total
         .          .    153:	return false
         .          .    154:}
         .          .    155:
         .          .    156:// DFS - Depth-First-Search and node numbering.
         .          .    157://
      20ms       20ms    158:func DFS(currentNode *cfg.BasicBlock, nodes []*UnionFindNode, number map[*cfg.BasicBlock]int, last []int, current int) int {
      10ms      560ms    159:	nodes[current].Init(currentNode, current)
      20ms      450ms    160:	number[currentNode] = current
         .          .    161:
         .          .    162:	lastid := current
      70ms      540ms    163:	for ll := currentNode.OutEdges().Front(); ll != nil; ll = ll.Next() {
     370ms         3s    164:		if target := ll.Value.(*cfg.BasicBlock); number[target] == unvisited {
     110ms      5.74s    165:			lastid = DFS(target, nodes, number, last, lastid+1)
         .          .    166:		}
         .          .    167:	}
      90ms      1.04s    168:	last[number[currentNode]] = lastid
      20ms       20ms    169:	return lastid
         .          .    170:}
         .          .    171:
         .          .    172:// FindLoops
         .          .    173://
         .          .    174:// Find loops and build loop forest using Havlak's algorithm, which

  <div style="height: 80%; width: 80%">![CPU Profile结果](/pic/go3201909262046.jpg)</div>  
分析Profile结果发现，mapaccess1_fast64占用CPU采用时间最多，




