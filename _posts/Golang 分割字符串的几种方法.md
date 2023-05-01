---
title: "Golang 分割字符串的几种方法"
date: 2023-05-01 22:52:00
updated: 2023-05-01 22:52:00
categories:
- 编程语言基础
- Golang
tags: [Golang]
---





## 1. 按单个分隔字符串分割



按单个分隔符或分隔字符串分割可使用 `Split()` 函数，声明如下：

```go
func Split(s, sep string) []string
```

```go
func Split(s, sep []byte) [][]byte
```



两个方法是一样的，只是操作的字符串类型一个是 `string` 一个是 `[]byte`。

我们以操作 `string` 的方法为例：

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Printf("%q\n", strings.Split("a,b,,,c", ","))
	fmt.Printf("%q\n", strings.Split("a man a plan a canal panama", "a "))
	fmt.Printf("%q\n", strings.Split(" xyz ", ""))
	fmt.Printf("%q\n", strings.Split("", "Bernardo O'Higgins"))
}

```

输出：

```
["a" "b" "" "" "c"]
["" "man " "plan " "canal panama"]
[" " "x" "y" "z" " "]
[""]
```



可以看到，分隔字符串 `sep` 被视为一个整体，而不是像一些其他语言那样将 `sep` 字符串中的每个字符算作一个单独的分隔符。



想要同时使用多个分隔符，就需要下面第 2 小节的方法。



\-



这里再简单介绍下另一个方法，不是很常用，但偶尔会用到：

```go
func SplitAfter(s, sep string) []string
```

这个方法会在分割后的子串里保留分隔符，看个例子：

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Printf("%q\n", strings.SplitAfter("a,b,c", ","))
}
```

输出：

```
["a," "b," "c"]
```



## 2. 按多个分隔符分割

要想使用多个分隔符分割字符串，需要用到 strings 包内的 `FieldsFunc()` 方法，其函数声明如下：

```go
func FieldsFunc(s string, f func(rune) bool) []string
```

这个方法也很简单，其除了接受待分割字符串 `s` 外，还接受一个参数为 `rune`，返回值为 `bool` 的函数 `f`。当将 `s` 中的字符作为参数传递给 `f` 函数时，返回值若为 true，则将这个字符视为分隔符。最终将分割好的字符串返回。



下面看一段代码示例：

```go
package main

import (
	"fmt"
	"strings"
	"unicode"
)

func main() {
	f0 := func(c rune) bool {
		return !unicode.IsLetter(c) && !unicode.IsNumber(c)
	}
	f1 := func(c rune) bool {
		return c == ' ' || c == ';' || c == ',' || c == '.'
	}
	fmt.Printf("Fields are: %q\n", strings.FieldsFunc("  foo1;bar2,baz3...", f0))
	fmt.Printf("Fields are: %q\n", strings.FieldsFunc("  foo1;bar2,baz3...", f1))
}

```

输出：

```go
Fields are: ["foo1" "bar2" "baz3"]
Fields are: ["foo1" "bar2" "baz3"]
```

要注意到这个方法有一点和 `Split()` 不同，**`FieldsFunc()` 并不会将空串视为一个分割后的字符串。**



## 3. 使用正则分割



Golang 的 regexp 包中的 Regexp 对象也提供一个使用正则分割字符串的方法，即用正则匹配分隔符或分隔字符串。方法声明如下：



```go
func (re *Regexp) Split(s string, n int) []string
```

此方法接受两个参数：

* `s` 是待分割的字符串。
* `n` 是最大的分割数量：
  * `n > 0` 则指定最大分割段数（分割从左自由，达到最大分割数量后，剩余的字符串全在分割后的最后一个子串里）；
  * `n = 0` 则不分割，直接返回 `nil`；
  * `n < 0` 表示全部分割。



返回值为分割后的字符串数组。



下面看一段代码示例：



```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	a := regexp.MustCompile(`a`)
	fmt.Println(a.Split("banana", -1))
	fmt.Println(a.Split("banana", 0))
	fmt.Println(a.Split("banana", 1))
	fmt.Println(a.Split("banana", 2))

	zp := regexp.MustCompile(`z+`)
	fmt.Println(zp.Split("pizza", -1))
	fmt.Println(zp.Split("pizza", 0))
	fmt.Println(zp.Split("pizza", 1))
	fmt.Println(zp.Split("pizza", 2))

	s := regexp.MustCompile("a*").Split("abaabaccadaaae", 5) // s: ["", "b", "b", "c", "cadaaae"]
	fmt.Println(s)
}
```

输出：

```
[b n n ]
[]
[banana]
[b nana]
[pi a]
[]
[pizza]
[pi a]
[ b b c cadaaae]
```





