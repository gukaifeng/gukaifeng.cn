---
title: Crossing River（贪心）
mathjax: true
date: 2018-09-07 09:59:23
updated: 2018-09-07 09:59:23
tags: [POJ,贪心算法]
categories: [算法题解]
toc: true
---


**传送门：**[POJ 1700 - Crossing River](http://poj.org/problem?id=1700)

**题目大意：**
有 $n$ 个人要过一条河，但只有 $1$ 条最多同时载两个人的船，已知每个人的过河时间，两个人的过河时间取决于两个人中过河时间长的那一个。问最短需要多久可以让所有人都过河。

**解题思路：**
我们按过河时间从短到长将所有人排序，设时间第 $i$ 短的人为 $p_i$，过河时间为 $t_i$，所有人都过河了的最短时间为 $T_{min}$。
当 $n=1,2,3$ 时，$T_{min}$ 分别为 $t_1,t_2,t_1+t_2+t_3$，现在我们来考虑 $n\geq 4$ 的情况。
从待过河的人中选出 $4$ 人分为一组，假设待过河的还有 $n_{wait}$ 人，那么这 $4$ 个人包括 $p_1,p_2,p_{n_{wait}-1},p_{n_{wait}}$，也就是待过河的人中过河时间最短的、次短的、次长的、最长的。对于这 $4$ 个人，我们要实现一个结果，就是将 $p_{n_{wait}-1},p_{n_{wait}}$ 送到对岸，并且 $p_1,p_2$ 重新回到待过河的人中。重复此过程，直到待过河的人数为 $2$ 或 $3$。为了实现上述结果，有两种方案：
1\. $p_1,p_2$ 过河，$p_1$划船返回，$p_{n_{wait}-1},p_{n_{wait}}$ 过河，$p_2$ 划船返回，时间为 $t_1+2t_2+t_{n_{wait}}$；
2\. $p_1,p_{n_{wait}-1}$ 过河，$p_1$ 划船返回，$p_1,p_{n_{wait}}$ 过河，$p_1$ 划船返回，时间为 $2t_1+t_{n_{wait}-1}+t_{n_{wait}}$。
因为对于每一组人，两种方案的过河时间哪个更短未知，所以我们需要都计算出来，然后取小的那一个，$T_{min}=T_{min}+min(t_1+2t_2+t_{n_{wait}},2t_1+t_{n_{wait}-1}+t_{n_{wait}})$。
当剩余的待过河的人数为 $2$ 或 $3$ 时，转换为当 $n=2$ 和 $n=3$ 时的最短时间计算，分别为 $t_2$ 和 $t_1+t_2+t_3$，将其加到 $T_{min}$ 上，$T_{min}$ 即为最终结果。
<!--more-->

**通过代码：**
```cpp
#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <algorithm>
using namespace std;
const int MAXN = 1005;
int T, n, t[MAXN], T_min;
int main(void)
{
    scanf("%d", &T);
    while (T--)
    {
        scanf("%d", &n);
        for (int i = 1; i <= n; ++i)
            scanf("%d", &t[i]);
        sort(t + 1, t + 1 + n);
        if (n == 1)
            cout << t[1] << endl;
        else if (n == 2)
            cout << t[2] << endl;
        else if (n == 3)
            cout << t[1] + t[2] + t[3] << endl;
        else
        {
            if (n & 1) // 当 n 是奇数时最后一定剩 3 个人，是偶数时最后一定剩 2 个人
                T_min = t[1] + t[2] + t[3];
            else
                T_min = t[2];
            for (int i = n; i >= 4; i -= 2)
                T_min += min(t[1] + 2 * t[2] + t[i], 2 * t[1] + t[i - 1] + t[i]);
            cout << T_min << endl;
        }
    }
    return 0;
}
```