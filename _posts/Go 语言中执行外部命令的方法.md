---
title: Go 语言中执行外部命令的方法
date: 2022-01-10 16:59:11
updated: 2022-01-10 16:59:11
categories:
- 编程语言基础
- Golang
tags: [Golang]
---



Go 语言中执行外部命令主要的方法是使用包 `os/exec`。

此包的详细文档见 [exec package - os/exec - pkg.go.dev](https://pkg.go.dev/os/exec)，这里只介绍几种常用操作。



---

执行命令也分几种情况：

1. 仅执行命令；
2. 执行命令，获取结果，不区分 stdout 和 stderr；
3. 执行命令，获取结果，区分 stdout 和 stderr。

另外，默认的命令执行是在 go 进程当前的目录下执行的，我们可能还需要指定命令执行目录。

下面我们逐个说。



<!--more-->

## 1. 仅执行命令

执行命令，首先要拼接一下命令和参数，然后运行命令。

* 拼接命令与参数使用 `exec.Command()`，其会返回一个 `*Cmd`；

    ```go
    func Command(name string, arg ...string) *Cmd
    ```

    

* 执行命令使用 `*Cmd` 中的 `Run()` 方法，Run() 返回的只有 error。

    ```go
    func (c *Cmd) Run() error
    ```

    

我们直接看代码：

```go
package main

import (
	"log"
	"os/exec"
)

func ExecCommand(name string, args ...string) {
	cmd := exec.Command(name, args...) // 拼接参数与命令
	if err := cmd.Run(); err != nil {  // 执行命令，若命令出错则打印错误到 stderr
		log.Println(err)
	}
}

func main() {
	ExecCommand("ls", "-l")
}
```

执行代码，没有任何输出。

上面的代码中，我们执行了命令 `ls -l`，但是没有得到任何东西。





## 2. 获取结果



### 2.1. 不区分 stdout 和 stderr

要组合 stdout 和 stderr 输出，，`Cmd` 中有方法：

```go
func (c *Cmd) CombinedOutput() ([]byte, error)
```

用这个方法来执行命令（即这个方法是已有 `Run()` 方法的作用的，无需再执行 `Run()`）。

我们修改上述代码：

```go
package main

import (
	"fmt"
	"log"
	"os/exec"
)

func ExecCommand(name string, args ...string) {
	cmd := exec.Command(name, args...) // 拼接参数与命令

	var output []byte
	var err error

	if output, err = cmd.CombinedOutput(); err != nil {
		log.Println(err)
	}
	fmt.Print(string(output)) // output 是 []byte 类型，这里最好转换成 string
}

func main() {
	ExecCommand("ls", "-l")
}
```

我们得到了 `ls -l` 这条命令的输出：

```
total 4
-rw-rw-r-- 1 gukaifeng gukaifeng 401 Jan 10 18:21 main.go
```





### 2.2. 区分 stdout 和 stderr



区分 stdout 和 stderr，要先给 `cmd` 中的成员指定一个输出 buffer，然后执行 `Run()` 就可以。

```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"os/exec"
)

func ExecCommand(name string, args ...string) {
	cmd := exec.Command(name, args...) // 拼接参数与命令

	var stdout bytes.Buffer
	var stderr bytes.Buffer
	var err error

	cmd.Stdout = &stdout
	cmd.Stderr = &stderr

	if err = cmd.Run(); err != nil {
		log.Println(err)
	}
  
	fmt.Print(stdout.String())
	fmt.Print(stderr.String())
}

func main() {
	ExecCommand("ls", "-l")
}
```



## 3. 指定代码执行目录

指定代码执行的目录也很简单，修改 `cmd.Dir` 即可。

`cmd.Dir` 是一个 string 类型的成员，既可以是相对路径，也可以是绝对路径。

如果 `cmd.Dir` 是空字符串，则 `Run()` 在调用进程的当前目录中运行命令。

我们修改 2.2 中的代码，使其在 `/` 目录下执行我们的 `ls -l` 命令：

```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"os/exec"
)

func ExecCommand(dir string, name string, args ...string) {
	cmd := exec.Command(name, args...) // 拼接参数与命令

	var stdout bytes.Buffer
	var stderr bytes.Buffer
	var err error

	cmd.Stdout = &stdout
	cmd.Stderr = &stderr

	cmd.Dir = dir

	if err = cmd.Run(); err != nil {
		log.Println(err)
	}

	fmt.Print(stdout.String())
	fmt.Print(stderr.String())
}

func main() {
	ExecCommand("/", "ls", "-l")
}
```

我们修改了函数 `ExecCommand()` 的参数，使其在第一个参数中指定代码的执行目录。

得到以下输出：

```
total 24
lrwxrwxrwx.   1 root root    7 Nov  3  2020 bin -> usr/bin
dr-xr-xr-x.   5 root root 4096 Sep 22 10:52 boot
drwxr-xr-x   19 root root 2880 Dec  7 18:05 dev
drwxr-xr-x. 100 root root 8192 Dec 13 14:59 etc
drwxr-xr-x.   3 root root   23 Nov  4 23:51 home
lrwxrwxrwx.   1 root root    7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root    9 Nov  3  2020 lib64 -> usr/lib64
drwxr-xr-x.   2 root root    6 Nov  3  2020 media
drwxr-xr-x.   2 root root    6 Nov  3  2020 mnt
drwxr-xr-x.   2 root root    6 Nov  3  2020 opt
dr-xr-xr-x  134 root root    0 Dec  7 18:05 proc
dr-xr-x---.   6 root root  205 Dec 13 14:59 root
drwxr-xr-x   30 root root  880 Dec  7 18:05 run
lrwxrwxrwx.   1 root root    8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x.   2 root root    6 Nov  3  2020 srv
dr-xr-xr-x   13 root root    0 Dec  8 02:05 sys
drwxrwxrwt.  10 root root 4096 Jan 10 18:58 tmp
drwxr-xr-x.  12 root root  144 Sep 22 10:43 usr
drwxr-xr-x.  21 root root 4096 Sep 22 02:48 var
```

