---
title: wireshark使用
date: 2021-01-15 15:33:15
categories:
- 工具
tags:
- wireshark
---

# 1 wireshark显示TCP连接stream序号
  ![显示stream序号](/pic/wireshark1.png)

* 点击一条抓包记录
* 在下方[Stream index]上右键
* 应用为列

这样就可以根据tcp.stream排序，相当于对整个抓包记录追踪TCP流