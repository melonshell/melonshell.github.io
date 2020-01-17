---
title: 旋转数组查找
date: 2019-12-25 16:11:03
categories:
- 数据结构与算法
tags:
- 旋转数组
- 查找
---

# 1 定义
将数组右移k位，移出的元素依次补充到数组的最左边，即得到旋转数组。

# 2 查找指定元素
将递增数组经过旋转操作后，查找某元素。
```cpp
int binarySearch(int a[], int left, int right, int e)
{
    while(left <= right)
    {
        int mid = (left + right) / 2;
        if(a[mid] == e)
        {
            return mid;
        }
        else if(a[left] < a[mid])
        {
            if(a[left] <= e && e < a[mid])
            {
                right = mid - 1;
            }
            else
            {
                left = mid + 1;
            }
        }
        else if(a[left] > a[mid])
        {
            if(a[mid] < e && e <= a[right])
            {
                left = mid + 1;
            }
            else
            {
                right = mid - 1;
            }
        }
        else
        {
            left = mid + 1;
        }
    }
    return -1;
}
```
**算法思想**
```
1) 二分查找，判断a[mid]和e是否相等，相等则返回，否则继续；
2) 如果a[left] < a[mid]，则表明[left, mid]升序，
   * 若a[left] <= e < a[mid]，则在[left, mid - 1]范围查找；
   * 否则，在[mid + 1, right]范围查找；
3) 如果a[left] > a[mid]，则表明[mid, right]升序，
   * 若a[mid] < e < a[right]，则在[mid + 1, right]范围查找；
   * 否则，在[left, mid - 1]范围查找；
4) 如果a[left] == a[mid]，则在[mid + 1, right]范围查找；
```

# 3 查找最小值
```cpp
int binarySearch(int a[], int left, int right)
{
    while(left < right)
    {
        if(a[left] < a[right]) return left;

        int mid = (left + right) / 2;
        if(a[left] < a[mid])
        {
            left = mid + 1;
        }
        else if(a[left] > a[mid])
        {
            right = mid;
        }
        else
        {
            left++;
        }
    }
    return left;
}
```
**算法思想**
```
1) 如果a[left] < a[right]，则a[left]为最小值；
2) 否则，二分查找
   * a[left] < a[mid]，最小值在[mid + 1, right]；
   * a[left] > a[mid]，最小值在[left, mid]；
   * a[left] = a[mid]，
        I. left = mid，a[left] >= a[right]，left++；
        II.left != mid，非严格增时，无法确定最小值在mid左边还是右边，left++；
```