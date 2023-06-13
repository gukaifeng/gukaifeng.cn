
**传送门：**[HDU 2161 - Primes](http://acm.hdu.edu.cn/showproblem.php?pid=2161)

**题目大意：**

给你一个数，让你判断它是不是素数。

**解题思路：**

超级大水题，题目简单数据范围也小，随便一个[素数筛子](https://gukaifeng.me/2018/09/06/%E7%B4%A0%E6%95%B0%E6%B5%8B%E8%AF%95/)就过了，注意下这个题认为 $1$ 和 $2$ 不是素数就行了。<!--more-->

**通过代码：**
```cpp
#include <iostream>
#include <cstring>
using namespace std;
const int N = 17000;
bool isprime[N];
int prime[N], nprime; // 下标从 1 开始
void doprime(void)
{
    nprime = 0;
    memset(isprime, true, sizeof(isprime));
    isprime[1] = false;
    for (long long i = 2; i < N; ++i)
        if (isprime[i])
        {
            prime[++nprime] = i;
            for (long long j = i * i; j < N; j += i)
                isprime[j] = false;
        }
    isprime[2] = false;
}
int main(void)
{
    doprime();
    int num, i = 0;
    while (cin >> num && num > 0 && ++i)
        cout << i << ": " << (isprime[num] ? "yes" : "no") << endl;
    return 0;
}
```
