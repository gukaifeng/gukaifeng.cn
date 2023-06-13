
包帮助管理大型软件系统，将语义近似的类组织到包中，解决命名冲突问题。

包可以包含类和子包。

## 1. 关键字`package`

package 语句作为 Java 源文件的第一条语句，指明该文件中定义的类所在的包（若缺省该语句，则指定为无名包）。它的格式为：`package 顶层包名.子包名`，用 '.' 来指明包的层次。包常用小写单词，类名首字母通常大写。

## 2. 关键字`import`

关键字`import`用来导入其他包。

import 出现在 package 语句之后，类定义之前，可以有多个 import 语句。

import.lee.* 语句，表示导入 lee 包下所有的类，而 lee 包的子包内的类不会被导入。

import 语句不是必须，可以在类内使用其他类的全名。

<!--more-->

## 3. JDK 中主要的包介绍

1. java.lang：包含一些 java 语言的核心类，如 String、Math、Integer、System 和 Thread，提供常用功能。
2. java.net：包含执行与网络相关的操作的类和接口。
3. java.io：包含能提供多种输入/输出功能的类。
4. java.util：包含一些实用工具类，如定义系统特性、接口的集合框架类、使用与日期日历相关的函数。
5. java.text：包含了一些 java 格式化相关的类。
6. java.sql：包含了 java 进行 JDBC 数据库编程的相关类/接口。
7. java.awt：包含了构成抽象窗口工具集（abstract window toolkits）的多个类，这些类被用来构建和管理应用程序的图形用户界面（GUI）。
8. java.applet：包含 applet 运行所需的一些类。