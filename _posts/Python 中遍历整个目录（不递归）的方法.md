
一般来说，遍历一个目录下的全部文件（目录），有两种常见场景。

1. 只需要遍历这个目录下的文件（目录），不继续深入遍历其中的子目录；
2. 要遍历这个目录下的全部文件（目录），包含子目录（递归）。



<!--more-->

**这里要说的是第 1 个场景。**

Python 中的 `os` 库提供了一个函数 `listdir()` 可以用来遍历一个目录下的全部内容，**不深入子目录**。

下面从示例代码中看 `os.listdir()` 的用法：

```python
import os

dir_path = "~/directory/"  # 待遍历目录，这里在最后加上 `/`，以后拼全路径的时候就可以不加了。
for dir_or_file in os.listdir(dir_path):  # 这里起名 dir_or_file 是因为 os.listdir() 返回的不仅有文件名，还有目录名
  file = dir_path + dir_or_file  # 注意这里，os.listdir() 得到的只有文件名，如果在后面要对文件操作，得补全路径
```

最后这个 `file` 就是目标目录下的文件名（或目录名）了（补全了路径的）。



**\* 扩展**

如果你只需要对目标目录下的文件操作，或者只对目标目录下的子目录操作，这里给出两个函数来判断 `file` 是文件还是目录。

1. `os.path.isdir(file)`：如果 `file` 是目录，返回 `true`，否则返回 `false`。
2. `os.path.isfile(file)`：如果 `file` 是文件，返回 `true`，否则返回 `false`。