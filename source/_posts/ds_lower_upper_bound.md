---
title: lower_bound和upper_bound算法
date: 2020-01-18 23:24:13
categories:
- 数据结构与算法
tags:
- lower_bound
- upper_bound
- 查找
---

# 1 定义
lower_bound：在有序序列中，查找第一个不小于k的元素位置；
upper_bound：在有序序列中，查找第一个大于k的元素位置；
下面假设数组升序。

# 2 lower_bound
```cpp
int lower_bound(int a[], int left, int right, int k)
{
    while(left < right)
    {
        int mid = (left + right) / 2;
        if(a[mid] < k)
        {
            left = mid + 1;
        }
        else
        {
            right = mid;
        }
    }

    if(a[left] >= k) return left;

    return -1;
}
```
**算法思想**
```
1) 只要left < right，必定mid < right；
2) 若a[mid] < k，则第一个不小于k的元素必定位于[mid + 1, right]；
3) 若a[mid] >= k，则第一个不小于k的元素必定位于[left, mid]；
4) 当left == right退出while循环时，判断a[left]是否满足条件；
```

# 3 upper_bound
```cpp
int upper_bound(int a[], int left, int right, int k)
{
    while(left < right)
    {
        int mid = (left + right) / 2;
        if(a[mid] <= k)
        {
            left = mid + 1;
        }
        else
        {
            right = mid;
        }
    }

    if(a[left] > k) return left;

    return -1;
}
```
**算法思想**
```
1) 只要left < right，必定mid < right；
2) 若a[mid] <= k，则第一个大于k的元素必定位于[mid + 1, right]；
3) 若a[mid] > k，则第一个大于k的元素必定位于[left, mid]；
4) 当left == right退出while循环时，判断a[left]是否满足条件；
```