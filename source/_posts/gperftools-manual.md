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
  ./configure  
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
**2 动态链接**  

# 使用     
  