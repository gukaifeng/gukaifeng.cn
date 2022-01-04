---
title: Python 压缩图片大小（非尺寸）
date: 2022-01-05 00:27:15
updated: 2022-01-05 00:27:15
categories: [技术杂谈]
tags: [Python]
---

这篇文章要说的是，在不改变图片尺寸的前提下，缩小图片的尺寸（肯定会降低图片质量）。

如果你同时想要改变图片的尺寸，看 [Python 缩放图片（保持宽高比）](https://www.gukaifeng.cn/posts/python-suo-fang-tu-pian-bao-chi-kuan-gao-bi/)。



---

思路：

1. 设定一个期望大小 target_size；
2. 牺牲部分质量压缩图片，得到新图片大小 new_size；
3. 比较 new_size 和 target_size；
4. 如果 new_size <= target_size，结束。否则重复 2 - 4。

同时我们还需要设定一个最大压缩次数，或者等价的概念，  
因为有可能我们的图片无法压缩至目标大小。

---

代码：同样的只演示过程，代码尽量精简。

```python
import os
from PIL import Image

img_path = "path/to/your/image"  # 要压缩的图片路径
new_img_path = "path/to/your/new/image"  # 压缩好的图片保存路径

target_size = 150 * 1024  # 单位：字节(B)，即目标大小 150 KB
quality = 80  # 保留质量，1-100(默认 75)，1 最差，100 最好，不建议过高且 100 会禁用一些压缩算法
step = 10  # 降低质量的步长

current_img = Image.open(img_path)  # 初始化当前图片为原图
current_size = os.path.getsize(img_path)  # 初始化当前大小为原图片大小

while current_size > target_size and quality > 0:
  # current_img = current_img.convert(mode="P", palette=Image.ADAPTIVE)  # 可选，转为 8 位彩色图像，会更小，但图像质量也会更差
  current_img.save(new_img_path, quality=quality,
                   optimize=True)  # 压缩新图片到 new_img_path
  current_img = Image.open(new_img_path)  # 更新 current_img 为压缩后的新图片
  # 更新 current_size 为压缩后的新图片大小
  current_size = os.path.getsize(new_img_path)
  quality -= step
```

代码中 14 行的 while 循环表示，当前大小小于目标大小时，或 quality 小于等于 0 时跳出循环，这是因为有些图片可能无法压缩到我们的目标大小，所以尝试几次后结束。

代码中 15 行注释的代码，表示将模式改为 "P"。

1. 模式 "P" 为 8 位彩色图像，其每个像素用 8 个 bit 表示，其对应的彩色值是按照调色板查询出来的。
2. 模式 "RGBA" 为 32 位彩色图像，其每个像素用 32 个 bit 表示，其中 24 bit 表示红色、绿色和蓝色三个通道，另外 8 bit 表示 alpha 通道，即透明通道.
