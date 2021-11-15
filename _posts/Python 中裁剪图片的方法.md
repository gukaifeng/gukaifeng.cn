---
title: Python 中裁剪图片的方法
date: 2021-10-19 17:53:57
updated: 2021-10-19 17:53:57
categories: [技术杂谈]
tags: [Python, OpenCV]
toc: true
---

之前用代码生成的图片，两侧有较宽的空白，试着用 python 裁剪掉白边。

Python 中裁剪图片有多种方法的，这里介绍一种，使用 `OpenCV` 库提供的函数来裁剪。

下面从示例代码看方法：

<!--more-->

```python
import cv2  # 如果你没有这个库的话，安装的时候包名叫 opencv-python

file = "./example.png"  # 待裁剪的图片路径
img = cv2.imread(file)  # 先读取待裁剪的图片
new_img = img[0:500, 400:3650]  # 这里的 0:500 指的是纵向，从上到下的取第 0(含) - 500(不含) 的像素，其余的裁减掉；
                                # 这里的 400:3650 指的是横向，从左到右取第 400(含) - 3650(不含) 个像素，其余的裁减掉。
# cv2.imwrite(file, new_img)  # 写入文件，注意，危险，这个写法会覆盖掉原图片！
new_img_path = "./new_example.png"
cv2.imwrite(new_img_path, new_img)  # 把裁剪后的图片写入 new_img_path
```



**\* 扩展**

* 图片的 `shape` 属性可以图片尺寸，例如 `print(img.shape)` 就可以打印出图片的尺寸信息了。
* 对于裁剪图片的像素范围，可以简写。例如 `img[: 20:]`  表示上下不裁剪，左边裁剪 20 个像素，右边不裁剪。