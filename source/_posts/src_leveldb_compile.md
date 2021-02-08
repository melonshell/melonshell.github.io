---
title: levelDB编译
date: 2021-01-17 23:51:53
categories:
- levelDB
tags:
- levelDB
- 源码阅读
---

# 1 源码下载
git clone https://github.com/google/leveldb.git

# 2 编译安装
cd leveldb
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=Release .. && cmake --build .
在build目录生成静态库libleveldb.a, 将libleveldb.a复制到/usr/local/lib/, 相关头文件复制到/usr/local/include/

# 3 编译遇到问题
(1) CMake Error at CMakeLists.txt:308 (set_property):
    set_property could not find TARGET gtest.  Perhaps it has not yet been
    created.
(2) ld: library not found for -lbenchmark
    clang: error: linker command failed with exit code 1 (use -v to see invocation)

原因：最新的commit支持了googletest和benchmark, 但只是cmake里面加了googletest和benchmark，并没有在项目里加入第三方库;
解决方法1:
googletest和benchmark作为leveldb的子模块:
git submodule update --init --recursive

解决方法2:
cd third_party
git clone https://github.com/google/googletest.git
git clone https://github.com/google/benchmark.git   
