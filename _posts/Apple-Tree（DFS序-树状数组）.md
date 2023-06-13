

**传送门：** [POJ 3321 - Apple Tree](http://poj.org/problem?id=3321)

**题目大意：**
给你一棵树（不是二叉树，多少个枝干都有可能），树的形式以结点关系描述的形式给出，比如给你 $u$ 和 $v$，表示编号为 $v$ 的结点是编号为 $u$ 的结点的子结点。每个结点上有一个苹果，有两个操作 C 和 Q ：
操作 C ：告诉你一个结点，如果这个结点当前有苹果，那就把它摘下来，如果没有，那这个结点就长出一个苹果。
操作 Q ：同样是告诉你一个结点，问你以这个结点为根的子树上一共有多少个苹果。

**解题思路：**

遇到这类有频繁的单点修改和区间查询操作的题目，一般首先想到的都是树状数组或者线段树来解，这个题目也没例外。

对于这个题，给出的是结点的关系，那么我们就可以用 DFS序 得到这个树的遍历序列，我们知道，DFS序有一个特殊的性质，就是一个结点的子树一定是在一个连续的区间内的，比如题目样例里面的数据，DFS序为 "122331"，对于以结点 1 为根的子树，每一个结点的进入与离开时间，全都在结点 1 的进入时间和离开时间之间。

我们用树状数组来维护DFS序列，对于操作 C ，我们只要把DFS序列中这个结点的进入和离开的序列位置都进行 -1 或者 +1 就好了；对于操作 Q ，就是用数组求被询问结点的进入时间到离开时间序列和，然后因为序列中既有进入的又有离开的，每个结点加了两次，所以结果要再除以 2。

这个题有点小坑，Runtime Error 了几发， 又 Time Limit Exceeded 了几发。前者不怪我，是题目坑，题里给的数据范围不对着呢...我试了好几次，最后开到了 200000 才好（题目里明明写的最大才 100000， =。=）。超时的原因貌似是卡了 vector，改用手写的动态链表就过了。<!--more-->

**通过代码：**
```cpp
#include <iostream>
#include <cstdio>
#define MAX 200000
#define lowbit(x) x & -x
using namespace std;
class Node
{
public:
    Node(int Val)
    {
        v = Val;
        next = NULL;
    }
    int v;
    Node *next;
};
class Branch
{
public:
    Branch(void)
    {
        L = 0;
        first = rear = NULL;
    }
    int L;
    Node *first, *rear;
}Branches[MAX];
int N, M;
int In[MAX], Out[MAX]; 
char op;
int num, u, v;
int Tree[MAX << 1], len;
bool HasApple[MAX];
void dfs(int i)
{
    In[i] = ++len;
    Node *p = Branches[i].first;
    while (p)
    {
        dfs(p->v);
        p = p->next;
    }
    Out[i] = ++len; 
}
void Modify(int i, int v)
{
    while (i <= len) 
    {
        Tree[i] += v;
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
int Query(int i, int j)
{
    return Sum(j) - Sum(i - 1);
}
int main(void)
{
    scanf("%d", &N);
    for (int i = 1; i <= N - 1; ++i)
    {
        scanf("%d%d", &u, &v);
        if (Branches[u].first == NULL)
            Branches[u].first = Branches[u].rear = new Node(v);
        else
        {
            Branches[u].rear->next = new Node(v);
            Branches[u].rear = Branches[u].rear->next;
        }
    }
    dfs(1);
    for (int i = 1; i <= len; ++i)
    {
        Modify(i, 1);
        HasApple[i] = true;
    }
    scanf("%d", &M);
    for (int i = 1; i <= M; ++i)
    {
        scanf(" %c%d", &op, &num);
        if (op == 'C')
        {
            int value = HasApple[num] ? -1 : 1;
            HasApple[num] = HasApple[num] ? false : true;
            Modify(In[num], value);
            Modify(Out[num], value);
        }
        else
            printf("%d\n", Query(In[num], Out[num]) / 2);
    }
    return 0;
}
```