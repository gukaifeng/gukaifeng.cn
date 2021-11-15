---
title: 'LeetCode: 172. 阶乘后的零'
mathjax: true
date: 2019-04-26 14:50:53
updated: 2019-04-26 14:50:53
tags: [LeetCode, 数论]
categories: [算法题目]
toc: true
---



原题链接（英文）：https://leetcode.com/problems/factorial-trailing-zeroes/description/
原题链接（中文）：https://leetcode-cn.com/problems/factorial-trailing-zeroes/description/


如果将一个正整数因式分解，那么其末尾的 $0$ 必然可以分解为 $2\times 5$。
也就是说，因子中每一对 $2$ 和 $5$ 就对应着末尾的一个 $0$，因为 $2$ 肯定比 $5$ 多。
综上，所以我们只要统计因子中有多少个 $5$ 就是结果了。
根据[算术基本定理](https://gukaifeng.me/2018/10/04/%E7%AE%97%E6%9C%AF%E5%9F%BA%E6%9C%AC%E5%AE%9A%E7%90%86/)中性质(5)：$n!$ 的素因子分解中的素数 $p$ 的幂为 $[\frac{n}{p}]+[\frac{n}{p^2}]+[\frac{n}{p^3}]+\cdots$
我们把 $p=5$ 代入，计算出结果即为所求。<!--more-->

```C
int trailingZeroes(int n)
{
    int count_zeros = 0;
    long long p = 5;
    
    while (n / p)
    {
        count_zeros += n / p;
        p *= 5;
    }
    
    return count_zeros;
}
```