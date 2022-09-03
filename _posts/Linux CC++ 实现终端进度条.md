---
title: Linux C/C++ 实现终端进度条
date: 2022-09-04 02:19:00
updated: 2022-09-04 02:19:00
categories: [技术杂谈]
tags: [Linux, C, Cpp]
toc: true
---

在终端打印进度条原理很简单，有以下几点要清楚：

1. `\r` 的作用是将光标移回行首。

2. `\n` 的作用是换行（通常会 flush 缓冲的行）。

3. 我们可以用 `\r` 将光标移回行首，重新打印一样格式的内容，覆盖旧的内容，由于机器运行速度非常快，看起来就像原地修改一样。

下面是我自己实现的一个简单的进度条类，不过只是实现功能，不保证 bug free。

```cpp
#include <cstdio>
#include <ios>
#include <iostream>
#include <string>
#include <sys/types.h>
#include <unistd.h>

class ProgressBar {
public:
    ProgressBar(const char finish = '#', const char unfini = '.')
        : _flags("-\\|/"),
          _finish(finish),
          _progress_str(100, unfini),
          _cur_progress(0) {}

    void print_bar(const ushort n) {
        if (_cur_progress != 0 && n <= _cur_progress) {
            std::cerr << "\e[31merror\e[m: n(" << n 
                     << ") should > _cur_progress(" 
                     << _cur_progress << ")" << std::endl; 
            return ;
        }
        for (ushort i = _cur_progress; i < n; i++) {
            _progress_str[i] = _finish;
        }
        _cur_progress = n;
        std::string f, p;
        if (n == 100) {
            f = "\e[1;32mOK\e[m";
            p = "\e[1;32m100%\e[m";
        } else {
            f = _flags[n % 4];
            p = std::to_string(n) + '%';
        }
        std::cout << std::unitbuf
                  << '[' << f << ']'
                  << '[' << _progress_str << ']'
                  << '[' << p << "]" << '\r';
        if (n >= 100) {
            std::cout << std::endl;
        }
    }
private:
    std::string _flags;
    std::string _progress_str;
    ushort _cur_progress;
    char _finish;
};

// test
int main(int argc, char* argv[]) {
    ProgressBar pb;
    for (int i = 0; i <=100; i++) {
        pb.print_bar(i);
        usleep(50000);
    }
    return 0;
}
```

输出：
![](https://gukaifeng.cn/posts/linux-cc-shi-xian-zhong-duan-jin-du-tiao/linux-cc-shi-xian-zhong-duan-jin-du-tiao_1.gif)
