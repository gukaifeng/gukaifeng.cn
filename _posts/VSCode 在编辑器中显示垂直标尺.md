




我们知道一行代码不宜过长，不同的公司、团队、项目通常有不同的规范。



最常见的行最大字符数的规范比如：

* 每行字符数最大不得超过 80。
* 每行字符数最大不得超过 120。
* 每行建议不超过 80 字符，最大不得超过 120 字符。
* ......





VSCode 支持设定垂直标尺，即在编辑器中，在指定字符数的位置给你画一条竖线。



例如下面这样，就是分别在 80 字符处和 120 字符处分别花了一条竖线，这样方便我们在写代码的时候控制每行字符数。



![在 80 和 120 字符处显示垂直标尺](https://gukaifeng.cn/posts/vscode-zai-bian-ji-qi-zhong-xian-shi-chui-zhi-biao-chi/vscode-zai-bian-ji-qi-zhong-xian-shi-chui-zhi-biao-chi_1.png)





设置方法也很简单，我们依次点击 File -> Perferences -> Setting：



![File -> Perferences -> Setting](https://gukaifeng.cn/posts/vscode-zai-bian-ji-qi-zhong-xian-shi-chui-zhi-biao-chi/vscode-zai-bian-ji-qi-zhong-xian-shi-chui-zhi-biao-chi_2.png)



然后在上面的搜索栏里搜索 "Editor.rulers"：



![Editor.rulers](https://gukaifeng.cn/posts/vscode-zai-bian-ji-qi-zhong-xian-shi-chui-zhi-biao-chi/vscode-zai-bian-ji-qi-zhong-xian-shi-chui-zhi-biao-chi_3.png)

点击 "Edit in setting.json"，在里面修改 `editor.rulers` 字段，像下面这样就是在 80 和 120 字符处显示垂直标尺：



![settings.json](https://gukaifeng.cn/posts/vscode-zai-bian-ji-qi-zhong-xian-shi-chui-zhi-biao-chi/vscode-zai-bian-ji-qi-zhong-xian-shi-chui-zhi-biao-chi_4.png)



然后保存就 OK 了。
