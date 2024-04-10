

## 1. 准备环境



在开始前，我们需要系统内有依赖如下：

* [Go 1.18+](https://go.dev/dl/)
* [NPM (Node 15+)](https://nodejs.org/)
* Wails



前两个点击链接到官网自行下载，然后安装下 Wails：

```powershell
go install github.com/wailsapp/wails/v2/cmd/wails@latest
```



最后执行下 Wails 官方提供的检查命令：

```powershell
wails doctor
```

最后输出有输出 " SUCCESS  Your system is ready for Wails development!" 即为环境已经符合要求了。





## 2. 创建项目



Wails 项目的目录结构相对复杂，所以官方提供了生成初始项目的命令。

我这里选择前端的技术栈为 Vue + TypeScript：

```powershell
wails init -n myproject -t vue-ts
```

更多前端技术栈见 https://wails.io/zh-Hans/docs/gettingstarted/firstproject 。

目录结构如下（不含 `README.md` 和 `.gitignore`）：

```powershell
.
├── build/			# 项目构建目录
│   ├── appicon.png	# 应用程序图标
│   ├── darwin/		# Mac 特定的项目文件
│   └── windows/	# Windows 特定的项目文件
├── frontend/		# 前端项目文件
├── go.mod			# Go module 文件
├── go.sum			# Go module 校验文件
├── main.go			# 主应用
└── wails.json		# 项目配置
```



`go.mod` 文件中第一行中的 "changeme" 换成你自己的项目名字：

```
module changeme
```



然后就可以开始开发了。



\-



我生成的 `go.mod` 文件里有一行这个：

```
toolchain go1.22.2
```

但是会报错 "unknown directive: toolchain"，这行没啥用，删了就 OK。





## 3. 调试项目



执行：

```shell
wails dev
```

然后 Wails 会做几件事：

* 启动一个桌面窗口运行我们的程序。
* 启动一个本地服务 http://localhost:34115/ 在浏览器中预览我们的项目。



Wails 使用了 Vite 前端工具，当我们的代码修改时，浏览器中的项目预览会立即生效变化，桌面窗口会自动重启以应用变化。



![Wails 桌面窗口](https://gukaifeng.cn/posts/windows-shi-yong-go-wails-gou-jian-di-yi-ge-zhuo-mian-cheng-xu/windows-shi-yong-go-wails-gou-jian-di-yi-ge-zhuo-mian-cheng-xu_1.png)

## 4. 编译项目



执行：

```shell
wails build
```

这会在 `build/bin/` 目录下生成一个可执行文件。例如 Windows 系统中是 `wails.exe`，这个名字在 `wails.json` 中的 `outputfilename` 字段配置。