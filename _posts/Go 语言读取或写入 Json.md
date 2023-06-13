
Go 语言提供了关于 json 的标准库包 `encoding/json`。

详细官网文档见 [json package - encoding/json - pkg.go.dev](https://pkg.go.dev/encoding/json)，这里只介绍几种常用操作。

<!--more-->

Json 包最常用的场景主要有两个：

1. Json 文件与结构体对象的交互：

   * 将一个 Json 文件中的内容读取到一个结构体对象的成员变量中。

   * 将一个结构体对象的成员变量写入到一个 Json 文件中。

2. 将对象进行 Json 编码/解码：

   * 对一个对象进行 Json 编码。

   * 从 Json 编码构造对象。

Json 文件与结构体对象交互，通常用来读取/写入配置文件，持久化某些信息等。

将对象进行 Json 编码与解码，通常用来在网络上传输传输对象。将对象进行 Json 编码得到一个字节数组 []byte，即字节流，然后进行网络传输，接收方可通过此 Json 编码，构造出该对象。

要注意的是，Json 中字段的 key 本身是不区分大小写的，Go 语言的 Json 包解析 Json 的时候也同样是不区分 key 的大小写的（虽然有些编程语言或包可能区分大小写）。

## 1. Json 文件与结构体对象交互

Json 文件与结构体对象交互，通常用来读取/写入配置文件，持久化某些信息等。

### 1.1. 前置知识：Json 文件中的 key 如何与结构体中的成员一一对应



Json 文件中的 key 是如何与结构体中的成员一一对应，这是很多人第一次接触 Json 编码的疑惑。

在 Go 语言中，**默认情况下**，Json 文件中的 key，对结构体中的成员变量名，一一对应，且**不区分大小写**。但要注意结构体中的这些与 Json 文件读取有关的成员变量名必须是可导出的（即首字母必须大写），不然 Json 库无法读取这些成员。

除了默认情况，我们还可以指定某个结构体对象的成员，对应的 Json 字段 key 是哪个。方法是给成员添加标签，例如：

```go
type Config struct {
	Repo string `json:"myRepo"`
	Blog string
	Name string
}
```

这样，在将此对象写入 Json 文件（或反之）时，成员 `Repo` 对应的 Json 文件中的 key 就是 `myRepo`。

注意 Go 中将对象写入 Json 文件时，key（就是成员名，或标签指定的名）此时是会区分大小写的，即 `Repo` 成员的值会写入到 Json 字段 `myRepo` 中。但反之不会，即不论是 `myRepo`、`MyRepo` 还是 `MYREPO` 等等的 Json 字段，都会正确写入成员 `Repo` 中。

当结构体内有嵌套结构体时（与 Json 中有嵌套 Json 时对应），此时会**递归**的遵循上述规则。

### 1.2. 将结构体对象中的成员写入 Json 文件

编码分为两步：

1. 创建一个新的 json 文件；
2. 将数据结构中的内容按格式写入 json 文件。

第二步的写入，首先要使用 `json.NewEncoder()` 创建一个编码器，其参数是我们新创建的 json 文件指针。然后编码要使用编码器中的 `Encode()` 方法，其参数是我们要编码进 json 的内容。

我们看代码（重点关注有注释的语句）：

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





### 1.3. 从 Json 文件中读取内容到结构体对象

同样的，解码也是分两步：

1. 打开待解码的 json 文件；
2. 使用 json 包提供的方法解码 json 文件到数据结构中。

第 2 步的解码首先要使用 `json.NewDecoder()` 创建一个解码器，其参数是我们打开的 json 文件指针。然后解码要使用解码器中的 `Decode()` 方法，其参数是将要存储解码信息的数据结构对象。

我们下面通过具体例子来解释。

#### 1.3.1. 只有一组 json 项



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

type Config struct { // 这里成员的顺序与 json 文件中的顺序无需一致
	Repo string
	Blog string
	Name string
}

func main() {
	jsonPath := "./myinfo.json"

	jsonFile, err := os.Open(jsonPath)  // 打开 json 文件
	if err != nil {
		log.Printf("open json file %v error [ %v ]", jsonPath, err)
		return
	}
	defer jsonFile.Close()

	var conf Config
	decoder := json.NewDecoder(jsonFile) // 创建 json 解码器
	err = decoder.Decode(&conf)          // 解码 json 文件
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





#### 1.3.2. 有多组 json 项



我们上面的 json 示例中，只有一组项，即 name/blog/repo，我们看看有多组项的时候如何解析。

有多组项的时候，不论是外层还是里层的值，只要把存储解析的变量改为数组即可。





##### 1.3.2.1. 外层多组 json 项

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

type Config struct { // 这里成员的顺序与 json 文件中的顺序无需一致
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





##### 1.3.2.2. 内层多组 json 项

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

type Config struct { // 这里成员的顺序与 json 文件中的顺序无需一致
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

输出：

```
{[git@github.com:gukaifeng/hexo.git git@github.com:gukaifeng/hexo.git2 git@github.com:gukaifeng/hexo.git3] https://gukaifeng.cn gukaifeng}
```

我们只修改第 11 行即可，将我们 struct 中的 Repo 从 string 改为数组 []string。





#### 1.3.3. 举一反三

我们看完了 1.2.1 和 1.2.2 节的全部内容，其实就完全可以解决 json 文件解析的问题了。

当然 1.2.1 和 1.2.2 节中并没有列出全部的情况~~（毕竟 json 项是可以无限嵌套的）~~。

其实我们只要理解，Json 文件中的项与结构体对象的成员是**递归**对应的，就可以了。

我们再次修改我们 1.2.1 节中的 `myinfo.json` 如下：

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

与 1.2.2.2 中的 json 中 repo 仅有多个值不同，这里的 repo 的值也是一个完整的 json。

对于 repo 项的解析，是和 1.2.1 中一样的，我们看代码：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
)

type Config struct { // 这里成员的顺序与 json 文件中的顺序无需一致
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

	jsonFile, err := os.Open(jsonPath)  // 打开 json we
	if err != nil {
		log.Printf("open json file %v error [ %v ]", jsonPath, err)
		return
	}
	defer jsonFile.Close()

	var conf Config
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

输出：

```
{{git@github.com:gukaifeng/hexo.git git@github.com:gukaifeng/hexo.git2 git@github.com:gukaifeng/hexo.git3} https://gukaifeng.cn gukaifeng}
```



注意第 11-15 行，这里把 Repo 也改为了一个 struct。

这小节要说的就是这些了，json 嵌套的情况是无限多的，避免套娃！我们学会举一反三就可以了！





## 2. 将对象进行 Json 编码/解码



将对象进行 Json 编码与解码，通常用来在网络上传输传输对象。

将对象进行 Json 编码得到一个字节切片 []byte，即字节流，然后进行网络传输，接收方可通过此 Json 编码，构造出该对象。**这个过程与其他语言中的序列化/反序列化非常相似。**

我们在 1.1 小节介绍了 Json 中的字段与对象中的成员的对应关系，这些规则对本小节仍然适用。但由于此小节的场景通常不在乎这些对应关系，我们只关注将对象转换成字节流、从字节流构造对象，所以这里就不关注那些了。

### 2.1. 将对象转换为 Json 编码

```go
func Marshal(v any) ([]byte, error)
```

`Marshal()` 返回 `v` 的 JSON 编码（`v` 是对象或指针都可以）。

我们看一段代码示例：

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	type ColorGroup struct {
		ID     int
		Name   string
		Colors []string
	}
	group := ColorGroup{
		ID:     1,
		Name:   "Reds",
		Colors: []string{"Crimson", "Red", "Ruby", "Maroon"},
	}
	b, err := json.Marshal(group)
	if err != nil {
		fmt.Println("error:", err)
	}
	fmt.Printf("%s\n", b)
}
```

输出：

```go
{"ID":1,"Name":"Reds","Colors":["Crimson","Red","Ruby","Maroon"]}
```

可以看到，字节切片 `b` 以 Json 编码存储了对象 `group` 中的内容（我们要理解一点，不论多复杂的自定义类型，都是由各种基本类型构成的，所以不论怎样的结构体对象，都一定可以进行 Json 编码）。

我们在使用网络相关包、工具传输数据时，就可以使用 `b` 了，这里就不解释了，不是本文重点。





### 2.2. 从 Json 编码构造对象

```go
func Unmarshal(data []byte, v any) error
```



`Unmarshal()` 解析了 JSON 编码的 `data`，并将结果存储在 `v` 指向的值中（`v` 必须为一个指针）。

我们看一段代码示例，这段代码示例与 2.1 反向：

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	type ColorGroup struct {
		ID     int
		Name   string
		Colors []string
	}
  // 假装这个 b 来自网络（实际是我从 2.1 复制过来的）
	var b = []byte(`{"ID":1,"Name":"Reds","Colors":["Crimson","Red","Ruby","Maroon"]}`)
	var group ColorGroup
	err := json.Unmarshal(b, &group)
	if err != nil {
		fmt.Println("error:", err)
	}
	fmt.Printf("%v\n", group)
}
```

输出：

```go
{1 Reds [Crimson Red Ruby Maroon]}
```

可以看到我们成功通过字节切片 `b` 构建了对象 `group`。
