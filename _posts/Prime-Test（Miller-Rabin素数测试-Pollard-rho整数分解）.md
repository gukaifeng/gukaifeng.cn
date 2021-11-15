---
title: Prime Test（Miller-Rabin素数测试+Pollard rho整数分解）
mathjax: true
date: 2018-10-07 17:25:39
updated: 2018-10-07 17:25:39
tags: [POJ,素数测试,因数分解,数论]
categories: [算法题解]
toc: true
---

**传送门：**[POJ 1811 - Prime Test](http://poj.org/problem?id=1811)

**题目大意：**

给你一个数 $N$ $(2\leqslant N\leqslant 2^{54})$ 判断 $N$是否为素数，是的话直接输出 "Prime"，不是的话输出最小质因数。

**解题思路：**

看到这个数据范围，我就知道，常规算法肯定是行不通了=。=！所以就得用[Miller-Rabin素数测试](https://gukaifeng.me/2018/09/06/%E7%B4%A0%E6%95%B0%E6%B5%8B%E8%AF%95/#4-Miller-Rabin%E7%B4%A0%E6%95%B0%E6%B5%8B%E8%AF%95)来判断是否是素数了，然后如果是的话，直接输出"Prime"就行了，不是就用[Pollard rho整数分解方法](https://gukaifeng.me/2018/10/07/%E6%95%B4%E6%95%B0%E5%88%86%E8%A7%A3%EF%BC%88%E5%9B%A0%E6%95%B0%E5%88%86%E8%A7%A3%EF%BC%89/#3-Pollard-rho-%E6%95%B4%E6%95%B0%E5%88%86%E8%A7%A3%E6%96%B9%E6%B3%95)算质因数，再找出所有质因数中最小的那一个就好了。要注意的地方也就是在进行Miller-Rabin素数测试的时候，要使用二次探测定理，排除卡迈克尔数（一个合数 $n$，若对所有满足 $gcd(b,n)=1$ 的正整数 $b$ 都有 $b^{n-1}\equiv 1\pmod n​$ 成立，则称之为卡迈克尔数），因为卡迈克尔数会导致素数测试结果出错。<!--more-->

**通过代码：**

```cpp
#include <iostream>
#include <cstdlib>
using namespace std;
typedef long long ll;
const int maxn = 1000;
const int N = 5; // 测试次数
const int C = 200; // 指定 find() 函数参数2
int T;
ll factor[maxn], cnt;	// 素因子和素因子的数目，下标从 1 开始
ll mini, m;
ll gcd(ll a, ll b)
{
    return b == 0 ? a : gcd(b, a % b);
}
ll random(ll n)
{
    return (ll)((double)rand() / RAND_MAX * n + 0.5);
}
ll multi(ll a, ll b, ll m) // a * b % m
{
    ll ret = 0;
    while (b)
    {
        if (b & 1)
            ret = (ret + a) % m;
        b >>= 1;
        a = (a << 1) % m;
    }
    return ret;
}
ll quick_mod(ll a, ll b, ll m)
{
    ll ans = 1;
    while (b)
    {
        if (b & 1)
            ans = multi(ans, a, m);
        b >>= 1;
        a = multi(a, a, m);
    }
    return ans;
}
bool Witness(ll a, ll n)
{
    ll m = n - 1;
    int j = 0;
    while (!(m & 1))
    {
        ++j;
        m >>= 1;
    }
    ll x = quick_mod(a, m, n);
    if (x == 1 || x == n - 1)
        return false;
    while (j--)
    {
        x = x * x % n;
        if (x == n - 1)
            return false;
    }
    return true;
}
bool miller_rabin(ll n)
{
    if (n == 2)
        return true;
    if (n < 2 || !(n & 1))
        return false;
    for (int i = 1; i <= N; ++i)
    {
        ll a = random(n - 2) + 1;
        if (Witness(a, n))
            return false;
    }
    return true;
}
ll pollard_rho(ll n, int c)
{
    ll x, y, d, i = 1, k = 2;
    x = random(n - 1) + 1;
    y = x;
    while (true)
    {
        ++i;
        x = (multi(x, x, n) + c) % n;
        d = gcd(y - x, n);
        if (1 < d && d < n)
            return d;
        if (y == x)
         	return n;
        if (i == k)
        {
            y = x;
            k <<= 1;
		}
    }
}
void find(ll n, int k)
{
    if (n == 1)
        return ;
    if (miller_rabin(n))
    {
        factor[++cnt] = n;
		mini = min(n, mini);
        return ;
    }
    ll p = n;
    while (p >= n)
        p = pollard_rho(p, k--);
    find(p, k);
    find(n / p, k);
}
int main(void)
{
    cin >> T;
    while (T--)
    {
        cin >> m;
        if (miller_rabin(m))
            cout << "Prime" << endl;
        else
        {
            mini = __LONG_LONG_MAX__;
            find(m, C);
            cout << mini << endl;
        }
    }
    return 0;
}
```

