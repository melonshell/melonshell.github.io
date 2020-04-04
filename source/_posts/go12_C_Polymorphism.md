---
title: C语言实现多态
date: 2020-04-04 22:24:15
categories:
- go语言
tags:
- C语言
- 多态
---

```c
//父类虚表
typedef struct vtlbA
{
    void (*pfun1)();
}vtlbA;

//子类虚表
typedef struct vtlbB
{
    vtlbA vtbl;
    void (*pfun2_B)();
}vtlbB;
```
虚表内存布局如下：
  ![虚表内存布局](/pic/go12_1.png)

```c
//父类虚函数
void fun1()
{
    printf("父类fun1\n");
}

//子类重写父类虚函数fun1
void fun1_B()
{
    printf("子类重写fun1\n");
}

//子类特有函数
void fun2_B(int)
{
    printf("子类特有fun2\n");
}

//父类
typedef struct A
{
    vtblA* pvtl;// 父类虚表指针
    int a;      // 父类数据成员
}

//子类
typedef struct B
{
    A base_a; // 从父类继承而来的基类A
    int b;    // 子类数据成员
}B;
```
类内存布局如下：
  ![类内存布局](/pic/go12_2.png)

```c
// 父类虚表结构
vtblA g_vtbl_A = {&fun1};

// 子类虚表结构
vtblB g_btbl_B = { {&fun1_B}, &fun2_B };

// 父类构造函数
void init_A(A* pa)
{
    pa->a = 10;
    pa->pvtl = &g_vtbl_A;
}

// 子类构造函数
void init_B(B* pb)
{
    init_A((A*)pb);
    pb->base_a.pvtl = (vtblA*)&g_btbl_B;
    pb->b = 20;
}
```
构造函数执行之后内存布局如下：
  ![构造函数执行之后内存布局](/pic/go12_3.png)

```c
// 测试多态函数
void dosomething(A* p)
{
    // 如果p指向子类对象那么输出结果就是重写后的函数pfun1
    vtblA* pvtbl = (vtblA*)p->pvtl;
    (*pvtbl).pfun1();
}

int main()
{
    // 定义子类对象并构造
    B b;
    init_B(&b);
    // 调用父类自己的函数
    vtblB* pvtvl = (vtblB*)b.base_a.pvtl;
    (*pvtvl).fun2_B(5);
    // 定义父类指针
    A* pa = (A*)&b;
    // 测试多态
    dosomething(pa);
    return 0;
}
```