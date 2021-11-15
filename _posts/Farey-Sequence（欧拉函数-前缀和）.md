---
title: Farey Sequence（欧拉函数+前缀和）
mathjax: true
date: 2018-10-03 19:35:51
updated: 2018-10-03 19:35:51
tags: [POJ,欧拉函数,前缀和,数论]
categories: [算法题解]
toc: true
---


**传送门：**[POJ 2478 - Farey Sequence](http://poj.org/problem?id=2478)

**题目大意：**
给你一个数 $n$，让你求一个特定序列 $Fn$ 中的元素有多少个。
这个序列是这样的，有很多个分数，分子小于分母，分母不大于 $n$，且分子分母的最大公约数为 $1$，这些分数从小到大排列（其实排列这个条件没鸟用）。例如：
$
F2 = \\{\frac{1}{2}\\} \\\
F3 = \\{\frac{1}{3},\ \frac{1}{2},\ \frac{2}{3}\\} \\\
F4 = \\{\frac{1}{4},\ \frac{1}{3},\ \frac{1}{2},\ \frac{2}{3},\ \frac{3}{4}\\} \\\
F5 = \\{\frac{1}{5},\ \frac{1}{4},\ \frac{1}{3},\ \frac{2}{5},\ \frac{1}{2},\ \frac{3}{5},\ \frac{2}{3},\ \frac{3}{4},\ \frac{4}{5}\\} 
$

**解题思路：**
我们可以把问题转化为求对于 $i$ ($i\in [2,n]$)，在不大于 $i$ 的正整数中与 $i$ 互素的数的个数，这一看就是[欧拉函数](https://gukaifeng.me/2018/10/03/%E6%AC%A7%E6%8B%89%E5%87%BD%E6%95%B0/)嘛，然后把各个 $i$ 算出来的值求和就是答案。于是乎，我们就可以算出从 $2$ 到 $n$ 每一个数的欧拉函数值，然后再求前缀和存到数组中，这样对于每一个输入，我们直接从数组中取数就可以了。要注意下 $1$ 是不会有做分母的时候的，所以前缀和从 $n=2$ 开始加。<!--more-->

**通过代码：**
```cpp
#include <iostream>
using namespace std;
typedef long long ll;
const ll maxn = 1000005;
ll phi[maxn], n;
int main(void)
{
    for (int i = 1; i < maxn; ++i)
        phi[i] = i;
    for (int i = 2; i < maxn; i += 2)
        phi[i] /= 2;
    for (int i = 3; i < maxn; i += 2)
        if (phi[i] == i)
            for (int j = i; j < maxn; j += i)
                phi[j] = phi[j] / i * (i - 1);
    for (int i = 3; i < maxn; ++i)
        phi[i] += phi[i - 1];
    while (cin >> n && n)
        cout << phi[n] << endl;
    return 0;
}
```
