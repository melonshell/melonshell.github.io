---
title: select问题
date: 2020-01-20 12:56:43
categories:
- 数据结构与算法
tags:
- 查找第k大数
---

# 1 问题描述
给定一个集合$S$，$|S|$表示$S$元素的个数，找到$S$中第$k$大的元素，当$k=|S|/2$时，即中位数；

# 2 堆
维护一个k个元素的最小堆，BuildHeap复杂度为$O(k)$，然后遍历剩下$n-k$个元素，如果比堆顶元素大，则替换堆顶元素，执行percolate down操作，时间复杂度为$O((n - k)logk)$，总时间复杂度为$O(k + (n - k)logk)$，当$k=n/2$时，时间复杂度为$O(logn)$；

# 3 划分
取$s \in S$，将$S$按$s$划分为两部分$s_l$和$s_r$，其中$s_l$中的元素均小于等于$s$，$s_r$中的元素均大于$s$，
如果$|s_l| >= k$，则第$k$大数必定在$|s_l|$中；
如果$|s_l| == k - 1$，则$s$为第$k$大数；
如果$|s_l| < k - 1$，则第$k$大数必定在$s_r$中，且是$s_r$中第$k - |s_l| - 1$大元素；

## 3.1 实现
选取第一个元素为基准元素，为保证划分均匀(可以考虑所有元素相等的情况)，i和j遇到和pivot相等的元素均停止遍历：
```cpp
int partition(int a[], int left, int right)
{
    int i = left + 1, j = right;
    int pivot = a[left];

    while(i < j)
    {
        while(i <= j && a[i] < pivot) i++;
        while(i <= j && a[j] > pivot) j--;

        if(i < j)
        {
            swap(a[i], a[j]);
        }
        else
        {
            break;
        }
    }

    swap(a[left], a[j]);
    return j;
}

int findKth(int a[], int left, int right, int k)
{
    int p = partition(a, left, right);
    int left_len = p - left;
    if(left_len >= k)
    {
        return findKth(a, left, p - 1, k);
    }

    if(left_len == k - 1)
    {
        return a[p];
    }

    return findKth(a, p + 1, right, k - left_len - 1);
}

```
## 3.2 复杂度分析
1) 最坏情况
$T(n) = T(n - 1) + cn$
于是
$T(n) = T(n - 1) + cn$
$T(n - 1) = T(n - 2) + c(n - 1)$
...
$T(2) = T(1) + c1$

依次相加：
$T(n) = O(n)$

2) 平均情况
$T(n) = T(\frac{n}{2}) + cn$
于是
$\frac{T(n)}{n} = \frac{T(\frac{n}{2})}{n} + c$
$\frac{T(n)}{n} = \frac{1}{2}\frac{T(\frac{n}{2})}{\frac{n}{2}} + c$
令$A_n = \frac{T(n)}{n}$，则：
$A_n = \frac{1}{2}A_\frac{n}{2} + c$
故有
$A_n - 2C = \frac{1}{2}(A_\frac{n}{2} - 2c)$
令$B_n = A_n - 2C$，则：
$B_n = \frac{1}{2}B_\frac{n}{2}$
于是
$\frac{B_n}{B_\frac{n}{2}} = \frac{1}{2}$
$\frac{B_\frac{n}{2}}{B_\frac{n}{4}} = \frac{1}{2}$
...
$\frac{B_4}{B_2} = \frac{1}{2}$
$\frac{B_2}{B_1} = \frac{1}{2}$
依次相乘：
$\frac{B_n}{B_1} = {(\frac{1}{2})}^{\log(n)}$
$B_n = \frac{c}{n}$
故
$A_n = \frac{c}{n} + 2C$
$T_n = c + 2Cn = O(n)$
