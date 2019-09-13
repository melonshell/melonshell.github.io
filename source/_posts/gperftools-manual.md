---
title: gperftools使用指南
date: 2019-09-10 22:55:47
categories:
- 工具
- 性能分析
tags:
- gperftools
- 指南
---

# 编译
**1 下载**  
  github地址：https://github.com/melonshell/mygperftools

**2 解压**
  tar -xvzf libunwind-1.4-rc1.tar.gz  
  mv libunwind-1.4-rc1 libunwind  
  unzip gperftools-cpp.zip  
  mv gperftools-f47a52ce85c3d8d559aaae7b7a426c359fbca225 gperftools  

**3 编译libunwind**  
  64位操作系统需要libunwind，libunwind介绍：http://www.nongnu.org/libunwind/index.html  
  ./configure CFLAGS=-fPIC 
  make  
  生成的库文件目录：src/.libs，COPY出来：cp -R .libs ../libs

**4 编译gperftools**  
  ./autogen.sh  
  ./configure --help  查看帮助信息  
  <font color=DarkCyan><u>指定libunwind库的目录：</u></font>  
  ./configure CPPFLAGS=-I../libunwind/include LDFLAGS=-L../libunwind/libs  
  make  
  尽量不用多线程编译，否则可能会报错；  
  生成的库文件目录：.libs，COPY为非隐藏文件：cp -R .libs libs  
  
# 链接
**1 静态链接** 
  代码如下：  
  ```C

    #include "profiler.h"
	#include <iostream>
	using namespace std;
	void func1() {
	  int i = 0;
	  while (i < 100000) {
	      ++i;
	  }   
	}
	
	void func2() {
	  int i = 0;
	  while (i < 200000) {
	      ++i;
	  }   
	}
	
	void func3() {
	   for (int i = 0; i < 1000; ++i) {
	       func1();
	       func2();
	   }   
	}
	
	int main(){
	    ProfilerStart("my.prof"); // 指定所生成的profile文件名
	    func3();
	    ProfilerStop(); // 结束profiling
	    return 0;
	}
  ```
  
  编译，由于目录下同时存在静态库和动态库，因此须直接指定： 
  g++ -o test test.cc -I./gperftools/src/gperftools ./gperftools/libs/libprofiler.a ./libunwind/libs/libunwind.a  
  
  或者  
  g++ -o test test.cc -I./gperftools/src/gperftools -L./libunwind/libs/ -L./gperftools/libs/ -Wl,-Bstatic -lprofiler -lunwind -Wl,-Bdynamic 
  (链接静态库：-Wl,-Bstatic -llibname；链接动态库：-Wl,-Bdynamic -llibname)  

  <font color=BlueViolet>**如何Profile?**</font>  
  <font color=BlueViolet>**头文件包含gperftools/profiler.h，把要Profile的代码置于ProfilerStart()和ProfilerStop()；**</font>  

  这里有几点需要说明：  
  1) 链接顺序问题，libprofiler.a依赖libunwind.a；  
  2) 静态库由ar命令生成，也可用ar命令查看静态库包含哪些.o文件：  
     ar -t libprofiler.a  
  3) 静态库可以看成一组目标文件(.o/.obj文件)的集合；  
  4) 动态库和静态库都由目标文件生成，目标文件是在编译的**汇编**阶段生成，如果库有多级依赖，那么依赖库也需要在gcc/g++命令中按依赖顺序指定。  
  
**2 动态链接**  
  代码如下：  
  ```C
	#include <iostream>
    #include <unistd.h>
	using namespace std;
	void func1() {
	  int i = 0;
	  while (i < 100000) {
	      ++i;
	  }   
	}
	
	void func2() {
	  int i = 0;
	  while (i < 200000) {
	      ++i;
	  }   
	}
	
	void func3() {
	   for (int i = 0; i < 1000; ++i) {
	       func1();
	       func2();
	   }   
	}
	
	int main(){
        while(1) {
	    	func3();
            sleep(1);
        }
	    return 0;
	}
  ```
  编译，动态链接libprofiler.so:  
  g++ -o test test.cc -I./gperftools/src/gperftools -L./libunwind/libs/ -L./gperftools/libs/ -lprofiler -lunwind  
  静态库和动态库同名时，gcc/g++优先链接动态库；  
  
  ldd命令查看test依赖的动态库：  
  env LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./gperftools/libs/:./libunwind/libs/ ldd test  
  
  <font color=BlueViolet>**如何Profile?**</font>  
  <font color=BlueViolet>**方法1：定义环境变量CPUPROFILE(profile导出文件名称)**</font>  
      env LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./gperftools/libs/:./libunwind/libs/ CPUPROFILE=my.prof ./test
  <font color=BlueViolet>**方法2：定义环境变量CPUPROFILE和CPUPROFILESIGNAL，通过指定的信号值控制(信号值须程序本身未使用)profile，profile功能打开和关闭均由信号触发**</font>
      env LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./gperftools/libs/:./libunwind/libs/ CPUPROFILE=my.prof  CPUPROFILESIGNAL=12 ./test &
      killall -12 test
      killall -12 test
  
  以上动态库搜索路径是通过环境变量LD_LIBRARY_PATH指定，也可在编译时用**-Wl,-rpath**指定程序运行时的动态库搜索路径：  
      g++ -o test test.cc -I./gperftools/src/gperftools -L./libunwind/libs/ -L./gperftools/libs/ -lprofiler -lunwind -Wl,-rpath=/root/gperftools/libunwind/libs/:/root/gperftools/gperftools/libs/  
  profile执行命令：  
      env CPUPROFILE=my.prof ./test  
      env CPUPROFILE=my.prof  CPUPROFILESIGNAL=12 ./test &
  

# 使用     
  **1 图形化工具安装**
  gperftools依赖graphviz生成图形分析结果，安装graphviz：  
  yum install graphviz  
  yum install ghostscript  
  
  **2 pprof文件解析**  
  pprof --text ./test my.prof > output.txt  
  pprof --pdf ./test my.prof > output.pdf  
  生成结果格式如下：  
  <div style="height: 80%; width: 80%">![gperftools分析结果](/pic/gperftools201909130015.png)</div>

  **3 结果分析**  
  图形风格的结果有结点和有向边组成，每个结点代表一个函数，结点数据格式：  
  Class Name  
  Method Name  
  local (percentage)  
  of cumulative (percentage)  
  Profile通过采样实现，默认每秒采样100次，因此输出结果中的时间单位为10ms，例如func2执行本地指令耗时67个单位，约670ms。local是直接执行函数内的指令(包括内联函数)所消耗时间，cumulative是callees和local时间之和，如果cumulative时间和local时间相等，则cumulative不显示。
  
  

  