




在 C/C++ 中，strftime() 和 strptime() 两个函数用于时间与字符串之间的转化，  
它们都在头文件 `time.h` 或 `ctime` 中。

<!--more-->

#### 1. strftime() -- 将 tm 时间结构体转为指定格式的时间字符串

函数声明如下

```cpp
size_t strftime (char *__restrict __s, size_t __maxsize,
			const char *__restrict __format,
			const struct tm *__restrict __tp);
```

* `__s`: 存放转换后指定格式时间字符串的起始地址；
* `__maxsize`: 可以在 `__s` 写入字符串的最大长度（含末尾的 `\0`）；
* `__format`: 待写入 `__s`  的格式化字符串；
* `__tp`: 时间 tm 结构体的指针；
* 返回值(`size_t`): 在 `__s` 写入的字符串长度（不含 `\0`）。

下面用代码说明一下

```cpp
#include <iostream>
#include <ctime>

int main(int argc, char *argv[])
{
     time_t rawtime;
     struct tm *timeinfo;

     time(&rawtime);
     timeinfo = localtime(&rawtime);

     char buffer[80];
     strftime(buffer, 80, "the current time is %Y-%m-%d %H:%M:%S", timeinfo); // 这里的说明符含义在下文列出

     puts(buffer);

     return 0;
}
```

输出

```
the current time is 2021-09-06 23:50:13
```







#### 2. strptime() -- 将指定格式的时间字符串转为 tm 时间结构体

函数声明如下

```cpp
char *strptime (const char *__restrict __s,
		       const char *__restrict __fmt, struct tm *__tp);
```

* `__s`: 待转换为 tm 结构体的时间字符串；
* `__fmt`: 待读取的格式化字符串；
* `__tp`: 时间 tm 结构体的指针，从字符串读取出来的时间信息会存到这里。
* 返回值(`char *`): 指向 `__s` 中第一个未匹配成功的字符，如果都匹配成功了的话，返回值是 `NULL`。



下面用代码说明一下，我们把上面的输出结果再转回去

```cpp
#include <iostream>
#include <ctime>

int main(int argc, char *argv[])
{
     struct tm timeinfo;

     char cur_time[] = "the current time is 2021-09-06 23:50:13";
     strptime(cur_time, "the current time is %Y-%m-%d %H:%M:%S", &timeinfo);  // 这就把时间信息存到 timeinfo 里了

     printf("%s\n", asctime(&timeinfo));
     
     return 0;
}
```

输出

```
Mon Sep  6 23:50:13 2021
```

注意我这里没输出任何其他内容，这是 asctime() 函数的默认输出格式，可以看到，这和 `2021-09-06 23:50:13` 是等价的（多了个 Mon 星期一）。



#### 3. 综合示例：将一种时间格式的字符串转换为另一种时间格式的字符串

有一个很常见的场景是，我们要在 C/C++ 代码里得到一个时间字符串，但是有两种麻烦的情况：

1. asctime() 和 ctime() 这种函数输出的时间格式不是我们想要的；
2. 我们拿到的已经是一个现成的字符串，但时间格式不是我们想要的。

第一种情况，在上面 strptime() 的示例中已经解决了，这里主要说第二种情况。

**场景：**已有一个某种格式的时间字符串；

**目标：**将其转换为另一种格式的时间字符串。

这里假定我们已有一个比较混乱的时间字符串（举个例子弄乱点嘛） `2021年 Sep 6 Mon 23:50:13`，要把它转换为 `2021-09-06 23:50:13 `。

思路比较简单， 先用 strptime() 把比较混乱的时间字符串读到 tm 结构体里，再用 strftime() 得到我们想要的格式。

下面看代码

```cpp
#include <iostream>
#include <ctime>

int main(int argc, char *argv[])
{
     struct tm timeinfo;

     char cur_time[] = "2021年 Sep 6 Mon 23:50:13";
     strptime(cur_time, "%Y年 %b %d %a %H:%M:%S %Y-%m-%d %H:%M:%S", &timeinfo);  // 这就把时间信息存到 timeinfo 里了
     
     size_t buf_size = 80;
     char buf[buf_size];
     strftime(buf, buf_size, "%Y-%m-%d %H:%M:%S", &timeinfo);

     printf("%s\n", buf);
     
     return 0;
}
```

输出

```
2021-09-06 23:50:13
```

完工！



#### 4. 说明符表

| 说明符 | 指代                                                         | 示例                       |
| ------ | ------------------------------------------------------------ | -------------------------- |
| `%a`   | Abbreviated weekday name *                                   | `Thu`                      |
| `%A`   | Full weekday name *                                          | `Thursday`                 |
| `%b`   | Abbreviated month name *                                     | `Aug`                      |
| `%B`   | Full month name *                                            | `August`                   |
| `%c`   | <font color=orange>Date and time representation *</font>     | `Thu Aug 23 14:55:02 2001` |
| `%C`   | Year divided by 100 and truncated to integer (`00-99`)       | `20`                       |
| `%d`   | Day of the month, zero-padded (`01-31`)                      | `23`                       |
| `%D`   | <font color=orange>Short `MM/DD/YY` date, equivalent to `%m/%d/%y` </font> | `08/23/01`                 |
| `%e`   | <font color=orange>Day of the month, space-padded (` 1-31`)</font> | `23`                       |
| `%F`   | <font color=orange>Short `YYYY-MM-DD` date, equivalent to `%Y-%m-%d`</font> | `2001-08-23`               |
| `%g`   | <font color=orange>Week-based year, last two digits (`00-99`)</font> | `01`                       |
| `%G`   | <font color=orange>Week-based year                           | `2001`                     |
| `%h`   | <font color=orange>Abbreviated month name * (same as `%b`)</font> | `Aug`                      |
| `%H`   | Hour in 24h format (`00-23`)                                 | `14`                       |
| `%I`   | Hour in 12h format (`01-12`)                                 | `02`                       |
| `%j`   | Day of the year (`001-366`)                                  | `235`                      |
| `%m`   | Month as a decimal number (`01-12`)                          | `08`                       |
| `%M`   | Minute (`00-59`)                                             | `55`                       |
| `%n`   | <font color=orange>New-line character (`'\n'`)</font>        | ``                         |
| `%p`   | AM or PM designation                                         | `PM`                       |
| `%r`   | <font color=orange>12-hour clock time *</font>               | `02:55:02 pm`              |
| `%R`   | <font color=orange>24-hour `HH:MM` time, equivalent to `%H:%M`</font> | `14:55`                    |
| `%S`   | Second (`00-61`)                                             | `02`                       |
| `%t`   | <font color=orange>Horizontal-tab character (`'\t'`)</font>  | ``                         |
| `%T`   | <font color=orange>ISO 8601 time format (`HH:MM:SS`), equivalent to `%H:%M:%S`</font> | `14:55:02`                 |
| `%u`   | <font color=orange>ISO 8601 weekday as number with Monday as `1` (`1-7`)</font> | `4`                        |
| `%U`   | Week number with the first Sunday as the first day of week one (`00-53`) | `33`                       |
| `%V`   | <font color=orange>ISO 8601 week number (`01-53`)</font>     | `34`                       |
| `%w`   | Weekday as a decimal number with Sunday as `0` (`0-6`)       | `4`                        |
| `%W`   | Week number with the first Monday as the first day of week one (`00-53`) | `34`                       |
| `%x`   | Date representation *                                        | `08/23/01`                 |
| `%X`   | Time representation *                                        | `14:55:02`                 |
| `%y`   | Year, last two digits (`00-99`)                              | `01`                       |
| `%Y`   | Year                                                         | `2001`                     |
| `%z`   | <font color=orange>ISO 8601 offset from UTC in timezone (1 minute=1, 1 hour=100) If timezone cannot be determined, no characters</font> | `+100`                     |
| `%Z`   | Timezone name or abbreviation * If timezone cannot be determined, no characters | `CDT`                      |
| `%%`   | A `%` sign                                                   | `%`                        |



\* 用星号(*)标记的说明符依赖于语言环境。

**注意：** 橙色的行表示 C99 引入的说明符和子说明符。从 C99 开始，可以在百分比符号(%)和说明符之间插入两个特定于区域设置的修饰符来请求另一种格式，如果适用的话：

| 修饰符 | 含义                                                         | 应用于                                                |
| ------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| `E`    | <font color=orange>Uses the locale's alternative representation</font> | `%Ec %EC %Ex %EX %Ey %EY`                             |
| `O`    | <font color=orange>Uses the locale's alternative numeric symbols</font> | `%Od %Oe %OH %OI %Om %OM %OS %Ou %OU %OV %Ow %OW %Oy` |
