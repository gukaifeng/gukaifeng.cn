---
title: "Golang 中正则表达式 regexp 的基本用法"
date: 2023-05-01 00:44:00
updated: 2023-05-01 21:18:00
categories:
- 编程语言基础
- Golang
tags: [Golang]
---



Golang 中使用正则一般就是使用 regexp 包。



这里只介绍一些常用操作，已经可以满足大部分开发需求，更多内容请详见官方文档 [regexp package](https://pkg.go.dev/regexp)。







## 1. 正则表达式的语法



我们知道想要使用正则匹配，就需要严格按照正则语法设定正则匹配字符串，即 Pattern。



正则语法篇幅较长，这里就不介绍了，并且 Golang 官方的正则语法文档已经非常全面了，可以自己看看 [regexp/syntax](https://pkg.go.dev/regexp/syntax)。





## 2. regexp 的用法



regexp 包的主要使用方式有两种：

1. 直接使用 regexp 包内的全局函数进行模式匹配。
2. 先将 Pattern 封装进 `Regexp` 类中，再通过 `Regexp` 类提供的方法来进行正则匹配相关操作。



第 1 种方式能做的事很少，一共仅有 4 个方法，但却是正则操作里很常用方法。  
第 2 种方式相对复杂，但提供了非常多的方法，几乎能够满足正则匹配各种场景下的所有需求。





两种方式没有优劣之分，一般来说，简单需求用方式 1，方式 1 无法满足的时候用方式 2，就可以了。



下面第 3 小节对应方式 1，第 4小节对应方式 2。



## 3. 使用 regexp 包内的全局方法

regexp 包内的全局方法一共有四个。



### 3.1. 正则匹配 `[]byte`



函数声明：

```go
func Match(pattern string, b []byte) (matched bool, err error)
```

用法示例：

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	matched, err := regexp.Match(`foo.*`, []byte(`seafood`))
	fmt.Println(matched, err)
	matched, err = regexp.Match(`bar.*`, []byte(`seafood`))
	fmt.Println(matched, err)
	matched, err = regexp.Match(`a(b`, []byte(`seafood`))
	fmt.Println(matched, err)

}
```

输出：

```
true <nil>
false <nil>
false error parsing regexp: missing closing ): `a(b`
```





### 3.2. 正则匹配 `io.RuneReader`



函数声明：

```go
func MatchReader(pattern string, r io.RuneReader) (matched bool, err error)
```

这个方法似乎不是很常用，也很简单，这里就不举例了。



### 3.3. 正则匹配 `string`



函数声明：

```go
func MatchString(pattern string, s string) (matched bool, err error)
```

用法示例：

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	matched, err := regexp.MatchString(`foo.*`, "seafood")
	fmt.Println(matched, err)
	matched, err = regexp.MatchString(`bar.*`, "seafood")
	fmt.Println(matched, err)
	matched, err = regexp.MatchString(`a(b`, "seafood")
	fmt.Println(matched, err)
}
```

输出：

```
true <nil>
false <nil>
false error parsing regexp: missing closing ): `a(b`
```







### 3.4. 转义 Pattern



有些时候，我们的 Pattern 字符串里，本身就含有一些正则符号，如果不处理直接使用，就会被当做正则语法进行匹配，这不是我们期望的。



regexp 提供了一个方法，可以把一个字符串里所有与正则语法一致的字符进行转义，这样作为 Pattern 的时候就不会错误匹配了，



函数声明：

```go
func QuoteMeta(s string) string
```

用法示例：

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	fmt.Println(regexp.QuoteMeta(`Escaping symbols like: .+*?()|[]{}^$`))
}
```

输出：

```
Escaping symbols like: \.\+\*\?\(\)\|\[\]\{\}\^\$
```



## 4. 通过 Regexp 类进行正则相关操作



### 4.1. 编译解析正则表达式（必须）



我们不会手动创建一个 Regexp 类型的对象，而是通过 regexp 包提供的方法编译解析正则表达式，并得到一个 Regexp 类的对象。



编译解析正则表达式的方法一共有 4 个，但最常用的是 `MustCompile()`：

```go
func MustCompile(str string) *Regexp
```

参数 `str` 即为我们要使用的正则字符串 pattern，此方法返回一个 Regexp 类型的对象。如果 pattern 不合法，进程将 panic。



这一步是必须的，我们必须通过此方法获得的 Regexp 类对象进行各类正则操作。



代码示例如下：

```go
package main

import "regexp"

func main() {
	re := regexp.MustCompile(`regular_expression`)
}
```

* `re` 即为 Regexp 类型的对象，我们使用 `re` 来进行正则相关操作。
* `regular_expression` 为正则表达式，注意这里在参数传入正则表达式的时候，要用反引号 `` ` `` ，而不是双引号 `"`，否则可能会被转义。



\-



我们调用上面的方法编译解析正则表达式以后，就一直使用 Regexp 对象操作了，不过也许也会在某处想知道用来初始化 Regexp 对象的原始正则字符串是哪个，就会用到下面这个方法：

```go
func (re *Regexp) String() string
```

这个就不写代码示例了，太简单了，就是直接返回用来初始化 Regexp 对象的原始正则字符串。



\-



另外三个编译解析正则表达式的方法是：

```go
func Compile(expr string) (*Regexp, error)
func CompilePOSIX(expr string) (*Regexp, error)
func MustCompilePOSIX(str string) *Regexp
```



这里简单说一下 4 个编译解析正则表达式的方法的区别：

* `Compile()` 与 `CompilePOSIX()`，当编译解析正则表达式失败时（例如 pattern 不合法），程序会继续运行，程序要需要自己处理返回的 `err`。
* `MustCompile()` 与 `MustCompilePOSIX()`，功能与上两个一样，但当编译解析正则表达式失败时，进程会直接 panic。
* 两个名称带有 POSIX 字样的方法，采用的是另一种 POSIX 的正则规则，而我们通常使用的是 Perl 的正则规则。两种正则规则略有差异，如我们常用的 Perl 正则规则的是最左最先匹配，而 POSIX 则是最左最长匹配，两种正则规则的表达式语法也略有差异。



因为我们实际开发中编译解析正则这一步是必须成功的，不然无法进行下一步，所以 `Compile()` 和 `CompilePOSIX()` 就不太常用了，除非正则的 pattern 是外部传入的（例如由用户输入）。



而正则表达式的语法与匹配规则方面，通用的是 Perl 的，而不是 POSIX 的，所以 `CompilePOSIX()` 和 `MustCompilePOSIX()` 就也不那么常用了。



所以最终就有我前面说的，除非有特殊需求，不然最常用的是 `MustCompile()`。本文后面的代码示例中，编译解析正则表达式的步骤，也都会使用 `MustCompile()` 而非其他 3 个。





\-



下面说 Regexp 类对象支持的一些常用正则操作，注意这里只有我认为比较常用的，一些比较复杂的用法详见官方文档  [regexp package](https://pkg.go.dev/regexp)。







### 4.2. 仅检查字符串是否包含正则表达式的任何匹配



regexp 有两个方法检查字符串是否包含正则表达式的任何匹配：

```go
func (re *Regexp) Match(b []byte) bool
```

```go
func (re *Regexp) MatchString(s string) bool
```

这两个方法作用是一样的，唯一的区别就是接收的参数一个是 `[]byte` 类型，一个是 `string` 类型。如果匹配成功了就返回 true，否则返回 false。



这里以 `MatchString()` 举例：

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	re := regexp.MustCompile(`(gopher){2}`)
	fmt.Println(re.MatchString("gopher"))
	fmt.Println(re.MatchString("gophergopher"))
	fmt.Println(re.MatchString("gophergophergopher"))
}
```

输出：

```go
false
true
true
```









### 4.3. 返回字符串中匹配的首个子串



`Match()` 和 `MatchString()` 的方法仅检查是否匹配，而这里则是要得到的具体匹配成功的字符串。我们先看两个函数声明：



```go
func (re *Regexp) Find(b []byte) []byte
```

```go
func (re *Regexp) FindString(s string) string
```



这两个方法返回字符串中与正则表达式匹配的首个子串，区别同样是接收的参数一个是 `[]byte` 类型，一个是 `string` 类型。如果匹配成功了就返回匹配的子串，否则 `Find()` 返回 `nil`，`FindString()` 返回一个空串。



返回的首个符合条件的子串，也就是最左边的。



具体的匹配长度偏好，规则与正则符号的偏好一致。例如 `?` 匹配前面字符的 0 个或 1 个，偏好 1 个；`*` 匹配前面字符的任意个，偏好是越多越好。





这里以 `FindString()` 举例：

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	re := regexp.MustCompile(`foo.?`)
	fmt.Printf("%q\n", re.FindString("seafood fool"))
	fmt.Printf("%q\n", re.FindString("meat"))
}
```

输出：

```
"food"
""
```

注意的匹配长度偏好，正则表达式 `foo.?` 中的 `?` 匹配前面字符的 0 个或 1 个，偏好 1 个。我们 `FindString()` 方法的匹配长度偏好与正则符号的偏好是一致的，所以这里匹配的结果时 "food"，`?` 取偏好 1 个。







### 4.4. 返回字符串中匹配的所有子串



`Find()` 和 `FindString()` 只返回首个匹配的字符串，如果我们想要全部的，就需要下面这两个方法：



```go
func (re *Regexp) FindAll(b []byte, n int) [][]byte
```

```go
func (re *Regexp) FindAllString(s string, n int) []string
```



同样是一个匹配 `[]byte` 一个匹配 `string`，如果有至少有一个匹配的子串，那么就返回子串数组，否则就返回 `nil`。



第二个参数 `n` 的含义是匹配子串的最大数量（从左开始数），`n` 为负数的话则表示匹配全部。



注意这里的匹配长度规则，多个匹配子串之间是不会重叠的，每个匹配串的长度偏好同样与正则符号的偏好一致。



这里以 `FindAllString()` 举例：



```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	re := regexp.MustCompile(`a.`)
	fmt.Println(re.FindAllString("paranormal", -1))
	fmt.Println(re.FindAllString("paranormal", 2))
	fmt.Println(re.FindAllString("graal", -1))
	fmt.Println(re.FindAllString("none", -1))
}
```

输出：

```
[ar an al]
[ar an]
[aa]
[]
```



### 4.5. 返回字符串中匹配子串的索引



前面说的 `Find()`、`FindString()`、`FindAll()` 和 `FindAllString()` 都是返回匹配的字符串，但是没有给出查到的字符串的位置。



我们我们要想知道盘匹配的字符串的起始索引，就需要下面的方法：



```go
func (re *Regexp) FindIndex(b []byte) (loc []int)
```

```go
func (re *Regexp) FindStringIndex(s string) (loc []int)
```

```go
func (re *Regexp) FindAllIndex(b []byte, n int) [][]int
```

```go
func (re *Regexp) FindAllStringIndex(s string, n int) [][]int
```

这些方法与前面说过的 `Find()`、`FindString()`、`FindAll()` 和 `FindAllString()`  一一对应，使用方法完全一样，仅返回值不同。



对于匹配的子串，`FindIndex()` 和 `FindStringIndex()` 返回的是一个长度为 2 的 int 数组，`loc[0]` 是子串的起始索引，`loc[1]` 是子串结束索引的下一位，**注意这里是左闭右开 `[loc[0], loc[1])` **，索引从 0 开始，。`FindAllIndex()` 和 `FindAllStringIndex()` 也差不多，返回的是一个二维的 int 数组，同样存的是每个子串的起始和结束索引。如果没有匹配到任何子串，这些方法都返回 `nil`。





这里分别以 `FindStringIndex()` 和 `FindAllStringIndex()` 举例：

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	re := regexp.MustCompile(`ab.?`)
	fmt.Println(re.FindString("tablett"))
	fmt.Println(re.FindStringIndex("tablett"))
	fmt.Println(re.FindAllString("abcabdab", -1))
	fmt.Println(re.FindAllStringIndex("abcabdab", -1))
}
```



输出：

```
abl
[1 4]
[abc abd ab]
[[0 3] [3 6] [6 8]]
```



### 4.6. 使用正则分割字符串



Regexp 对象也提供一个使用正则分割字符串的方法，即用正则匹配分隔符或分隔字符串。方法声明如下：



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







## 5. 扩展：Regexp 的并发安全性



关于 Regexp 对象，由于我们所有的操作都是只读的，不会修改 Regexp 对象，也不会修改待匹配的字符串（即便是替换操作，如 `ReplaceAll()`，也是返回一个替换后的副本），所以 Regexp 对象是并发安全的。



不过有一点例外，就是成员方法 `Longest()`。这个方法会修改 Regexp 对象本身，其修改了正则匹配的规则，这个方法不是并发安全的。



所以只要我们没有使用 `Longest()` 方法，Regexp 对象的相关操作就都是并发安全的，我们大可放心，无需操心并发和同步问题。
