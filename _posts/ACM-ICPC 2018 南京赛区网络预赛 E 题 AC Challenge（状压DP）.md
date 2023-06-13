
**传送门：**[ACM-ICPC 2018 南京赛区网络预赛 E 题](https://nanti.jisuanke.com/t/30994)  

**题目大意：**
有 $n$ 道题目，Dlsj 已经知道了每一道题的答案，可以直接提交并 AC，每道题的提交时间间隔为 1 分钟，提交题目 $i$ 的得分为 $t×a_i+b_i$，($t$ 为提交时间，$a_i, b_i$已知)。想要提交某道题需要已经 AC 了特定的某几个题，否则不可以提交。例如题目告诉你说，提交第 5 题前，必须要先过了第 2 题和第 6 题，那么想要提交第 5 题，就需要第 2 题、第 6 题已经 AC。可以有些题目不做，因为 $a_i, b_i$ 可能是负数，做了反而分数更低，当然如果有题目不做，那么需要这个题已经 AC 才能提交的题目也做不了。问 Dlsj 能达到的最高分是多少。    

<!--more-->

**解题思路：**
因为 $n$ 最大为 20，一道题只有两个状态（已经 AC 了，或者还没做），所以一共最多有 $2^{20}$ 个状态。对于每个状态，我们可以用一个 int 类型的数（二进制）来表示，右数第 i 位为 1 表示题目 $i$ 已经 AC 了，为 0 表示题目 $i$ 还没做。所以我们需要一个大小为 $2^{20}$ 的 int 类型数组，下标值为状态，数据值为到达这个状态时，最多能得多少分，于是题目便可以用 状压DP 来求解了，遍历从 $1$ 到 $2^n$(代码中通常写作 1 << n) 的所有状态，DP 递推就可以了。

**通过代码：**
```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 20;
struct node
{
    int a, b, pre;                           // pre 为提交该题需要已经 AC 的题目状态
} p[MAXN];
int n, si, pi, ans;
int mp[1 << MAXN];                           // mp[i]为到达i状态时，获得的最大分数
int T[1 << MAXN];                            // T[i]表示执行到状态 i 一共用时多少分钟
bool judge(int i, int j)                     // 判断由状态 j 提交第 i 题是否可行
{
    for (int k = 1; k <= n; ++k)             // 判断提交第 i 题的前置条件是否满足
        if ((p[i].pre >> (k - 1)) & 1)
            if (!((j >> (k - 1)) & 1))
                return false;
    for (int k = 1; k <= n; ++k)             // 判断状态 j 包含的题目，前置条件是否都不包含第 i 题
    {
        if (k == i)
            continue;
        if (j & (1 << (k - 1)))             // 第 k 题在状态 j 中
            if (p[k].pre & (1 << (i - 1)))  // 第 k 题的前置条件包括第 i 题
                return false;
    }
    return true;
}
int main(void)
{
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i)
    {
        scanf("%d%d%d", &p[i].a, &p[i].b, &si);
        for (int j = 1; j <= si; ++j)
        {
            scanf("%d", &pi);
            p[i].pre |= 1 << (pi - 1);
        }
    }
    for (int i = 1; i < 1 << n; ++i)
        for (int j = 1; j <= n; ++j)
            if (i & (1 << (j - 1)))
                ++T[i];
    for (int i = 1; i < 1 << n; ++i)    // 遍历所有可能的题目状态
    {
        for (int j = 1; j <= n; ++j)    // 对于状态 i ，最后一个提交的题目是第 j 个题时
        {
            int ps = 1 << (j - 1);      // ps 为最后一个提交题目位置, i - ps 则为前置状态
            if ((i & ps) && judge(j, i - ps)) // 状态 i 包括了 j 题 && 最后一个提交 j 时，前置状态满足 j 题的提交条件
                mp[i] = max(mp[i], mp[i - ps] + (T[i - ps] + 1) * p[j].a + p[j].b);
        }
        ans = max(ans, mp[i]);
    }
    printf("%d\n", ans);
    return 0;
}
```