
在我用 typecho 写博客之前，曾用过一段时间的 hexo，hexo 在用命令创建 markdown 文件的时候，可以使用模板，然后创建出来的 markdown 就会带了一些模板中预设的内容，比如头部说明，并且 hexo 使用头部说明来辅助解析 markdown。

<!--more-->

在 typecho 这里是不需要这个头部内容来辅助解析 markdown 的，但是按很多人写 typecho 的习惯，一般是先在本地写，写完拷贝到 typecho 浏览器后台的编辑器里，设定一下发布时间、类别、标签等信息再发布，**并且本地也留一份备份**。

问题就在于，我认为本地的这个备份中如果包含头部信息，以后看起来会比较方便，拷贝到 typecho 编辑器里的时候，略过这个头部就好了。

之前每次都是手动在每个 markdown 文档中添加头部，着实有点麻烦，于是就想着写个简单的小程序，稍微减轻一下这个工作量。

**适用系统：Unix 系**

#### 1. 效果预览

先看看效果吧，我这里现在编译好的程序文件名为 `createPost`

在命令行输入

```shell
./createPost "写一个小程序在新建 md 文件时添加头部信息"
```

然后就会在这个小程序的同级目录下创建一个名为 “写一个小程序在新建 md 文件时添加头部信息.md” 的 markdown 文件，打开这个文件，就可以看到下面的内容：

```markdown
---
title: 写一个小程序在新建 md 文件时添加头部信息
date: 2021-09-07 00:05:23
categories: []
tags: []
---
```

* `title`: 顾名思义，文章名，也就小程序执行时后面的那个参数，不用加 `.md` 后缀；
* `date`：自动填入的，你创建这个文档的本地时间；
* `categories`: 默认设置为空了，方便后面往里填；
* `tags`: 同上。

#### 2. C/C++ 代码

```cpp
#include <iostream>
#include <ctime>

int main(int argc, char *argv[])
{

    if (argc != 2)
    {
        printf("Usage: %s [postname]\n", argv[0]);
    }

    char post[300] = {0};
    char str[5][20] = {"---", "title", "date", "categories", "tags"};
    size_t written_len = 0;

    // write "---"
    sprintf(post + written_len, "%s\n", str[0]);
    written_len += strlen(str[0]) + 1;

    // write "title: xxxxxx"
    sprintf(post + written_len, "%s: %s\n", str[1], argv[1]);
    written_len += strlen(str[1]) + 2 + strlen(argv[1]) + 1;

    // write "date: xxxxxx"
    time_t rawtime;
    struct tm *timeinfo;

    time(&rawtime);
    timeinfo = localtime(&rawtime);

    char buffer[80];
    strftime(buffer, 80, "%Y-%m-%d% %H:%M:%S", timeinfo);

    sprintf(post + written_len, "%s: %s\n", str[2], buffer);
    written_len += strlen(str[2]) + 2 + strlen(buffer) + 1;

    // write "categories: []"
    sprintf(post + written_len, "%s: []\n", str[3]);
    written_len += strlen(str[3]) + 4 + 1;

    // write "tags: []"
    sprintf(post + written_len, "%s: []\n", str[4]);
    written_len += strlen(str[4]) + 4 + 1;

    // write "---"
    sprintf(post + written_len, "%s\n", str[0]);
    written_len += strlen(str[0]) + 1;

    // write into file
    char path[200] = {0};
    sprintf(path, "./%s.md", argv[1]);

    FILE *post_fname = fopen(path, "wb");
    fwrite(post, 1, written_len, post_fname);
    fflush(post_fname);
    fclose(post_fname);

    return 0;
}
```

#### 3. 可执行文件下载

如果你比较懒，不想自己去编译，也不需要改代码，只想要我写好的这个可执行文件的话，点击下方链接即可下载。

[createPost](https://gukaifeng.cn/downloads/createPost)