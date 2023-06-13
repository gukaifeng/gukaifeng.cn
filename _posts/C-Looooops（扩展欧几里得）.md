

**传送门：**[POJ 2115 - C Looooops](http://poj.org/problem?id=2115)

**题目大意：**

给你一个 $C$语言中的 $for$ 循环伪代码，如下：

```cpp
for (variable = A; variable != B; variable += C)
    statements;
```

varibale是变量名，所有数据的类型都是一个 $k$ 位的无符号整数，给你 $A,B,C,k$ 的值，问你要这个循环会执行多少次后退出。

**解题思路：**

我们知道在 $C$语言中，如果一个数超过了它的上界，那么会从下界重新开始，同样如果超过了下界也是一样会从上界重新开始。
比如说，int类型的范围是 $[-2147483648,2147483647]$。有如下代码：

```cpp
int int_max = 2147483647;
cout << int_max + 1 << endl;
```
输出如下：

```cpp
-2147483648
```

可以看出数值从上界超出后回到了下界，这样这道题就很好去算了，设循环会执行 $x$ 次，我们可以根据题意列出一个等式：
$(A+Cx)\%2^k=B$
变形一下成为我们需要的形式：
$Cx+2^ky=B-A$
这里 $y$ 的值不重要，就是表示一下 $2^k$ 的倍数使等式成立。
我们需要的是 $x$，因为循环次数肯定是非负的，所以我们要去计算这个等式的 $x$ 解的最小的非负解。
分析到这里，这个题就很明显了，是一个[扩展欧几里得](https://gukaifeng.me/2018/08/31/%E6%89%A9%E5%B1%95%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%B7/)求解二元一次方程的板子题，然后在把 $x$ 算一下最小非负的值就搞定了。
然后就是解方程的时候如果发现没有整数解，那就是 FOREVER 的情况了。<!--more-->

**通过代码：**

```cpp
#include <iostream>
using namespace std;
typedef long long ll;
ll exgcd(ll a, ll b, ll &x, ll &y)
{
    if (b == 0)
    {
        x = 1, y = 0;
        return a;
    }
    ll q = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return q;
}
bool eqa_solve(ll &x, ll &y, ll a, ll b, ll c)
{
    ll q = exgcd(a, b, x, y);
    if (c % q != 0)
        return false;
    ll tms = c / q;
    x *= tms;
    y *= tms;
    ll m = b / q;
    x = (x % m + m) % m; // 这句就是把 x 变为最小的非负整数解
    return true;
}
int main(void)
{
    ll a, b, c, k;
    while (cin >> a >> b >> c >> k && k)
    {
        ll x, y, m = (ll)1 << k;
        if (eqa_solve(x, y, c, m, b - a))
            cout << x << endl;
        else
            cout << "FOREVER" << endl;
    }
    return 0;
}
```
