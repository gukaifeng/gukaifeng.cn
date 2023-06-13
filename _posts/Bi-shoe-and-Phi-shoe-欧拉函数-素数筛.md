
**传送门：**[LightOJ 1370 - Bi-shoe and Phi-shoe](https://vjudge.net/problem/LightOJ-1370)

**题目大意：**
有一些撑杆跳选手，每个人有一个幸运值。现在要给这些选手每人买一个竹竿，每个竹竿都有一个得分，分数是它的长度数值的欧拉函数值，比如竹竿长 $l$，那这个竹竿的得分就是 $\varphi(l)$，$\varphi(l)$ 是不大于 $l$ 的正整数中与 $l$ 互素的数的个数。[不知道什么是欧拉函数的同学看下这里](https://gukaifeng.me/2018/10/03/%E6%AC%A7%E6%8B%89%E5%87%BD%E6%95%B0/)。要求是给每个人买的竹竿的得分要大于或等于这个人的幸运值，然后一个竹竿的价格在数值上等于它的长度，问你在满足前面要求的前提下，最少要花多少钱买竹竿。

**解题思路：**
虽然这道题考察的是欧拉函数的性质，但并没有让你真的去算欧拉函数值。我们知道，一个数 $x$ 的欧拉函数值 $\varphi(x)$ 一定是小于 $x$ 的，反过来说，一个数的欧拉函数值为 $y$，那么这个数一定大于 $y$。这样对于每一个幸运值(假设为 $x$)，我们就可以从 $x+1$ 往后查找，第一个素数就是我们要的，因为一个素数的欧拉函数值就等于这个素数减一，所以从 $x+1$ 往后第一个素数就是欧拉函数值大于等于 $x$ 的最小的数了（其实具体怎么证明这个问题，我还不清楚，不过这么算是没错的，如果有谁知道相关证明或定理，欢迎留言呢）。因为题目范围只有 $10^6$，比这个范围大的第一个素数是 $1000003$，所以我们就用下素数筛，筛到大于等于 $1000003$ 的样子（我后面的代码筛到了 $1000004$），方便下后面找我们要的那个素数就可以了。<!--more-->

**通过代码：**
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll N = 1000005;
bool isprime[N];
ll prime[N], nprime, T, n, lucky, minS;
void doprime(void)
{
    nprime = 0;
    for (ll i = 0; i < N; ++i)
        isprime[i] = 1;
    for (ll i = 2; i < N; ++i)
    {
        if (isprime[i] == 1)
            prime[++nprime] = i;
        for (ll j = 1; j <= nprime && prime[j] * i < N; ++j)
        {
            isprime[prime[j] * i] = 0;
            if (i % prime[j] == 0)
                break;
        }
    }
}
int main(void)
{
    scanf("%lld", &T);
    doprime();
    for (ll t = 1; t <= T; ++t)
    {
        minS = 0;
        scanf("%lld", &n);
        while (n--)
        {
            scanf("%lld", &lucky);
            for (ll i = lucky + 1; true; ++i)
                if (isprime[i])
                {
                    minS += i;
                    break;
                }
        }
        printf("Case %lld: %lld Xukha\n", t, minS);
    }
    return 0;
}
```
