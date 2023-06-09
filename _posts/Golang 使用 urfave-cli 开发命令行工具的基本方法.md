---
title: "Golang 使用 urfave/cli 开发命令行工具的基本方法"
date: 2023-06-10 02:03:00
updated: 2023-06-10 02:03:00
categories:
- 编程语言基础
- Golang
tags: [Golang]
---



## 1. 从一个项目模板开始



首先创建一个目录，目录结构如下：

```shell
go-cli-example
└── cmd
   └── cmd.go
```

我这里项目名字为 “go-cli-example”。

在项目根目录执行 `go mod` 初始化操作，我这里为：

```go
go mod init go-cli-example
```

我这里是随意起的项目名字，如果你的是正常的可维护项目，“go-cli-example” 可能应当替换为形如 "github.com/gukaifeng/go-cli-example" 这样。

我这里 **cmd.go** 就是入口文件了，我们先从一个 `urfave/cli` 模板开始：

```go
package cmd

import (
	"fmt"
	"os"

	"github.com/urfave/cli/v2"
)

func main() {
	app := &cli.App{
		Name:  "greet",
		Usage: "say a greeting",
		Action: func(c *cli.Context) error {
			fmt.Println("Greetings")
			return nil
		},
	}

	app.Run(os.Args)
}
```



如果你没有使用过 `urfave/cli` 包，那么需要在命令行执行 `go get` 下载包（如果系统里有这个包了就可以跳过）：



```shell
go get -u github.com/urfave/cli
```



然后在项目根目录执行 mod 整理命令：

```shell
go mod tidy
```



到这里，你的目录结构应该是下面这样的并且没有任何报错：



```
go-cli-example
├── cmd
│   └── cmd.go
├── go.mod
└── go.sum
```



然后我们运行这个程序：



```shell
$ go run cmd/cmd.go
Greetings
```

输出 "Greetings"，表示程序正确运行了。





## 2. `urfave/cli` 模板分析



我们看刚刚的 `main()` 方法中是如何使用 `urfave/cli` 的。



### 2.1. `cli.App` 对象的属性



```go
func main() {
	app := &cli.App{
		Name:  "greet",
		Usage: "say a greeting",
		Action: func(c *cli.Context) error {
			fmt.Println("Greetings")
			return nil
		},
	}

	app.Run(os.Args)
}
```

首先我们需要创建一个 `cli.App` 的对象，并操作其地址指针，即 ：

```go
app := &cli.app { ... }
```

我们看 `cli.App` 对象的几个基本属性：

* `Name`：此程序的名字，这个名字只用来显示在程序信息或帮助信息里。我们实际执行此程序的时候，用的是编译出的二进制文件，二进制的文件名字并不是一定要和这个 `cli.App.Name` 一样的，只是通常来说我们会设定成一样的，但这不是强制要求。

* `Usgae`：此程序的简介。这个 “Usage” 虽然是用法的意思，但后面会说到，每个参数都会有一个 “Usage”，通常会在每个参数的 “Usage” 里详细介绍每个参数的用法。而在这里，一般都是一句或几句话简单介绍此程序。

* `Action`：当未使用子命令时，此程序要进行的动作。其值应当为一个 `cli.ActionFunc` 函数：

  ```go
  type ActionFunc func(*Context) error
  ```

  在上面的例子中，我们使用了一个匿名函数，实际开发中通常不会这样使用（除非动作真的很简单）。



下面是将 `cli.App.Action` 中的匿名函数分离出来的修改：



```go
package main

import (
	"fmt"
	"os"

	"github.com/urfave/cli/v2"
)

func greet(c *cli.Context) error {
	fmt.Println("Greetings")
	return nil
}

func main() {
	app := &cli.App{
		Name:   "greet",
		Usage:  "say a greeting",
		Action: greet,
	}

	app.Run(os.Args)
}
```

使用 `urfave/cli` 实现的命令行程序会有一个默认参数 `--help` `-h`，可以查看我们上面设定的相关信息：

```shell
$ go run cmd/cmd.go -h
NAME:
   greet - say a greeting

USAGE:
    [global options] command [command options] [arguments...]

COMMANDS:
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h  show help
```



这里也可以再次解释一下为什么上面的 `cli.App.Usgae` 我建议设定为程序的简介。我们可以看到 `cli.App.Usage` 的值是显示在 `cli.App.Name` 的同一行的右侧的，这里并不适合完整的介绍此程序的使用方法，而更适合对程序进行简单的一点介绍。







### 2.2. 启动程序 `cli.App.Run()`





然后 `app` 变量就是我们的命令行程序的入口，启动的方法是固定的：

```shell
app.Run(os.Args)
```

这里 `app.Run()` 接收 `[]string` 切片，`os.Args` 是 `os` 包提供的一个 `[]string` 切片，我们在命令行里传递的参数都会存储在 `os.Args` 中。到这里命令行工具就算是开始工作了。



事实上 `app.Run()` 是有返回值的，其声明如下：

```go
func (*cli.App).Run(arguments []string) (err error)
```



我们是有必要处理这个返回的错误的，因为 `error` 可能会通过内部执行的方法传递出来。如果你的命令行程序期望是长期运行，那么可能应当使用日志的方式记录这个错误；如果你的命令行程序是工具性质的，即用即停，那么可以直接将错误信息输出到终端中。这里采用后者的方案，修改代码：



```go
package main

import (
	"fmt"
	"os"

	"github.com/urfave/cli/v2"
)

func greet(c *cli.Context) error {
	fmt.Println("Greetings")
	return nil
}

func main() {
	app := &cli.App{
		Name:   "greet",
		Usage:  "say a greeting",
		Action: greet,
	}

	if err := app.Run(os.Args); err != nil {
		fmt.Fprintf(os.Stderr, "%v run error - %v\n", app.Name, err)
	}
}
```





## 3. 为程序添加参数



作为命令行程序，参数是重中之重，我们看看如何为 `urfave/cli` 程序添加参数。





### 3.1. 定义参数



`cli.App` 中的属性 `Flags` 定义了当前命令的参数（目前我们没有设定子命令，所以就是主程序的参数，如果后面设定子命令并在子命令的字段中设定 `Flags`，那么相应的就是定义子命令的参数）。`Flags` 是 `[]cli.Flag` 切片，我们一般直接在 `Flags` 字段值里定义，以提高程序可读性。我们看下面的修改：



```go
package main

import (
	"fmt"
	"os"

	"github.com/urfave/cli/v2"
)

func greet(c *cli.Context) error {
	name := c.String("name")
	times := 1
	if c.Bool("t") {
		times = 3
	}
	for i := 0; i < times; i++ {
		fmt.Println("Greetings", name)
	}
	return nil
}

func main() {
	app := &cli.App{
		Name:   "greet",
		Usage:  "say a greeting",
		Action: greet,
		Flags: []cli.Flag{
			&cli.StringFlag{
				Name:  "name",
				Usage: "greetings to whom",
			},
			&cli.BoolFlag{
				Name:    "three",
				Aliases: []string{"t", "3times"},
				Value:   false,
				Usage:   "whether to greet three times",
			},
		},
	}

	if err := app.Run(os.Args); err != nil {
		fmt.Fprintf(os.Stderr, "%v run error: %v\n", app.Name, err)
	}
}
```



这段代码中，我们为主程序添加了两个参数：

```go
Flags: []cli.Flag{
    &cli.StringFlag{
        Name:  "name",
        Usage: "greetings to whom",
    },
    &cli.BoolFlag{
        Name:    "three",
        Aliases: []string{"t", "3times"},
        Value:   false,
        Usage:   "whether to greet three times",
    },
},
```

可以从名字看出，`&cli.StringFlag` 代码块内描述的是一个 `string` 类型的参数，`&cli.BoolFlag` 代码块内描述的是一个 `bool` 类型的参数。

* `Name`：参数名。
* `Usage`：参数的描述。
* `Aliases`：参数的其他名称。这是一个 `[]string` 切片，里面所有的值都与 `Name` 字段里设定的等价。
* `Value`：参数的默认值。如果参数为设定默认值，又没有在命令行中传入，那么其值将为其类型的零值。



作为一个问候程序（本文中示例程序名为 "greet"），`string` 类型参数 "name" 参数表示问候谁，`bool` 类型参数 "three"、"t"、"3time" 表示是否要问候三次。

### 3.2. 根据参数改变动作

我们首先需要在动作函数 `cli.App.Action` 里，接收我们设定的参数，再根据参数决定具体要做什么。

```go
func greet(c *cli.Context) error {
	name := c.String("name")
	times := 1
	if c.Bool("t") {
		times = 3
	}
	for i := 0; i < times; i++ {
		fmt.Println("Greetings", name)
	}
	return nil
}
```

如代码所示，`cli.Context.String()` 用于接收 `string` 类型的参数，`cli.Context.Bool()` 用于接收 `bool` 类型的参数。



不论 `cli.Context.String()` 还是 `cli.Context.Bool()`，其都接受一个字符串参数，对应之前在 `Flags` 设定的各个参数的 `Name` 字段或 `Aliases` 字段（这两个字段设定的系列名称，都可以无差别的对待的，没有区别），然后返回这个参数的值。如果没有传递这个参数，那么就令其为默认值，如果也没有设定默认值，则为参数类型的零值。如果获取一个不存在的参数，则也为零值。我们也可以通过 `cli.Context.IsSet()` 方法来判断一个参数是否确实设置了（比如有些时候零值是此参数的一个有效的值），这里就不举例了。



当我们拿到参数的值后，我们就可以根据不同的值做不同的动作了。



### 3.3. 在命令行中传递参数



我们定义完了参数，也引用了参数，然后就是在使用时如何传递参数了。

默认的 `-h` 参数，可以打印出我们定义的参数的用法：

```shell
NAME:
   greet - say a greeting

USAGE:
   greet [global options] command [command options] [arguments...]

COMMANDS:
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --name value           greetings to whom
   --three, -t, --3times  whether to greet three times (default: false)
   --help, -h             show help
```

可以看到这里列出了我们刚刚定义的参数、描述、默认值等。

这里可以先看到一点，就是如果参数的长度大于 1，如 "name"、"three"、"3times"、"help"，则指定参数时前面是 `--`（两个 `-`）。如果参数长度为 1，如 '"t"、"h"，则指定参数时前面是 `-`（一个 `-`）。



我这里举几个例子，我们从例子来理解。



首先，指定参数时，参数与值可以用空格隔开，也可以用 `=` 连接，如下两种方法完全等价：

```shell
$ go run cmd/cmd.go --name gukaifeng --three
Greetings gukaifeng
Greetings gukaifeng
Greetings gukaifeng
```

```shell
$ go run cmd/cmd.go --name=gukaifeng --three
Greetings gukaifeng
Greetings gukaifeng
Greetings gukaifeng
```



同一个参数的不同名称完全等价：



```shell
$ go run cmd/cmd.go --name gukaifeng -t
Greetings gukaifeng
Greetings gukaifeng
Greetings gukaifeng
```

```shell
$ go run cmd/cmd.go --name gukaifeng --three=true
Greetings gukaifeng
Greetings gukaifeng
Greetings gukaifeng
```

```shell
$ go run cmd/cmd.go --name gukaifeng --3times=false
Greetings gukaifeng
```

注意 `bool` 类型的参数一般是不需要接值的，不管其默认值 `cli.BoolFlag.Value` 是什么，只要设定了这个参数，其值就是 true。例如 `-t` `-three=true` 都是等价的，都是 true（除非指定为 false，如`--3times=false`）。**所以我们可以用 `bool` 类型实现无值参数。**



如果在命令行中传入了不存在的参数，或传入了相互之间不能兼容一起使用的参数，则会有报错，这里就不举这个例子了。



### 3.4. `urfave/cli` 支持的参数类型



我们前面定义了参数（`cli.StringFlag`，`cli.BoolFlag`），以及相应的获取参数值方法（`cli.Context.String()`，`cli.Context.Bool()`）。不过 `urfave/cli` 支持的参数类型还有很多，这里列出一下参数类型的 `Flag`，以及相应的获取参数值的方法。





| 序号 | 参数类型    | Flag                   | 获取参数值的方法             |
| ---- | ----------- | ---------------------- | ---------------------------- |
| 1    | `string`    | `cli.StringFlag`       | `cli.Context.String()`       |
| 2    | `[]string`  | `cli.StringSliceFlag`  | `cli.Context.StringSlice()`  |
| 3    | `bool`      | `cli.BoolFlag`         | `cli.Context.Bool()`         |
| 4    | `int`       | `cli.IntFlag`          | `cli.Context.Int()`          |
| 5    | `int64`     | `cli.Int64Flag`        | `cli.Context.Int64()`        |
| 6    | `uint`      | `cli.UintFlag`         | `cli.Context.Uint()`         |
| 7    | `uint64`    | `cli.Uint64Flag`       | `cli.Context.Uint64()`       |
| 8    | `float64`   | `cli.Float64Flag`      | `cli.Context.Float64()`      |
| 9    | `[]int`     | `cli.IntSliceFlag`     | `cli.Context.IntSlice()`     |
| 10   | `[]int64`   | `cli.Int64SliceFlag`   | `cli.Context.Int64Slice()`   |
| 11   | `[]uint`    | `cli.UintSliceFlag`    | `cli.Context.UintSlice()`    |
| 12   | `[]uint64`  | `cli.Uint64SliceFlag`  | `cli.Context.Uint64Slice()`  |
| 13   | `[]float64` | `cli.Float64SliceFlag` | `cli.Context.Float64Slice()` |
| 15   | `Path` *    | `cli.PathFlag`         | `cli.Context.Path()`         |
| 16   | `Timestamp` | ``cli.TimestampFlag``  | `cli.Context.Timestamp()`    |
| 17   | `Duration`  | ``cli.DurationFlag``   | `cli.Context.Duration()`     |

\* 参数类型 `Path` 和 `string` 其实是没有区别的，在源码里有 `type Path string`，并且相关方法也就是换了个名字，并没有额外的什么操作。我的理解就是 `Path` 类型是用于提升代码可读性的，专门用来表示“路径”这一含义。喜欢就用，直接用 `string` 类型也无妨。



---

除了上面内置的参数类型意外，`urfave/cli` 还支持一种通用的参数类型：

| 序号 | 参数类型  | Flag              | 获取参数值的方法        |
| ---- | --------- | ----------------- | ----------------------- |
| 18   | `Generic` | `cli.GenericFlag` | `cli.Context.Generic()` |

类型 `Generic` 是一个接口类型，实现了此接口的类型都可以用这个参数。

```go
type Generic interface {
	Set(value string) error
	String() string
}
```

这里不深入介绍 `Generic` 类型了，因为 `urfave/cli` 提供的内置参数类型通常已经足够使用了。





### 3.5. 参数 Flag 的其他属性

我们前面提到的 `cli.BoolFlag` 或是 `cli.StringFlag` 等等各种参数 Flag，其实都是 `cli.Flag` 接口的实现。这些参数 Flag 除了上面例子列出的几个属性，还有一些其他的。不过这里不打算介绍每个参数 Flag 都还有哪些属性，而是挑几个我认为比较有用的、通用的列出来（实际还有很多，但是我认为不常用，感兴趣的话读者可以自己看看官方的文档）。



前面提到过的属性，如：

```go
&cli.BoolFlag{
    Name:    "three",
    Aliases: []string{"t", "3times"},
    Value:   false,
    Usage:   "whether to greet three times",
},
```

一些可能有用但没有举例的：

* `Category`=string：参数的分类。设定了分类以后，在打印帮助信息时，同一分类的参数会被分组到一起。
* `Required`=bool：此参数是否是必须的，默认为 false。若为 true，不指定此参数的话会报错（输出帮助信息）。
* `Hidden`=bool：是否隐藏此参数，默认为 false。若为 true，则此参数不会显示在 `--help` `-h` 帮助信息里。
* `EnvVars`=[]string：此参数对应的环境变量，即可以从环境变量里取值。
* `Destination`：这是一个与参数类型一样的指针，如果设定了此属性，那么我们传入的参数会在这个指针指向的对象中也存储一份。
* `Action`：一个执行函数，当**显式**设定了参数时，会执行这个函数。





`Action` 函数声明如下（其中 `argType` 是参数类型，第二个参数就是传入的此参数的值）：

```go
Action func(*Context, argType) error
```

这些用法比较简单，这里就不举例了。



## 4. 添加子命令以及子命令参数



我们前面介绍的只有一个主命令，主命令即为二进制文件的名字（前面的例子里我们没有编译二进制文件，当时为 `go run cmd/cmd.go`），但很多场景下我们是需要子命令来更细粒度地控制程序的动作的。

主命令和子命令本质上没有差别，主命令的子命令在 `cli.App.Commands` 中定义，子命令的子命令（理论上可以无限递归）在  `cli.Command.Subcommands` 中定义，二者类型均为 `[]*cli.Command`。

下面看一段官方的示例代码：



```go
package main

import (
    "fmt"
    "log"
    "os"

    "github.com/urfave/cli/v2"
)

func main() {
    app := &cli.App{
        Commands: []*cli.Command{
            {
                Name:    "add",
                Aliases: []string{"a"},
                Usage:   "add a task to the list",
                Action: func(cCtx *cli.Context) error {
                    fmt.Println("added task: ", cCtx.Args().First())
                    return nil
                },
            },
            {
                Name:    "complete",
                Aliases: []string{"c"},
                Usage:   "complete a task on the list",
                Action: func(cCtx *cli.Context) error {
                    fmt.Println("completed task: ", cCtx.Args().First())
                    return nil
                },
            },
            {
                Name:    "template",
                Aliases: []string{"t"},
                Usage:   "options for task templates",
                Subcommands: []*cli.Command{
                    {
                        Name:  "add",
                        Usage: "add a new template",
                        Action: func(cCtx *cli.Context) error {
                            fmt.Println("new task template: ", cCtx.Args().First())
                            return nil
                        },
                    },
                    {
                        Name:  "remove",
                        Usage: "remove an existing template",
                        Action: func(cCtx *cli.Context) error {
                            fmt.Println("removed task template: ", cCtx.Args().First())
                            return nil
                        },
                    },
                },
            },
        },
    }

    if err := app.Run(os.Args); err != nil {
        log.Fatal(err)
    }
}
```

下面是这段代码的程序的一些使用示例：

```shell
$ go run cmd/cmd.go add task-test
added task:  task-test

$ go run cmd/cmd.go a task-test  
added task:  task-test

$ go run cmd/cmd.go complete task-test
completed task:  task-test

$ go run cmd/cmd.go c task-test       
completed task:  task-test

$ go run cmd/cmd.go template add template-test
new task template:  template-test

$ go run cmd/cmd.go template remove template-test
removed task template:  template-test

$ go run cmd/cmd.go t remove template-test 
removed task template:  template-test
```

不难看出定义子命令的方法，而且是可以递归定义子命令的，只有第一个子命令使用 `cli.App.Commands` 定义，再递归下的子命令使用 `cli.Command.Subcommands` 定义，二者本质没有区别，类型都是 `[]*cli.Command`。所以我们这里只需要介绍下 `cli.Command` 类型的属性就可以了。



我们节选上面的一段用于说明：

```go
{
    Name:    "template",
    Aliases: []string{"t"},
    Usage:   "options for task templates",
    Subcommands: []*cli.Command{
        {
            Name:  "add",
            Usage: "add a new template",
            Action: func(cCtx *cli.Context) error {
                fmt.Println("new task template: ", cCtx.Args().First())
                return nil
            },
        },
        {
            Name:  "remove",
            Usage: "remove an existing template",
            Action: func(cCtx *cli.Context) error {
                fmt.Println("removed task template: ", cCtx.Args().First())
                return nil
            },
            
        },
    },
},
```

* `Name`=string：子命令名。
* `Aliases`=[]string：子命令别名。
* `Usage`=string：子命令简介。
* `Subcommands`：下级的子命令。这里就递归下去了，不再介绍。



我们也可以为子命令设置参数（上述官方示例没有，但前面介绍过了，很简单）：

* `Flags`：子命令的参数，与我们在第 3 小节介绍的功能与用法完全一致，不再介绍。







可以看得出，子命令的定义是可以递归下去的，但是用法上和主命令基本上是一样的。





## 5. 设定程序版本



我们在使用其他工具时，一定见过输出软件版本的命令参数，或是 `--version`、或是 `-v` 或是 `-V` 等等。`urfave/cli` 同样支持，我们只需要为 `cli.App` 添加 `Version` 属性即可。

```go
package main

import (
	"log"
	"os"

	"github.com/urfave/cli/v2"
)

func main() {
	app := &cli.App{
		Name:    "greet",
		Version: "1.0.0",
	}

	if err := app.Run(os.Args); err != nil {
		log.Fatal(err)
	}
}
```

`urfave/cli` 会提供默认的查看版本参数，为 `--version` 或 `-v`，例如我们查看上面这段程序的版本：

```shell
$ go run cmd/cmd.go --version         
greet version 1.0.0

$ go run cmd/cmd.go -v         
greet version 1.0.0
```

可以看到默认显示的版本信息格式为：

````
cli.App.Name + "version" + cli.App.Name.Version
````

如果没有额外的需求，那么这样使用就可以了。

但有时我们期望打印更详细的版本信息，那么就需要自己定制下查看版本命令的输出。

我们看下面这段代码：

```go
package main

import (
	"fmt"
	"os"

	"github.com/urfave/cli/v2"
)

var (
	Version  = "v1.0.2"
	Revision = "fafafaf"
)

func main() {
	cli.VersionPrinter = func(cCtx *cli.Context) {
		fmt.Printf("version %s.%s\n", cCtx.App.Version, Revision)
	}

	app := &cli.App{
		Name:    "partay",
		Version: Version,
	}
	app.Run(os.Args)
}
```

即可以通过修改 `cli.VersionPrinter` 来定制输出的版本信息。测试输出如下：

```shell
$ go run cmd/cmd.go --version
version v1.0.2.fafafaf

$ go run cmd/cmd.go -v       
version v1.0.2.fafafaf
```





我们还可以通过修改 `cliVersionFlag` 来改变打印版本信息的参数：

```go
package main

import (
	"fmt"
	"os"

	"github.com/urfave/cli/v2"
)

var (
	Version  = "v1.0.2"
	Revision = "fafafaf"
)

func main() {
	cli.VersionPrinter = func(cCtx *cli.Context) {
		fmt.Printf("%s version %s.%s\n", cCtx.App.Name, cCtx.App.Version, Revision)
	}
	cli.VersionFlag = &cli.BoolFlag{
		Name:               "my-version",
		Aliases:            []string{"my-v"},
		Usage:              "print the version",
		DisableDefaultText: true,
	}

	app := &cli.App{
		Name:    "partay",
		Version: Version,
	}
	app.Run(os.Args)
}
```

测试输出：

```shell
$ go run cmd/cmd.go --my-version
partay version v1.0.2.fafafaf
$ go run cmd/cmd.go --my-v       
partay version v1.0.2.fafafaf
```





## 6. 命令行自动补全





`urfave/cli` 提供了默认的命令自动补全的设置，也支持自定义命令补全的策略。



`urfave/cli` 默认提供的自动补全一般情况下已经足够使用了，所以这里就用默认的。不过要注意，`urfave/cli` 自动配置的自动补全其实是不够准确的，例如可能会多出一些候选项，如果你刚需准确的自动补全，请参照官方文档进行自定义自动补全策略。



另外使用自动补全需要先将项目编译成一个二进制文件，使用这个二进制文件进行配置。前面的代码示例中都是直接用 `go run`，是不能配置自动补全的。



**下面的小节都假定你已经编译好了二进制文件，并将二进制文件放到了 `$PATH` 中。**

### 6.1. 代码中的配置



启动自动补全只需要在 `cli.App` 中设置参数：

```go
EnableBashCompletion: true,
```

我们以下面这个代码为例：

```go
package main

import (
    "fmt"
    "log"
    "os"

    "github.com/urfave/cli/v2"
)

func main() {
    app := &cli.App{
        EnableBashCompletion: true,
        Commands: []*cli.Command{
            {
                Name:    "add",
                Aliases: []string{"a"},
                Usage:   "add a task to the list",
                Action: func(cCtx *cli.Context) error {
                    fmt.Println("added task: ", cCtx.Args().First())
                    return nil
                },
            },
            {
                Name:    "complete",
                Aliases: []string{"c"},
                Usage:   "complete a task on the list",
                Action: func(cCtx *cli.Context) error {
                    fmt.Println("completed task: ", cCtx.Args().First())
                    return nil
                },
            },
            {
                Name:    "template",
                Aliases: []string{"t"},
                Usage:   "options for task templates",
                Subcommands: []*cli.Command{
                    {
                        Name:  "add",
                        Usage: "add a new template",
                        Action: func(cCtx *cli.Context) error {
                            fmt.Println("new task template: ", cCtx.Args().First())
                            return nil
                        },
                    },
                    {
                        Name:  "remove",
                        Usage: "remove an existing template",
                        Action: func(cCtx *cli.Context) error {
                            fmt.Println("removed task template: ", cCtx.Args().First())
                            return nil
                        },
                    },
                },
            },
        },
    }

    if err := app.Run(os.Args); err != nil {
        log.Fatal(err)
    }
}
```



---



在 `urfave/cli` 的 [GitHub](https://github.com/urfave/cli) 仓库中，官方提供了用于由 `urfave/cli` 编写的命令行程序的自动补全脚本，在 `autocomplete/` 目录下：



```shell
autocomplete/
├── bash_autocomplete
├── powershell_autocomplete.ps1
└── zsh_autocomplete
```

从这三个自动补全脚本的名字不难看出，这三个脚本用于 Bash、Zsh 和 PowerShell。

### 6.2. 在 Bash 中支持





**注意这里已假定你已经编译好了二进制文件，并将二进制文件放到了 `$PATH` 中。**

#### 6.2.1. 下载补全脚本 



首先下载官方的 Bash 自动补全脚本，我这里下载到项目的根目录下的 `autocomplete/` 文件夹内：

```shell
curl -o autocomplete/bash_autocomplete https://raw.githubusercontent.com/urfave/cli/main/autocomplete/bash_autocomplete --create-dirs
```



#### 6.2.2. 临时支持自动补全

临时支持自动补全仅适用于当前 Shell 窗口：

```shell
PROG=<myprogram> source autocomplete/bash_autocomplete
```

注意这里 `<myprogram>` 应该换成你的二进制文件名字，然后在当前 Shell 窗口，就可以进行自动补全了。







#### 6.2.3. 永久支持自动补全

```shell
sudo cp autocomplete/bash_autocomplete /etc/bash_completion.d/<myprogram>
source /etc/bash_completion.d/<myprogram>
```

注意这里 `<myprogram>` 应该换成你的二进制文件名字，然后就可以永久进行自动补全了。



### 6.3. 在 Zsh 中支持





**注意这里已假定你已经编译好了二进制文件，并将二进制文件放到了 `$PATH` 中。**





#### 6.3.1. 下载补全脚本



首先下载官方的 Zsh 自动补全脚本，我这里下载到项目的根目录下的 `autocomplete/` 文件夹内：

```shell
curl -o autocomplete/zsh_autocomplete https://raw.githubusercontent.com/urfave/cli/main/autocomplete/zsh_autocomplete --create-dirs
```

#### 6.3.2. 临时支持自动补全

临时支持自动补全仅适用于当前 Shell 窗口：

```shell
PROG=<myprogram> source autocomplete/zsh_autocomplete
```

注意这里 `<myprogram>` 应该换成你的二进制文件名字，然后在当前 Shell 窗口，就可以进行自动补全了。





#### 6.3.3. 永久支持自动补全

```shell
sudo cp autocomplete/zsh_autocomplete /etc/zsh_completion.d/<myprogram>
source /etc/zsh_completion.d/<myprogram>
```

注意这里 `<myprogram>` 应该换成你的二进制文件名字，然后就可以永久进行自动补全了。

### 6.4. 在 PowerShell 中支持





**注意这里已假定你已经编译好了二进制文件，并将二进制文件放到了 `$PATH` 中。**



#### 6.4.1. 下载补全脚本



首先下载官方的 PowerShell  自动补全脚本，我这里下载到项目的根目录下的 `autocomplete/` 文件夹内：



```shell
curl -o autocomplete/powershell_autocomplete.ps1 https://raw.githubusercontent.com/urfave/cli/main/autocomplete/powershell_autocomplete.ps1 --create-dirs
```

#### 6.4.2. 临时支持自动补全

首先将下载的 `powershell_autocomplete.ps1 ` 改名为 `<myprogrem>.ps1`，注意 `myprogrem` 应当替换为你的程序名字。

然后执行：

```shell
& autocomplete/<my program>.ps1
```

就可以在当前的 PowerShell 进行自动补全了。

#### 6.4.3. 永久支持自动补全



我们打开 `$profile`（使用命令 `code $profile` 或 `notepad $profile` 或其他的只要可以打开都可以）。

在里面添加一行：

```shell
& path/to/autocomplete/<my program>.ps1
```

注意这里要正确配置 ps1 脚本的名字和路径，然后就可以进行永久的自动补全了。

## 7. 一个推荐的开始模板



我写了一个 `urfave/cli` 的开始模板，大部分命令行工具的开发都可以从这个模板开始。

你可以从此模板开始，并增加或删除适合你自己项目的内容。

我对此模板进行了一点简单的设计，其目录结构如下：

```
.
├── autocomplete
│   ├── bash_autocomplete
│   ├── powershell_autocomplete.ps1
│   └── zsh_autocomplete
├── cmd
│   ├── cmd.go
│   ├── subcmd1.go
│   └── subcmd2.go
├── go.mod
├── go.sum
├── main.go
├── Makefile
└── README.md
```



其中的 **README.md** 文档描述了此模板如何使用。



为了节省篇幅，这里就不多进行解释了，你可以从我的 GitHub 仓库 [gukaifeng/go-cli-template](https://github.com/gukaifeng/go-cli-template) 克隆，并从此模板开始编写一个 `urfave/cli` 命令行程序。



