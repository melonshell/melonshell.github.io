---
title: 判断单链表是否有环
date: 2020-01-15 08:18:04
categories:
- 数据结构与算法
tags:
- 单链表
- 环
---

# 1 问题
给定一个单链表，判断是否存在环；

# 2 思路一
从第一个节点开始遍历链表，并用map<Node*, int>(key为遍历到的节点，val表示节点的顺序递增编号)保存遍历到的节点，在遍历到当前节点时：
查询map，当前节点是否已经在map中，如查询到则说明有环；如没查询到则编号递增，将当前节点插入map；
遍历至NULL，则说明链表无环；
链表有环，则第一个在map中查到的节点必定是环的入口节点，节点的最大编号 - 入口节点编号 + 1 = 环的长度，入口节点编号 - 1 - 非环部分长度；
```cpp
bool hasCircle(Node* n, int& circle_len, int& non_circle_len)
{
    int no = 0;
    map<Node*, int> nodes;
    map<Node*, int>::const_iterator iter;
    while(n)
    {
        iter = nodes.find(n);
        if(iter == nodes.end())
        {
            no++;
            node[n] = no;
        }
        else
        {
            break;
        }
        n = n->next;
    }

    if(n)
    {
        circle_len = no - iter->second + 1;
        non_circle_len = iter->second - 1;
        return true;
    }
    else
    {
        circle_len = 0;
        non_circle_len = no;
    }
    return false;
}
```
# 3 思路二
一个快指针和慢指针同时从第一个节点出发，快指针每次走两步，慢指针每次走一步；如果有环，则快慢指针必定相遇。
设非环部分长度为$L_1$，环长为$L_2$，第一个入环节点为$N$，慢指针走$s$步后快慢指针相遇，则相遇点$r$满足：
(s-$L_1$)$\equiv$r mod($L_2$)
(2s-$L_1$)$\equiv$r mod($L_2$)
则：
s$\equiv$0 mod($L_2$)
即$L_2$ | $s$时，快慢指针相遇，代入得相遇点为$r$=$t$$L_2$ - $L_1$，其中$t$为整数，此时放一个慢指针在链表初始位置，一个慢指针在相遇点，两指针同时出发，则必相遇于环的入口点。
```cpp
bool hasCircle(Node* n, int& circle_len, int& non_circle_len)
{
    Node *slow = n, *fast = n, *slow2 = n;
    non_circle_len = 0, circle_len = 0;

    while(fast && fast->next)
    {
        slow = slow->next;
        fast = fast->next->next;
        non_circle_len += 2;

        if(slow == fast)
        {
            break;
        }
    }

    if(!fast || !fast->next)
    {
        circle_len = 0;

        if(fast) non_circle_len++;
        return false;
    }

    non_circle_len = 0;
    while(true)
    {
        non_circle_len++;
        slow = slow->next;
        slow2 = slow2->next;
        if(slow == slow2)
        {
            break;
        }
    }

    while(true)
    {
        non_circle_len++;
        slow = slow->next;
        if(slow == slow2)
        {
            break;
        }
    }

    return true;
}
```