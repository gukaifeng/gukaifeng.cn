

描述
---
传送门：http://acm.hdu.edu.cn/showproblem.php?pid=3555

给你一个数 N，问在从 1 ~ N 的数中有多少个数包含 "49"，其中 4 和 9 必须是连续的。
例如 1 ~ 500中，有 49, 149, 249, 349, 449, 490, 491, 492, 493, 494, 495, 496, 497, 498, 499 一共 15 个数包含 "49"，所以答案是 15。

思路
---
题目范围已经到了 long long 的边界了，暴力肯定不可行。
然后这个题看起来就是数位DP的简单题 😂。只要在dfs()函数中记录下上一个数是不是 4 就可以了，然后将算好的值存在 dp 数组中，记忆化搜索，这样便可以算出有多少个数不包含 "49"，再做一下减法就是答案了。唯一要注意的也就是这个题的范围了。
<!--more-->

代码
---
```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#define MAX 20
using namespace std;
typedef long long ll;
int T, digits[MAX];
ll N, dp[MAX][2];
ll dfs(int pos, int sta, bool islimit)
{
    if (pos == 0)
        return 1;
    if (!islimit && dp[pos][sta] != -1)
        return dp[pos][sta];
    int up = islimit ? digits[pos] : 9;
    ll cnt = 0;
    for (int i = 0; i <= up; ++i)
    {
        if (sta && i == 9)
            continue;
        cnt += dfs(pos - 1, i == 4, islimit && i == digits[pos]);
    }
    if (!islimit)
        dp[pos][sta] = cnt;
    return cnt;
}
ll solve(ll num)
{
    int pos = 0;
    while (num)
    {
        digits[++pos] = num % 10;
        num /= 10;
    }
    return dfs(pos, 0, true);
}
int main(void)
{
    memset(dp, -1, sizeof(dp));
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
    {
        scanf("%lld", &N);
        printf("%lld\n", N - solve(N) + 1);
    }
    return 0;
}
```