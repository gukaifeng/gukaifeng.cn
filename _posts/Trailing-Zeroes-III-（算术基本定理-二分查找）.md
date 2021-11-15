---
title: Trailing Zeroes (III)（算术基本定理+二分查找）
mathjax: true
date: 2018-10-08 21:35:55
updated: 2018-10-08 21:35:55
tags: [LightOJ,算术基本定理,二分查找,数论]
categories: [算法题解]
toc: true
---

**传送门：**[LightOJ 1138 - Trailing Zeroes (III)](https://vjudge.net/problem/LightOJ-1138)

**题目大意：**

给你一个数 $Q$，问你，阶乘结果末尾有 $Q$ 个 $0$ 的数，最小是多少。

$1\leqslant Q\leqslant 1e8$

**解题思路：**

我们先来考虑如何计算一个正整数的阶乘结果末尾 $0$ 的个数。

如果将一个正整数因式分解，那么其末尾的 $0$ 必然可以分解为 $2\times 5$，也就是说，因子中每一对 $2$ 和 $5$ 就对应着末尾的一个 $0$，因为 $2$ 肯定比 $5$ 多，所以我们只要统计因子中有多少个 $5$ 就是结果了。
根据[算术基本定理](https://gukaifeng.me/2018/10/04/%E7%AE%97%E6%9C%AF%E5%9F%BA%E6%9C%AC%E5%AE%9A%E7%90%86/)中，性质(5)：
$n!$ 的素因子分解中的素数 $p$ 的幂为 $[\frac{n}{p}]+[\frac{n}{p^2}]+[\frac{n}{p^3}]+\cdots$
我们把 $p=5​$ 代入，计算出结果即为所求。代码如下：

```cpp
int trailingZeroes(int n) {
    int cnt = 0;
    long long p = 5;
    while (n / p)
    {
        cnt += n / p;
        p *= 5;
    }
    return cnt;
}
```

我们现在能计算出了阶乘的末尾 $0$ 的个数，但是反过来算，还是麻烦。一时没有什么思路，然后我决定打个表，部分结果如下：<!--more-->

```shell
0	0
1	0
2	0
3	0
4	0
5	1
6	1
7	1
8	1
9	1
10	2
11	2
12	2
13	2
14	2
15	3
16	3
17	3
18	3
19	3
20	4
21	4
22	4
23	4
24	4
25	6
26	6
27	6
28	6
29	6
30	7
31	7
32	7
33	7
34	7
35	8
36	8
37	8
38	8
39	8
40	9
41	9
42	9
43	9
44	9
45	10
46	10
47	10
48	10
49	10
50	12
```

设第一列是正整数 $N$，第二列对应着 $N!$ 末尾 $0$ 的个数。

看出规律了吧，虽然有的 "$N!$ 末尾 $0$ 的个数" 不存在对应的 $N$，比如 $5$，$11$，但是每一个存在的个数，都是对应着五个连续的数。

然后接着在题目范围的边缘试探打表，发现了一组神奇的数据！

```shell
400000020	100000001
```

当 $N=4e8+20$ 时，末尾 $0$ 的个数刚好超过 $Q$ 的最大值。

这样就有一个很容易想到的思路了，我们可以在 $[0, 4e8+20]$ 这个范围内，通过二分查找，找到任意一个末尾 $0$ 的个数等于 $Q$ 的 $N$（要是最终一个也没找到，就说明是 impossible），然后最多再往前查找五个数，就能找到末尾 $0$ 的个数等于 $Q$ 且最小的数。

思路就是这样了，后面的就是怎么写代码的问题了，没什么好说的。



**通过代码：**

```cpp
#include <iostream>
#include <cstdio>
using namespace std;
int T, Q, ans;
int trailingZeroes(int n) {
    int cnt = 0;
    long long p = 5;
    while (n / p)
    {
        cnt += n / p;
        p *= 5;
    }
    return cnt;
}
int binary_chop(int q)
{
    int low = 0, high = 4e8+20;
    int mid, tz;
    while (low <= high)
    {
        mid = low + (high - low) / 2;
        tz = trailingZeroes(mid);
        if (tz == q)
            return mid;
        if (tz > q)
            high = mid - 1;
        else
            low = mid + 1;
    }
    return -1;
}
int main(void)
{
    cin >> T;
    for (int t = 1; t <= T; ++t)
    {
        scanf("%d", &Q);
        ans = binary_chop(Q);
        if (ans == -1)
            printf("Case %d: impossible\n", t);
        else
        {
            while (Q == trailingZeroes(--ans));
            printf("Case %d: %d\n", t, ans + 1);
        }
    }
    return 0;
}
```

