---
title: Python 缩放图片（保持宽高比）
date: 2022-01-04 22:50:40
updated: 2022-01-04 22:50:40
mathjax: true
categories: [技术杂谈]
tags: [Python]
---





一个最简单的方法就是直接 resize()，唯一要额外做的就是计算新的宽（或高）。

---

我们记原图宽和高分别为 $width$ 和 $height$，新图的宽和高分别为 $new\_width$ 和 $new\_height$，那么有如下等式：
$$
\frac{width}{height} = \frac{new\_width}{new\_height}
$$
即有
$$
new\_width = width \times new\_height \div height \\
new\_height = height \times new\_width \div width
$$

---

下面，我们以固定宽，计算高的方式来举一个例子：

> 我写这篇博客的时候，是想把一些大尺寸图片，转换为宽固定为 2560 的图片，保持宽高比，所以需要计算的是新的高 new_height。

下面看代码：

这里使用 Pillow 中的 Image 实现。

```python
from PIL import Image

img_path = "path/to/your/image"  # 要缩放的图片路径
new_img_path = "path/to/your/new/image"  # 缩放好的图片保存路径

img = Image.open(img_path)  # 读取原图
width, height = img.size[0], img.size[1]  # 读取原图的宽和高 .[0] 是宽，.[1] 是高
new_width  = 2560  # 固定新宽
new_height = int(new_width * height / width)  # 计算新高
new_img = img.resize((new_width, new_height), Image.ANTIALIAS)  # 缩放
new_img.save(new_img_path)  # 保存缩放后的图片
```



完结，简单！

---

上面这种方式，是无损的，如果你想要的就是无损的缩放，那么 ok。

但是如果你想要图片小一点，就需要压缩一下图片，看 [Python 压缩图片大小（非尺寸）]()。
