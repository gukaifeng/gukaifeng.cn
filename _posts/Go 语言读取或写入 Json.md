---
title: Go 语言读取或写入 Json
date: 2022-01-11 14:23:40
updated: 2022-01-11 14:23:40
categories:
- 编程语言基础
- Golang
tags: [Golang,json]
---

Go 语言提供了关于 json 的标准库包 `encoding/json`。

详细官网文档见 [json package - encoding/json - pkg.go.dev](https://pkg.go.dev/encoding/json)，这里只介绍几种常用操作。

<!--more-->





## 1. 编码

编码分为两步：

1. 创建一个新的 json 文件；
2. 将数据结构中的内容按格式写入 json 文件。

第二步的写入，首先要使用 `json.NewEncoder()` 创建一个编码器，其参数是我们新创建的 json 文件指针。然后编码要使用编码器中的 `Encode()` 方法，其参数是我们要编码进 json 的内容。

我们看代码：

```go
package main

import (
	"encoding/json"
	"log"
	"os"
)

type RepoArr struct {
	Repo1 string
	Repo2 string
	Repo3 string
}

type Config struct {
	Name   string
	Blog   string
	Repo   RepoArr
	StrArr []string
}

func main() {
	jsonPath := "./myinfo.json"

	info := []Config{{"gukaifeng", "https://gukaifeng.cn", RepoArr{"git@github.com:gukaifeng/hexo.git", "git@github.com:gukaifeng/hexo.git_2", "git@github.com:gukaifeng/hexo.git_3"}, []string{}},
		{"gukaifeng2", "https://gukaifeng.cn2", RepoArr{"git@github.com:gukaifeng/hexo.git2", "git@github.com:gukaifeng/hexo.git2_2", "git@github.com:gukaifeng/hexo.git2_3"}, []string{"string-test-1", "string-test-2"}}}

	jsonFile, err := os.Create(jsonPath) // 创建 json 文件
	if err != nil {
		log.Printf("create json file %v error [ %v ]", jsonPath, err)
		return
	}
	defer jsonFile.Close()

	encode := json.NewEncoder(jsonFile) // 创建编码器
	err = encode.Encode(info)           // 编码
	if err != nil {
		log.Printf("encode error [ %v ]", err)
		return
	}
}

```

我们执行代码后，会在 go 程序同级目录发现一个新的 `myinfo.json` 文件，其内容如下：

```json
[{"Name":"gukaifeng","Blog":"https://gukaifeng.cn","Repo":{"Repo1":"git@github.com:gukaifeng/hexo.git","Repo2":"git@github.com:gukaifeng/hexo.git_2","Repo3":"git@github.com:gukaifeng/hexo.git_3"},"StrArr":[]},{"Name":"gukaifeng2","Blog":"https://gukaifeng.cn2","Repo":{"Repo1":"git@github.com:gukaifeng/hexo.git2","Repo2":"git@github.com:gukaifeng/hexo.git2_2","Repo3":"git@github.com:gukaifeng/hexo.git2_3"},"StrArr":["string-test-1","string-test-2"]}]
```

我们的程序将 json 写成了一行，这不影响解析等操作，不过我们看着费劲。

格式化一下（你可以使用任意编辑器进行格式化操作），如下：

```json
[
    {
        "Name": "gukaifeng",
        "Blog": "https://gukaifeng.cn",
        "Repo": {
            "Repo1": "git@github.com:gukaifeng/hexo.git",
            "Repo2": "git@github.com:gukaifeng/hexo.git_2",
            "Repo3": "git@github.com:gukaifeng/hexo.git_3"
        },
        "StrArr": []
    },
    {
        "Name": "gukaifeng2",
        "Blog": "https://gukaifeng.cn2",
        "Repo": {
            "Repo1": "git@github.com:gukaifeng/hexo.git2",
            "Repo2": "git@github.com:gukaifeng/hexo.git2_2",
            "Repo3": "git@github.com:gukaifeng/hexo.git2_3"
        },
        "StrArr": [
            "string-test-1",
            "string-test-2"
        ]
    }
]
```



上面的示例中包含了 json 中的各种情况的项的示例，按需使用就好。

我们可以发现，go 写入的 json，key 与 value 是和我们的写入数据结构体中的成员名与其值相对应的！





## 2. 解码

同样的，解码也是分两步：

1. 打开待解码的 json 文件；
2. 使用 json 包提供的方法解码 json 文件到数据结构中。

第 2 步的解码首先要使用 `json.NewDecoder()` 创建一个解码器，其参数是我们打开的 json 文件指针。然后解码要使用解码器中的 `Decode()` 方法，其参数是将要存储解码信息的数据结构对象。

我们下面通过具体例子来解释。

### 2.1. 只有一组 json 项



假设我们有一个 `myinfo.json`，与我们的 go 程序在同级目录，其中内容如下：

```json
{
    "name": "gukaifeng",
    "blog": "https://gukaifeng.cn",
    "repo": "git@github.com:gukaifeng/hexo.git"
}
```

我们看 go 语言解码代码：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
)

type Config struct {
	Repo string 
	Blog string
	Name string
}

func main() {
	jsonPath := "./myinfo.json"

	jsonFile, err := os.Open(jsonPath)
	if err != nil {
		log.Printf("open json file %v error [ %v ]", jsonPath, err)
		return
	}
	defer jsonFile.Close()

	var conf Config
	decoder := json.NewDecoder(jsonFile)
	err = decoder.Decode(&conf)
	if err != nil {
		log.Printf("decode error [ %v ]", err)
		return
	} else {
		fmt.Println(conf.Name)
		fmt.Println(conf.Blog)
		fmt.Println(conf.Repo)
	}
}
```

输出：

```
gukaifeng
https://gukaifeng.cn
git@github.com:gukaifeng/hexo.git
```

请注意，我们第 10-14 行定义的 Config，其中的成员顺序，无需与 json 文件中的顺序一致。

Go 语言中的 json 包会自动识别 json 项到相应的成员中（json 项与成员变量名对应，不区分大小写），所以我们在 33-35 行中打印出了正确的值。





### 2.2. 有多组 json 项



我们上面的 json 示例中，只有一组项，即 name/blog/repo，我们看看有多组项的时候如何解析。

有多组项的时候，不论是外层还是里层的值，只要把存储解析的变量改为数组即可。





#### 2.2.1. 外层多组 json 项

我们修改 2.1 中的  `myinfo.json` 如下：

```json
[
    {
        "name": "gukaifeng",
        "blog": "https://gukaifeng.cn",
        "repo": "git@github.com:gukaifeng/hexo.git"
    },
    {
        "name": "gukaifeng2",
        "blog": "https://gukaifeng.cn2",
        "repo": "git@github.com:gukaifeng/hexo.git2"
    }
]
```

我们只需要修改上面代码第 26 行：

```go
var conf Config  ->  var conf []Config
```

即：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
)

type Config struct { // 这里成员的顺序与 json 文件中的顺序不一致
	Repo string
	Blog string
	Name string
}

func main() {
	jsonPath := "./myinfo.json"

	jsonFile, err := os.Open(jsonPath) // 打开 json 文件
	if err != nil {
		log.Printf("open json file %v error [ %v ]", jsonPath, err)
		return
	}
	defer jsonFile.Close()

	var conf []Config
	decoder := json.NewDecoder(jsonFile) // 创建 json 解码器
	err = decoder.Decode(&conf)          // 解码 json 文件
	if err != nil {
		log.Printf("decode error [ %v ]", err)
		return
	} else {
		fmt.Println(conf)
	}
}
```

输出如下：

```
[{git@github.com:gukaifeng/hexo.git https://gukaifeng.cn gukaifeng} {git@github.com:gukaifeng/hexo.git2 https://gukaifeng.cn2 gukaifeng2}]
```





#### 2.2.2. 内层多组 json 项

同样的，我们修改 2.1 中的 `myinfo。json` 如下：

```json
{
    "name": "gukaifeng",
    "blog": "https://gukaifeng.cn",
    "repo": [
        "git@github.com:gukaifeng/hexo.git",
        "git@github.com:gukaifeng/hexo.git2",
        "git@github.com:gukaifeng/hexo.git3"
    ]
}
```

现在项 repo 里有多个值，我们看修改后的代码：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
)

type Config struct { // 这里成员的顺序与 json 文件中的顺序不一致
	Repo []string
	Blog string
	Name string
}

func main() {
	jsonPath := "./myinfo.json"

	jsonFile, err := os.Open(jsonPath)
	if err != nil {
		log.Printf("open json file %v error [ %v ]", jsonPath, err)
		return
	}
	defer jsonFile.Close()

	var conf Config
	decoder := json.NewDecoder(jsonFile)
	err = decoder.Decode(&conf)
	if err != nil {
		log.Printf("decode error [ %v ]", err)
		return
	} else {
		fmt.Println(conf)
	}
}
```

输出：

```
{[git@github.com:gukaifeng/hexo.git git@github.com:gukaifeng/hexo.git2 git@github.com:gukaifeng/hexo.git3] https://gukaifeng.cn gukaifeng}
```

我们只修改第 11 行即可，将我们 struct 中的 Repo 从 string 改为数组 []string。





### 2.3. 举一反三

我们看完了 2.1 和 2.2 节的全部内容，其实就完全可以解决 json 文件解析的问题了。

当然 2.1 和 2.2 节中并没有列出全部的情况~~（毕竟 json 项是可以无限嵌套的）~~。

我们再次修改我们 2.1 节中的 `myinfo.json` 如下：

```json
{
    "name": "gukaifeng",
    "blog": "https://gukaifeng.cn",
    "repo": {
        "repo1": "git@github.com:gukaifeng/hexo.git",
        "repo2": "git@github.com:gukaifeng/hexo.git2",
        "repo3": "git@github.com:gukaifeng/hexo.git3"
    }
}
```

与 2.2.2 中的 json 中 repo 仅有多个值不同，这里的 repo 的值也是一个完整的 json。

对于 repo 项的解析，是和 2.1 中一样的，我们看代码：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
)

type Config struct { // 这里成员的顺序与 json 文件中的顺序不一致
	Repo struct {
		Repo1 string
		Repo2 string
		Repo3 string
	}
	Blog string
	Name string
}

func main() {
	jsonPath := "./myinfo.json"

	jsonFile, err := os.Open(jsonPath)
	if err != nil {
		log.Printf("open json file %v error [ %v ]", jsonPath, err)
		return
	}
	defer jsonFile.Close()

	var conf Config
	decoder := json.NewDecoder(jsonFile)
	err = decoder.Decode(&conf)
	if err != nil {
		log.Printf("decode error [ %v ]", err)
		return
	} else {
		fmt.Println(conf)
	}
}
```

输出：

```
{{git@github.com:gukaifeng/hexo.git git@github.com:gukaifeng/hexo.git2 git@github.com:gukaifeng/hexo.git3} https://gukaifeng.cn gukaifeng}
```



注意第 11-15 行，这里把 Repo 也改为了一个 struct。

这小节要说的就是这些了，json 嵌套的情况是无限多的，避免套娃！我们学会举一反三就可以了！
