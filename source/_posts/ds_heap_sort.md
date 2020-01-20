---
title: 堆和堆排序
date: 2020-01-19 18:58:57
categories:
- 数据结构与算法
tags:
- 堆
- 堆排序
---

# 1 定义
堆是一棵完全二叉树，$N$个节点的高度为$O(logN)$；

# 2 性质
最大堆：节点值不小于孩子节点的值，子树是堆；
最小堆：节点值不大于孩子节点的值，子树是堆；

下面以最小堆为例。

# 3 插入操作
```cpp
//在heap堆数组的第j个位置插入v
void percolate_up(int heap[], int v, int j)
{
    while(j > 0)
    {
        int parent = (j - 1) / 2;
        if(heap[parent] > v)
        {
            heap[j] = heap[parent];
            j = parent;
        }
        else
        {
            break;
        }
    }

    heap[j] = v;
}
```
**算法思想**
从j开始向上调整，由于初始heap[j]已经保存在变量v中，因此j位置为一个空位；
如果heap[parent]大于v，则将heap[parent]保存在j位置，parent形成空位；

# 4 删除堆顶元素
删除堆顶元素，将最后一个元素置于堆顶，对堆顶元素执行percolate down操作，下面是更一般化的从j执行percolate down操作的代码：
```cpp
void percolate_down(int heap[], int j, int heap_len)
{
    int v = heap[j];

    while(2 * j + 1 < heap_len)
    {
        int i = 2 * j + 1;
        int t = heap[i];
        if(i + 1 < heap_len)
        {
            if(t > heap[i + 1])
            {
                t = heap[i + 1];
                i = i + 1;
            }
        }

        if(heap[i] < v)
        {
            heap[j] = heap[i];
            j = i;
        }
        else
        {
            break;
        }
    }

    heap[j] = v;
}
```
**算法思想**
从j开始向下调整；
找到左右孩子的最小者，注意判断左右孩子是否存在；
保存heap[j]至变量v，形成空位；
若左右孩子最小者i大于v，则保存heap[i]至heap[j]，i位置形成空位；

# 5 堆排序
## 5.1 BuildHeap
给定一个任意数组，将其堆化，叶子节点满足堆条件，从第一个非叶子节点开始执行percolate_down操作；
一个具有$n$个节点的完全二叉树，叶子节点个数为多少？
设叶子节点个数为$n_0$，度为1的节点个数为$n_1$，度为2的节点个数为$n_2$，则：
$n = n_0 + n_1 + n_2$
$n = 2n_2 + n_1 + 1$
故有$n_0 = n_2 + 1 = (n + 1 - n_1) / 2$
由于是完全二叉树，故$n_1 \in {0, 1}$，
当$n_1 = 0$时，$n_0 = (n + 1) / 2$，非叶子节点数目为$(n - 1) / 2$；
当$n_1 = 1$时，$n_0 = n / 2$，非叶子节点数目为$n / 2$；

```cpp
void build_heap(int heap[], int heap_len)
{
    for(int i = heap_len / 2; i >= 0; i--)
    {
        percolate_down(heap, i, heap_len);
    }
}
```

## 5.2 HeapSort
数组堆化后，不断删除堆顶最小元素并移至数组末尾
```cpp
void heap_sort(int heap[], int heap_len)
{
    build_heap(heap, heap_len);

    while(heap_len > 0)
    {
        int t = heap[0];
        heap[0] = heap[heap_len - 1];
        heap[heap_len - 1] = t;
        heap_len--;
        percolate_down(heap, 0, heap_len);
    }
}
```