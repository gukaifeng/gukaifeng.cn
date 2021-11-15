---
title: Death to Binary?（斐波那契+模拟）
mathjax: true
date: 2018-10-03 14:45:14
updated: 2018-10-03 14:45:14
tags: [POJ,斐波那契]
categories: [算法题解]
toc: true
---


**传送门：**[POJ 2116 - Death to Binary?](http://poj.org/problem?id=2116)

**题目大意：**

有两个数，这两个数以斐波那契基数的形式给出。对应的斐波那契序列以 $Fib[0]=1, Fib[1]=2$ 开始。比如题目描述中的例子，数字 $40$，用斐波那契基数表示，可以是 $1101001$，也可以是 $10001001$，这里以第一个为例来解释，类似二进制，只不过右数第 $k$ 位（$k$从 $0$ 开始）对应的不再是 $2^k$，而是 $Fib[k]$，因为 $40=Fib[6]+Fib[5]+Fib[3]+Fib[0]$，和二进制一样，某一位上有值的为 $1$，没有的为 $0$，所以 $40=1101001_{Fib}$。然后告诉你，每个数字用斐波那契基数表示时，有唯一的一种表示是不包含两个连续 $1$ 的，让你拿这种情况去计算。给你两个斐波那契基数表示的数（不一定是没有连续 $1$ 的形式），让你计算加法，最后也以斐波那契基数的表示形式输出结果。然后对输出的格式有一定的要求，并且输出的数还得是那种没有连续 $1$ 形式（我有一句MMP不知当讲不当讲）。

**解题思路：**

讲道理这个题很简单，但是做起来很坑很坑。WA了好多发。这里说一下那些坑，大家就肯定会做了。
坑1：输入可能包含前导 $0$。
坑2：输入可能是 $0\ 0$ 这种。
坑3：如果从高位往后依次进位 $11$ 为 $100$ 的话，可能导致高位重新出现 $11$， 比如 $1011$，进位后是 $1100$，这里我的做法是每轮扫描都从头开始，反正才 $40$ 的长度。
坑4：格式化输出废了我好多行代码，差评！
另外注意下初始化，然后如果用数组的稍微开大一点，两个 $40$ 位的相加最长会有 $43$ 位，不过我是用字符串做的，所以无所谓了。<!--more-->

**通过代码：**

```cpp
#include <iostream>
#include <string>
using namespace std;
string x, y;
int fib[45] = {1, 2};
void delete0(string &s)
{
    int i = 0;
    while (i < s.size() - 1 && s[i] == '0')
        ++i;
    s = s.substr(i);
}
void rep(string &s)
{
    int pos = s.find("11", 0);
    while (pos != string::npos)
    {
        if (pos - 1 >= 0 && s[pos - 1] == '0')
            s.replace(pos - 1, 3, "100");
        else
            s.replace(pos, 2, "100");
        pos = s.find("11", 0);
    }
}
string add(string x, string y)
{
    string res = "0";
    int b1 = 0, b2 = 0, sum;
    int len = x.size();
    for (int i = len - 1; i >= 0; --i)
    {
        b1 += x[i] == '1' ? fib[len - 1 - i] : 0;
        b2 += y[i] == '1' ? fib[len - 1 - i] : 0;
    }
    sum = b1 + b2;
    for (int i = 44; i >= 0; --i)
    {
        if (sum >= fib[i])
        {
            sum -= fib[i];
            res += '1';
        }
        else
            res += '0';
    }
    delete0(res);
    rep(res);
    return res;
}
void toequal(string &x, string &y)
{
    while (x.size() != y.size())
        if (x.size() > y.size())
            y = ' ' + y;
        else
            x = ' ' + x;
}
int main(void)
{
    for (int i = 2; i < 45; ++i)
        fib[i] = fib[i - 1] + fib[i - 2];
    while (cin >> x >> y)
    {
        delete0(x);
        delete0(y);
        rep(x);
        rep(y);
        toequal(x, y);
        string res = add(x, y);
        toequal(x, res);
        toequal(y, res);
        string line;
        while (line.size() < res.size())
            line += '-';
        x = "  " + x;
        y = "+ " + y;
        line = "  " + line;
        res = "  " + res;
        cout << x << endl
             << y << endl
             << line << endl
             << res << endl
             << endl;
    }
    return 0
}
```
