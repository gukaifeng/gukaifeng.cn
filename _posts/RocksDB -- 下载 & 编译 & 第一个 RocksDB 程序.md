---
title: RocksDB -- 下载 & 编译 & 第一个 RocksDB 程序
date: 2021-06-30
updated: 2021-06-30
categories: [数据库]
tags: [数据库, RocksDB]
toc: true
---




## 1. 下载

RocksDB 可以在 GitHub 上直接 clone 到本地，也可以在 GitHub 中的 Release 页面下载 tar.gz 文件。  

GitHub 页面：https://github.com/facebook/rocksdb 。  
Release 页面：https://github.com/facebook/rocksdb/releases 。

`此博客下载的 RocksDB 版本是 v6.25.3`


## 2. 编译

**\* 请不要用 `make` 或 `make all` 来编译 RocksDB，这样编译出的是 debug 版本，性能会远低于 release 版本。**  
**\* RocksDB 编译不依赖任何第三方库，部分功能所依赖的第三方库是可选的。**

这里使用官方推荐的编译方式 -- 分别编译静态库和动态库。

首先，进入我们 clone 下的或解压 tar.gz 文件出的 RocksDB 目录。
我这里是 `~/bubu/rocksdb-6.25.3`。

<!--more-->

### 2.1. 编译静态库

输入命令 `make static_lib`，将开始编译静态库.  

完成后，会在当前目录下创建一个 `librocksdb.a` 静态库文件。

### 2.2. 编译动态库

输入命令 `make shared_lib`，将开始编译静态库.  

完成后，会在当前目录下创建一个 `librocksdb.so.6.25.3` 动态库文件，  
还有 3 个软链接，均链接到 `librocksdb.so.6.25.3`。  
`librocksdb.so -> librocksdb.so.6.25.3`  
`librocksdb.so.6 -> librocksdb.so.6.25.3`  
`librocksdb.so.6.25 -> librocksdb.so.6.25.3`  



>这里你可能会遇到一个错误：  
>`/usr/bin/ld: 最后的链结失败: 输出不可表示的节`  
>解决方法是在编译命令后加上参数 `-no-pie`。  
>若你想知道为何，查看 [CentOS 中编译动态库出错 “/usr/bin/ld: 最后的链结失败: 输出不可表示的节”](https://gukaifeng.cn/archives/48/) 这篇文章。

*到这里， RocksDB 编译就完成了。*

## 3. 第一个 RocksDB 程序

这里使用 VSCode，**此处假定读者已经知道如何使用 VSCode 在 Linux 下编译运行 C++ 程序**。  

另外，需要注意的是，RocksDB 是嵌入式 KV 数据库，没有类似 MySQL、Redis 那种客户/服务器模式，也就不需要启动 server 或 client。  
RocksDB 是直接在代码中操作数据库的，类似 SQLite，这可能会让一些初次接触嵌入式数据库的人不太习惯（比如我）。

### 3.1. 配置 VSCode

接下来，分别编辑 `c_cpp_properties.json`、`tasks.json`、`launch.json` 文件。

#### 3.1.1 `c_cpp_properties.json`

```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "~/bubu/rocksdb-6.25.3/include/"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "gnu17",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```

上面的代码，基于 VSCode 自动创建的模板。  
在 `includePath` 中添加了 RocksDB 的中的 include 路径 `"~/bubu/rocksdb-6.25.3/include/"`（记得改成你自己的）。  

**\* 这里的 include 路径配置正确与否，与是否能正常编译无关，仅用于代码检查、提示、跳转等编辑时的功能。**

#### 3.1.2 `tasks.json`

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: g++ build active file",
            "command": "/usr/bin/g++",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}",
                "-I", "~/bubu/rocksdb-6.25.3/include/",
                "-L", "~/bubu/rocksdb-6.25.3/",
                "-l", "rocksdb"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "compiler: /usr/bin/g++"
        }
    ]
}
```

上面的代码，基于 VSCode 自动创建的模板。  
在 `args` 中添加了编译 RocksDB 需要的信息（记得改成你自己的）。  

* `"-I", "~/bubu/rocksdb-6.25.3/include/"`: RocksDB 的中的 include 路径。
* `"-L", "~/bubu/rocksdb-6.25.3/"`: 编译好的 RocksDB 静态库 `librocksdb.a` 路径。
* `"-l", "rocksdb"`: RocksDB 的动态库（动态库 `librocksdb.so.6.25.0` 名字中，lib 和 .so 中间的部分就是）。

#### 3.1.3 `launch.json`（可选）

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++ - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: g++ build active file",
            "miDebuggerPath": "/usr/bin/gdb"
        }
    ]
}
```

上面的代码，基于 VSCode 自动创建的模板。  
未作修改，只有当你使用 F5 在 VSCode 中运行程序的时候才用得到。

### 3.2. 编辑代码

在你的 cpp 文件中，拷贝以下内容，我这里是 `main.cpp`。

```cpp
#include <cassert>
#include "rocksdb/db.h"

int main(int argc, char* argv[]) {
  rocksdb::DB* db;
  rocksdb::Options options;
  options.create_if_missing = true;
  rocksdb::Status status =
      rocksdb::DB::Open(options, "./testdb", &db);
  assert(status.ok());

  return 0;
}
```

上面的代码，基于 FaceBook 提供的示例代码，有修改。  
上面的代码通过 `rocksdb::DB::Open(options, "./testdb", &db);` 打开当前目录下的 testdb 数据库。  
`options.create_if_missing = true` 表示如果 testdb 数据库不存在，自动创建，默认是 `false`。  
与之对应，你还可以添加一个 `options.error_if_exists = true`（在 Open 函数前），意思是如果数据库已经存在，就会报错。  
`options.create_if_missing = true` 与 `options.error_if_exists = true` 配合就表示只能是新建一个新的数据库，如果已经存在同名数据库，就会报错。


### 3.3. 编译代码

快捷键 `shift + cmd + B`（我使用的是 macOS，用你自己系统的快捷键），调用我们之前编写好的 `tasks.json`。  
编译成功后，会在当前目录生成一个可执行文件，我这里名为 `main`。

---

这里列出几个你可能会遇到的问题与解决方案：

1\. 关于链接库 `pthread` 的报错，**节选**如下：

```
...
/home/gukaifeng/rocksdb/rocksdb-6.25.3/port/port_posix.cc:71: undefined reference to `pthread_mutexattr_destroy'
...
/usr/include/c++/8/thread:131: undefined reference to `pthread_create'
...
```

**解决方案：**在 `task.json` 中 `tasks` -> `args` 字段中添加一行 `"-l", "pthread"`。

2\. 关于链接库 `dl` 的报错，**节选**如下：

```
...
/home/gukaifeng/rocksdb/rocksdb-6.25.3/env/env_posix.cc:243: undefined reference to `dlopen'
/home/gukaifeng/rocksdb/rocksdb-6.25.3/env/env_posix.cc:281: undefined reference to `dlerror'
...
```

**解决方案：**在 `task.json` 中 `tasks` -> `args` 字段中添加一行 `"-l", "dl"`。

3\. 关于链接库 `z` 的报错，**节选**如下：

```
...
/home/gukaifeng/rocksdb/rocksdb-6.25.3/./util/compression.h:826: undefined reference to `inflateInit2_'
/home/gukaifeng/rocksdb/rocksdb-6.25.3/./util/compression.h:852: undefined reference to `inflate'
...
/home/gukaifeng/rocksdb/rocksdb-6.25.3/./util/compression.h:758: undefined reference to `deflateEnd'
/home/gukaifeng/rocksdb/rocksdb-6.25.3/./util/compression.h:781: undefined reference to `deflateEnd'
...
```

**解决方案：**在 `task.json` 中 `tasks` -> `args` 字段中添加一行 `"-l", "z"`。


### 3.4. 运行

在运行之前，还有一个问题。  
此时我们的编译出的 `main` 可执行文件，是有可能找不到我们 `librocksdb.so.6.25.0` 的。  

如果出现找不到 `librocksdb.so.6.25.0` 的错误，则**在终端中输入 `export LD_LIBRARY_PATH=~/bubu/rocksdb-6.25.3/`，添加我们的 `librocksdb.so.6.25.0` 路径，同样记得换成你自己的。**

如果你之前配置了可选的 `launch.json`，按 F5 是编译并运行。  
也可以像我一样，不使用 F5。我喜欢将编译和运行分开（当然你也可以在 `launch.json` 中配置为只运行不编译，个人喜好而已）。

在终端中当前目录下，输入 `./main` 即可运行我们编译好的程序。  
运行成功的话，不会输出任何内容。  
但是由于我们是第一次运行此程序，并且设置了 RocksDB 选项 `options.create_if_missing = true`，  
所以在程序运行成功后，**会在当前目录下创建一个 testdb 目录，也就是程序新创建的 RocksDB 数据库**。

终端输入 `ll -R testdb/`，可以得到类似以下内容的输出。

```shell
testdb/:
总用量 4240
-rw-r--r-- 1 gukaifeng gukaifeng     0 2021-06-27 02:25:47 000005.log
-rw-r--r-- 1 gukaifeng gukaifeng    16 2021-06-27 02:25:47 CURRENT
-rw-r--r-- 1 gukaifeng gukaifeng    37 2021-06-27 02:25:47 IDENTITY
-rw-r--r-- 1 gukaifeng gukaifeng     0 2021-06-27 02:25:47 LOCK
-rw-r--r-- 1 gukaifeng gukaifeng 18964 2021-06-27 02:25:47 LOG
-rw-r--r-- 1 gukaifeng gukaifeng    57 2021-06-27 02:25:47 MANIFEST-000004
-rw-r--r-- 1 gukaifeng gukaifeng  6206 2021-06-27 02:25:47 OPTIONS-000007
```

### 3.5 更多

#### 3.5.1 关于 `rocksdb::Status`

你可能注意到了，`rocksdb::DB::Open(options, "./testdb", &db)` 的返回值类型是 `rocksdb::Status`。  
`rocksdb::Status` 是 RocksDB 大部分函数的返回值类型.  
假设返回值是 `rocksdb::Status s`，那么，如果函数成功结束，`s.ok()` 会返回 `true`，否则，你可以查看 `s.ToString()` 得到错误信息。  

[在这里查看更多关于 `rocksdb::Status` 的内容。]() 

#### 3.5.2 关闭数据库

上文说过，RocksDB 是一个嵌入式 KV 数据库，是直接在代码中操作数据库的。  
所以关闭数据库也非常简单，只要删除我们创建的那个 `DB*` 类型的指针 `db` 即可（删除这个指针不会删除我们的数据库）。

```cpp
/* open the db as described above */
/* do something with db */
delete db;
```


#### 3.5.3 读写数据库

RocksDB 提供了 Put、Delete 和 Get 方法来修改/查询数据库。  
下面的示例使用了这 3 个方法，实现将 key1 的 value 转移到 key2 上。  

```cpp
std::string value;
rocksdb::Status s = db->Get(rocksdb::ReadOptions(), key1, &value);
if (s.ok()) s = db->Put(rocksdb::WriteOptions(), key2, value);
if (s.ok()) s = db->Delete(rocksdb::WriteOptions(), key1);
```

**此示例仅用于简单说明，不可直接拷贝运行。**  
[在这里查看更多关于 RocksDB 读写操作的内容。]()



## 4. 参考资料

https://github.com/facebook/rocksdb/blob/master/INSTALL.md  
https://rocksdb.org/docs/getting-started.html