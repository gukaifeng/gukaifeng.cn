---
title: 快速幂模m算法
mathjax: true
date: 2018-09-07 20:34:31
updated: 2018-09-07 20:34:31
tags: [快速幂,整数快速幂,矩阵快速幂,快速幂模m算法,乘法取模]
categories: [基础算法]
---

当数据过大且需要取模的时候，[快速幂](https://gukaifeng.cn/posts/kuai-su-mi/)便无法再满足需求了，我们这里介绍快速幂模 m 算法，通常使用 long long 数据类型。

<!--more-->

代码实现：  
1\. 整数快速幂模m算法
计算 $a^n\%m$，函数 quick_mod() 返回计算结果。
```cpp
long long quick_mod(long long a, long long n, long long m)
{
    long long ans = 1;
    while (n)
    {
        if (n & 1)
            ans = (ans * a) % m;
        n >>= 1;
        a = a * a % m;
    }
    return ans;
}
```

2\. 矩阵快速幂模m算法
设 $A$ 为 MAXN 阶方阵(只有方阵可以使用快速幂)，计算 $A^n\%m$，函数 mat_quick_mod() 返回计算结果矩阵。
```cpp
const int MAXN = 10; // 方阵的阶+1
typedef struct // 矩阵下标从 1 开始
{
    long long m[MAXN][MAXN];
}Matrix;
Matrix mat_mul(Matrix a, Matrix b, long long m) // 矩阵乘法
{
    Matrix c;
    for (int i = 1; i <= MAXN; ++i)
        for (int j = 1; j <= MAXN; ++j)
        {
            c.m[i][j] = 0;
            for (int k = 1; k <= MAXN; ++k)
                c.m[i][j] = (c.m[i][j] + a.m[i][k] * b.m[k][j]) % m;
        }
    return c;
}
Matrix mat_quick_mod(Matrix a, long long n, long long m)
{
    Matrix c = {0};
    for (int i = 1; i <= MAXN; ++i)
        c.m[i][i] = 1;
    while (n)
    {
        if (n & 1)
            c = mat_mul(c, a);
        n >>= 1;
        a = mat_mul(a, a);
    }
    return c;
}
```

3\. 乘法取模
如果操作数据和模 $m$ 都非常大（但依然在 long long 内），乘法操作在取模以前就会超出 long long 范围，那么使用 mul_mod() 函数进行乘法取余运算。
将上述代码中的乘法取模操作用 mul_mod() 函数替换即可。乘法取模代码也可以单独使用。
```cpp
long long mul_mod(long long a, long long b, long long m) // a * b % m
{
    long long ret = 0;
    while (b)
    {
        if (b & 1)
            ret = (ret + a) % m;
        b >>= 1;
        a = (a << 1) % m;
    }
    return ret;
}
```