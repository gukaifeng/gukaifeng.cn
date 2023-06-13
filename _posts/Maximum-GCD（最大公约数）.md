

**传送门：**[UVA 11827 - Maximum GCD](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=2927)

**题目大意：**
给你几组数，每组数占输入的一行，但具体几个数没给你，让你算每组数中两两数的最大公约数最大是多少。

**解题思路：**
本来是是打算弄个素数筛，如果两数中有一个数是素数，就不算他俩的最大公约数了，伪暴力枚举两两数的最大公约数比较下。然后突然发现数据范围超级小啊，就直接暴力过了。。。注意下这题还是有几个坑的，首先是得按行读入，再把每行里的数分离出来，这样的话，读第一个数（就是那个测试用例数目）的时候，末尾换行符要处理一下。这里求公约数用的是[欧几里得](https://gukaifeng.me/2018/08/31/%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%B7/)算法。<!--more-->

**通过代码：**
```cpp
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <sstream>
using namespace std;
int gcd(int a, int b)
{
    return b == 0 ? a : gcd(b, a % b);
}
int n, m;
int arr[150];
string line;
int main(void)
{
    scanf("%d ", &n);
    while (n--)
    {
        getline(cin, line);
        stringstream ss(line);
        int ans = 1;
        for (m = 0; ss >> arr[m + 1]; ++m)
            for (int j = 1; j <= m; ++j)
                ans = max(ans, gcd(arr[j], arr[m + 1]));
        cout << ans << endl;
    }
    return 0;
}
```