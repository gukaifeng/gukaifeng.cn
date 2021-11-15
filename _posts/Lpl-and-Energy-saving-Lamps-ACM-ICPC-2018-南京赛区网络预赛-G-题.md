---
title: ACM-ICPC 2018 南京赛区网络预赛 G 题 Lpl and Energy-saving Lamps（线段树）
mathjax: true
date: 2018-09-02 09:17:13
updated: 2018-09-02 09:17:13
tags: [ACM,线段树,算法]
categories: [技术杂谈]
toc: true
---


**传送门：**[ACM-ICPC 2018 南京赛区网络预赛 G 题](https://nanti.jisuanke.com/t/30996)

**题目大意：**
有 n 个房间，每个房间里有一定数量的白炽灯泡（数量不同），Lpl 每个月会购买 m 个节能灯泡去替换每个房间原有的白炽灯泡。房间有编号，每次换灯泡从第一个房间开始，如果 Lpl 手里的节能灯泡数量比房间的白炽灯泡数量多，那就把这个房间的灯泡全换了，并且以后不再看这个房间，如果不够，那就整个房间的都不换，往后找其他房间。如果手里的节能灯泡都换完了，就停止换灯泡（下个月还是从第一个房间开始换，不是接着停下的位置换），如果查完所有房间，手里还有剩余节能灯泡没有换，那就把剩下的节能灯泡留着，等下个月新买了灯泡后再换。当所有房间都换完以后，Lpl 就不再买节能灯泡了。问到第 d 个月的时候，有多少个房间已经换完了灯泡，此时 Lpl 手里还剩多少个节能灯泡。

<!--more-->

**解题思路：**
考虑直接模拟 Lpl 的换灯泡过程，问题在于查找白炽灯泡数小于等于 Lpl 手里节能灯泡数的房间，暴力查找复杂度过高，我们采用线段树，维护每一小段连续房间的最小灯泡数，这样每次查找只需要 log n 的复杂度，当某个房间换完灯泡以后，把这个房间的灯泡数修改成成最大值（在题目给出的数据范围内，Lpl 永远都攒不够的灯泡数目就可以）。如果在某一时刻换完的房间数等于 n，也就是所有房间都换完了，那么后面的每个月就不再继续操作了，所以从换完的那个月开始，已经换完的房间数和 Lpl 手里剩余的节能灯泡数不会再发生变化。关于询问，因为每次询问没有涉及修改，又比较跳跃，所以我们先将所有可能的询问全部计算出来保存在一个数组里，这样对于每一个询问，直接输出数组里的解即可。

**通过代码：**
```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 100005;
int n, m, q, d;
int num;            // 当前剩余节能灯数目
int ans[MAXN][2];   // ans[i][0] 为到第i个月时已经完成的房间数，ans[i][1] 为到第i个月时剩余的灯泡数
struct node
{
    int L, R;
    int minV;
    int Mid(void)
    {
        return (L + R) >> 1;
    }
}Tree[MAXN << 2];
void Build(int rt, int l, int r)
{
    Tree[rt].L = l;
    Tree[rt].R = r;
    if (l != r)
    {
        Build(rt << 1, l, Tree[rt].Mid());
        Build(rt << 1 | 1, Tree[rt].Mid() + 1, r);
        Tree[rt].minV = min(Tree[rt << 1].minV, Tree[rt << 1 | 1].minV);
    }
    else
        scanf("%d", &Tree[rt].minV);
}
void Modify(int rt, int i)
{
    if (Tree[rt].L == Tree[rt].R)
    {
        Tree[rt].minV = 999999999;	    // Lpl 永远都攒不够的灯泡数目
        return ;
    }
    if (i <= Tree[rt].Mid())
        Modify(rt << 1, i);
    else
        Modify(rt << 1 | 1, i);
    Tree[rt].minV = min(Tree[rt << 1].minV, Tree[rt << 1 | 1].minV);
}
int Query(int rt, int num0, int& res)    // 返回房间灯泡数比剩余灯泡数少的第一个房间，如果没有返回-1, 参数3为这个房间当前的灯泡数
{
    if (Tree[rt].minV > num0)
        return -1;
    if (Tree[rt].L == Tree[rt].R)
    {
        res = Tree[rt].minV;
        return Tree[rt].L;
    }
    if (Tree[rt << 1].minV <= num0)
        return Query(rt << 1, num0, res);
    else
        return Query(rt << 1 | 1, num0, res);
}
int main(void)
{
    scanf("%d%d", &n, &m);
    Build(1, 1, n);
    for (int i = 1; i <= MAXN; ++i)
    {
        ans[i][0] = ans[i - 1][0];
        ans[i][1] = ans[i - 1][1];
        if (ans[i][0] >= n)
            continue;
        else
            num = ans[i][1] + m;
        int r, res;
        while (1)
        {
            r = Query(1, num, res);
            if (r == -1)
                break;
            ++ans[i][0];
            num -= res;
            Modify(1, r);
        }
        ans[i][1] = num;
    }
    scanf("%d", &q);
    for (int i = 1; i <= q; ++i)
    {
        scanf("%d", &d);
        printf("%d %d\n", ans[d][0], ans[d][1]);
    }
    return 0;
}
```