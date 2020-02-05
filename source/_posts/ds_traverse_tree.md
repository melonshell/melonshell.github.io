---
title: 非递归遍历二叉树
date: 2020-01-29 16:37:23
categories:
- 数据结构与算法
tags:
- 非递归遍历
---

定义树节点结构体：
struct Node
{
    Node* left;
    Node* right;
    int   val;
}

void visit(Node* n)
{
    if(!n) return;
    cout << n->val << endl;
}

# 1 先序遍历
用栈模拟递归调用；
首先根节点入栈；
栈顶元素出栈；
栈顶右孩子入栈；
栈顶左孩子入栈；

```cpp
void preOrder(Node* r)
{
    if(r == NULL) return;
    stack<Node*> s;
    s.push(r);

    while(!s.empty())
    {
        Node* t = s.top();
        s.pop();
        visit(t);
        if(t->right) s.push(t->right);
        if(t->left) s.push(t->left);
    }
}
```

# 2 中序遍历
根结点入栈，如果有左孩子，则左孩子入栈，一直到左孩子为空；
栈顶元素出栈；
如果栈顶元素有右孩子，右孩子入栈，如果右孩子有左孩子，依次入栈，直到左孩子为空；
```cpp
void inOrder(Node* r)
{
    stack<Node*> s;
    while(r) { s.push(r); r = r->left; }

    while(!s.empty())
    {
        Node* t = s.top();
        s.pop();
        visit(t);

        if(t->right)
        {
            Node* r = t->right;
            while(r) { s.push(r); r = r->left; }
        }
    }
}
```

# 3 后序遍历
**思路1：双栈**
后序遍历为：左孩子-->右孩子-->根
先序遍历为：根-->左孩子-->右孩子
逆先序遍历为：根-->右孩子-->左孩子
可以发现逆先序遍历和后序遍历正好相反

```cpp
void postOrder(Node* r)
{
    stack<Node*> s;
    stack<Node*> rs;
    if(r) s.push(r);
    while(!s.empty())
    {
        Node* t = s.top();
        s.pop();
        rs.push(t);
        if(t->left) s.push(t->left);
        if(t->right) s.push(t->right)；
    }

    while(!rs.empty()) rs.pop();
}

```

**思路2：用一个栈，记录上次遍历的节点**
类似于中序遍历，记录上次遍历的节点
```cpp
void postOrder(Node* r)
{
    if(!r) return;

    stack<Node*> s;
    Node* last = NULL;
    s.push(r);
    while(!s.empty())
    {
        Node* t = s.top();
        if(t->left != last && t->right != last && t->left)
        {
            s.push(t->left);
        }
        else if(t->right != last && t->right)
        {
            s.push(t->right);
        }
        else
        {
            s.pop();
            visit(t);
            last = t;
        }
    }
}
```

# 4 层次遍历
```cpp
void breadthTraverse(Node* r)
{
    if(!r) return;
    queue<Node*> q;
    q.push_back(r);
    while(!q.empty())
    {
        Node* t = q.front();
        visit(t);
        q.pop_front();
        if(t->left) q.push_back(t->left);
        if(t->right) q.push_back(t->right);
    }
}
```