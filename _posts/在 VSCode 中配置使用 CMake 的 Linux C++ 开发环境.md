


本文默认你已经有在 VSCode 中配置 Linux C/C++ 开发环境的经验。前置微软官方文档 [Gcc on Linux](https://code.visualstudio.com/docs/cpp/config-linux)。我们知道，在配置 C/C++ 的常规方法中，VSCode 会在项目目录中生成一个名为 `.vscode` 的隐藏文件夹，里面放着一些相关的配置 json 文件。其中与 C/C++ 环境开发有关的主要有三个：

* `task.json`：编译器的编译相关配置。例如编译命令、编译参数设置等等。
* `launch.json`：调试相关的配置。例如调试器路径、调试目录配置等。
* `c_cpp_properties.json`：编译器路径和智能提示(Intellisense)设置。例如 C++ 版本、用于智能提示的头文件路径设置等。



而如果使用 CMake 的话，我们就只需要配置 `CMakeLists.txt` 文件（可能是多个）就可以了。



> 本文用于在 VSCode 中配置使用 CMake 的 Linux C/C++ 开发环境的快速入门，关于更深入的内容，建议看：
>
> * [CMake Tools for Visual Studio Code documentation](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/README.md)：微软官方出的在 VSCode 中使用 CMake 的详细文档。
> * [CMake Online Reference Documentation](https://cmake.org/cmake/help/latest/)：CMake 官方文档。



下面开始。



## 1. 安装必要的软件包或扩展





VSCode 扩展可以直接在其内置的扩展商店搜索安装，也可以自己去网站下载安装：

* [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)：微软出品的用于 VSCode 的 C/C++ 扩展。
* [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)：微软出品的用于 VSCode 的 CMake 工具扩展。
* （可选）[CMake](https://marketplace.visualstudio.com/items?itemName=twxs.cmake)：这个我主要是用他的语法高亮，CMake Tools 没有提供 CMakeLists.txt 的语法高亮，看着不太得劲。



另外，请注意，这两个扩展都仅仅是 VSCode 上的辅助工具，所以编译 C/C++，需要你的系统中安装了 `gcc` 以及 `cmake`：

```shell
sudo dnf -y install gcc cmake
```

安装好后，可以通过下面两个命令查看 `gcc` 和 `cmake` 的版本：

```shell
gcc -v
```

```shell
cmake --version
```

这一步比较基础，灵活解决就好，就不做过多解释了。







## 2. 创建 CMake 项目

任意创建一个项目文件夹，我这里叫 "cmakeQuickStart"，然后用 VSCode 打开。

然后我们按快捷键 `ctrl+shift+p`（macOS 上是 `cmd+shift+p` 打开命令栏），输入 `CMake: Quick Start`。



选定后，接下来是一系列的配置提示，按顺序如下：



1. 选择编译器。插件会自动在你的电脑里寻找可用的编译器，我这里选择的是 `gcc 8.5` （全名省略了）。
2. 写入项目名称。我这里就写的 "cmakeQuictStart"。
3. 选择这是一个库(Library)还是一个可执行文件(Executable)。我这里选择的是 Executable，演示比较简单。



这些步骤完成后，会在你的项目目录下生成一个 `build` 目录和一个 `CMakeLists.txt` 文件，  
以及一个打印 "Hello, world!" 的 `main.cpp` 文件。我这里的目录树如下：

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z cmakeQuickStart]$ tree .
.
├── build
│   ├── ......
├── CMakeLists.txt
└── main.cpp

39 directories, 205 files
```



我这里省略了 `build`  目录中的内容，因为我们不需要关注这里，  
`build` 目录是自动生成的，即便我们没有使用 CMake Tools 提供的快速开始，而是直接自己编写一个 CMakeLists.txt，我们也可以在第 4 小节生成该目录。

我们目前只需要关注在项目根目录中的 `CMakeLists.txt`，我们刚刚配置生成的内容如下：

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(cmakeQuickStart VERSION 0.1.0)

include(CTest)
enable_testing()

add_executable(cmakeQuickStart main.cpp)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

这个 `CMakeLists.txt` 的内容比较简单，不过这里先不解释了，也不是本文的重点，本文只关注快速跑起来！



`main.cpp` 的内容如下：

```cpp
#include <iostream>

int main(int, char**) {
    std::cout << "Hello, world!\n";
}
```

\-

另外，第一步的编译器选定以后是可以修改的，有 `ctrl+shift+p` 命令 `CMake: Select a kit`，会弹出下拉菜单，重新选择即可。

也可以通过 VSCode 图形界面点击来实现：



![Cmake: Select a kit](https://gukaifeng.cn/posts/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing_1.png)



如果下拉菜单里没有你想要选择的编译器，也可以通过 `ctrl+shift+p` 命令 `CMake: Edit User-Local CMake Kits` 直接编辑 json 文件来解决，这里就不多说了。

## 3. 选择一个构建变体

执行 `ctrl+shift+p` 命令 `CMake: Select Variant`，VSCode 上方会出现一个下来菜单，有 4 种构建变体可以选择：

* `Debug`：调试模式，禁用优化，显示调试信息。
* `Release`：发布模式，开启优化，禁用调试信息。
* `MinRelSize`：针对大小优化，禁用调试信息。
* `RelWithDebInfo`：针对速度优化，显示调试信息。



这里根据需求选择就可以，本例中选择的是 `Debug`。



这里也可以鼠标操作，点击 VSCode 下方状态栏图标，和输入命令是一样的效果，像下面这样：



![CMake: Select Variant](https://gukaifeng.cn/posts/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing_2.png)

## 4. 执行 Cmake 项目配置

执行 `ctrl+shift+p` 命令 `CMake: Configure`。

该命令会依据我们之前的设定来配置我们的项目，在项目目录生成/更新 `build` 目录。



## 5. 编译运行 "Hello Wrold"

执行 `ctrl+shift+p` 命令 `CMake: Build`。





这里也可以鼠标操作，点击 VSCode 下方状态栏图标，和输入命令是一样的效果：



![CMake: Build](https://gukaifeng.cn/posts/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing_3.png)



这样就编译完成了。



编译后的可执行二进制文件在 `build` 目录下，我们 CMakeLists.txt 有一句话是：

```cmake
add_executable(cmakeQuickStart main.cpp)
```

这句话的意思是，`main.cpp` 源文件可编译出可执行文件，设置可执行文件的名字为 `cmakeQuickStart`。

所以我们可以通过命令行运行该可执行文件：

```shell
$ ./build/cmakeQuickStart 
Hello, world!
```

同样，这个操作也可以通过执行 `ctrl+shift+p` 命令 `CMake: Run Without Debugging` 或快捷键 `shift+F5` 或点击状态栏对应图标来实现：



![CMake: Run Without Debugging](https://gukaifeng.cn/posts/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing_4.png)

## 6. 调试断点

调试其实没有什么特殊的，和往常一样。

例如，我们要打断点的话，只需要在行首前的位置点一下，打个断点。



然后执行 `ctrl+shift+p` 命令 `CMake: Debug` 或直接按快捷键 `ctrl+F5` 即可开始调试，程序会在断点位置中断。



同样，这个操作也可以通过点击状态栏对应图标来实现：



![CMake: Debug](https://gukaifeng.cn/posts/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing/zai-vscode-zhong-pei-zhi-shi-yong-cmake-de-linux-c-kai-fa-huan-jing_5.png)



而后续的其他操作，就都和平常一样了，并非 CMake 专属的，不是本文重点。



本文到这里就结束了，本文只是演示了在 VSCode 使用 CMake 快速开始，能编译运行、打断点调试就 OK 了，  
至于更深入的内容，建议看文章开头处给出的文档链接。

Over！
