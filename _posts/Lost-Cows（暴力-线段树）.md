---
title: Lost Cows（暴力/线段树）
mathjax: true
date: 2018-10-24 16:15:06
updated: 2018-10-24 16:15:06
tags: [POJ,线段树]
categories: [算法题解]
toc: true
---

描述
---
传送门：[POJ 2182 - Lost Cows](http://poj.org/problem?id=2182)

有一些奶牛，从 $1$ 到 $N$ 编号，现在这些奶牛站成一行，但是没有按顺序，我们只知道一个条件：
对于某一只奶牛，站在它前面并且编号比它小的奶牛的数量。
要求计算出每只奶牛的编号。

思路
---
根据题意，我们考虑最后一只奶牛（设为 $N$），我们知道站在它前面的并且编号比它小的奶牛的数目（设为 $Pre[N]$ ），因为所有的其他奶牛已经都在它前面了，所以 $Pre[N]$ 也就是所有奶牛中编号比它小的数目，那么这最后一只奶牛，编号就是 $Pre[N]+1$ 。

然后从后往前推，第 $N-1$ 只奶牛，站在它前面的编号比它小的奶牛数目是 $Pre[N-1]$ ，因为我们是从后往前推的，那么站在这第 $N-1$ 只奶牛后面的，编号肯定都已经确定了，那么从编号 $1$ 开始往后查找，第 $Pre[N-1]+1$ 个没有被确定的编号，就是它的编号。以此类推，直到所有奶牛的编号都确定下来。

因为题目数据范围较小，只有 $8000$， $O(n^2)$ 的暴力算法就可以过了，但是秉着学习的态度，还是用线段树解了一发，万一遇到数据很大的类似题目呢~！
<!--more-->

代码 1 (暴力)
---
```cpp
#include <iostream>
#include <cstdio>
#define MAX 8005
using namespace std;
int N, pre[MAX];
int Cows[MAX], sign[MAX];
int main(void)
{
    scanf("%d", &N);
    for (int i = 2; i <= N; ++i)
        scanf("%d", &pre[i]);
    Cows[N] = pre[N] + 1;
    sign[pre[N] + 1] = 1;
    for (int i = N - 1; i >= 1; --i)
    {
        int cnt = 0, j = 1;
        while (cnt < pre[i] + 1)
            if (sign[j++] == 0)
                ++cnt;
        sign[j - 1] = 1;
        Cows[i] = j - 1;
    }
    for (int i = 1; i <= N; ++i)
        printf("%d\n", Cows[i]);
    return 0;
}
```

代码 2 (线段树)
---
```cpp
#include <cstdio>
#define MAX 8005
using namespace std;
struct node 
{
    int L, R;
    int Cnt;
    int Mid(void)
    {
        return (L + R) >> 1;
    }
}Tree[MAX << 2];
void Build(int rt, int l, int r)
{
    Tree[rt].L = l;
    Tree[rt].R = r;
    Tree[rt].Cnt = r - l + 1;
    if (l != r)
    {
        Build(rt << 1, l, Tree[rt].Mid());
        Build(rt << 1 | 1, Tree[rt].Mid() + 1, r);
    }
}
void Modify(int rt, int i)
{
    --Tree[rt].Cnt;
    if (Tree[rt].L == Tree[rt].R)
        return ;
    if (Tree[rt].Mid() >= i)
        Modify(rt << 1, i);
    else
        Modify(rt << 1 | 1, i);
}
int Query(int rt, int k)
{
    if (Tree[rt].L == Tree[rt].R)
    {
        Modify(1, Tree[rt].L);
        return Tree[rt].L;
    }
    if (Tree[rt << 1].Cnt >= k)
        return Query(rt << 1, k);
    else
        return Query(rt << 1 | 1, k - Tree[rt << 1].Cnt);
}
int N, Pre[MAX], Cows[MAX];
int main(void)
{
    scanf("%d", &N);
    Build(1, 1, N);
    for (int i = 2; i <= N; ++i)
        scanf("%d", &Pre[i]);
    Cows[N] = Pre[N] + 1;
    Modify(1, Pre[N] + 1);
    for (int i = N - 1; i >= 1; --i)
        Cows[i] = Query(1, Pre[i] + 1);
    for (int i = 1; i <= N; ++i)
        printf("%d\n", Cows[i]);
    return 0;
}
```