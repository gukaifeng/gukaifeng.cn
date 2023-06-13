
<script type='text/javascript' src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_CHTML'></script>

描述
---
传送门：http://poj.org/problem?id=2352

在笛卡尔坐标系上，有很多的猩猩，对于某一只猩猩，它的等级等于位置在它左下方的猩猩的数目。
换句话说，设一个猩猩的坐标为 $(x_0,y_0)$，那它的等级就是所有坐标为 $(x,y)$ 的猩猩的数目，其中 $x\leqslant x_0, y\leqslant y_0$ 。
一共有 $N$ 只猩猩，问你从等级 $0$ 到 $N-1$，各有多少只猩猩。

思路
---
考虑用树状数组求解，首先将坐标排序，按纵坐标从小到大，纵坐标相同时横坐标从小到大。虽然本题已经排好了，但我们仍然有必要知道为何要这样排，下面解释。

我们按照排列好的顺序依次计算，就能保证当计算到某一只猩猩的时候，满足计算它等级条件的所有猩猩都已经提前计算完了，假设这只猩猩坐标是 $(x_0,y_0)$，那它的等级就是 $Sum(x_0)$，然后再将这只猩猩也加入到树状数组当中，即 $Modify(x_0)$。我们只需要在计算等级的时候记录一下就可以了。

另外这题设置了一个小坑，我们知道树状数组和线段树这类数据结构在计算的时候，下标是必须从 1 开始的。
在本题中，我们以 $x$ 坐标建树，但题目中给出的 $x$ 是可以等于 $0$ 的，所以我们需要将所有的 $x$ 坐标加 $1$，把所有猩猩右移一位，计算结果保持不变。
<!--more-->

代码
---
```cpp
#include <iostream>
#include <cstdio>
#define MAXN 15005
#define MAXX 32005
#define lowbit(x) x & -x
using namespace std;
int N, Tree[MAXX];
class Star
{
public:
    int x, y;
}Stars[MAXN];
int Cnt[MAXN];
int MaxX;
void Modify(int i)
{
    while (i <= MaxX)
    {
        ++Tree[i];
        i += lowbit(i);
    }
}
int Sum(int i)
{
    int res = 0;
    while (i > 0)
    {
        res += Tree[i];
        i -= lowbit(i);
    }
    return res;
}
// bool operator<(Star a, Star b)   // 题目排好序了，那就不需要这个了，下面 sort() 同理
// {
//    if (a.y != b.y)
//        return a.y < b.y;
//    return a.x < b.x;
//}
int main(void)
{
    scanf("%d", &N);
    for (int i = 1; i <= N; ++i)
    {
        scanf("%d%d", &Stars[i].x, &Stars[i].y);
        ++Stars[i].x; // x 坐标右移
        MaxX = max(MaxX, Stars[i].x);
    }
    // sort(Stars + 1, Stars + 1 + N);
    for (int i = 1; i <= N; ++i)
    {
        ++Cnt[Sum(Stars[i].x)];
        Modify(Stars[i].x);
    }
    for (int i = 0; i <= N - 1; ++i)
        printf("%d\n", Cnt[i]);
    return 0;
}
```