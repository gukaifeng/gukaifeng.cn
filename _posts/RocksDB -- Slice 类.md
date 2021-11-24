---
title: RocksDB -- Slice 类
date: 2021-11-24 17:21:54
updated: 2021-11-24 19:31:54
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



RocksDB Slice 类定义在 `slice.h` 头文件中。[点此查看 `slice.h`](https://github.com/facebook/rocksdb/blob/main/include/rocksdb/slice.h)。



## 1. Slice 类的成员

Slice 是 RocksDB 中一个非常简单又很常用的类，其中只有两个成员：

```cpp
// private: make these public for rocksdbjni access
const char* data_;
size_t size_;
```

* `data_`: 这是一个指向外部存储的指针；
* `size_`: 记录 `data_` 指向的存储空间大小。

在目前 RocksDB 的版本中，这两个成员默认都是 `public` 的，  
可以注意到，源码中的 `private` 被注释掉了，原因也一并写在了注释中，当然我们暂且不关注这个。



<!--more-->



## 2. Slice 类的方法

Slice 类中的大多方法都直接实现在 `slice.h` 头文件中，部分实现在 `slice.cc`源文件中。[点此查看 `slice.cc`](https://github.com/facebook/rocksdb/blob/main/util/slice.cc)。

下面按照 `slice.h` 中的顺序，依次介绍其中方法。



### 2.1. 构造函数



构造函数有 **6** 个重载，下面按代码中的顺序依次说。

1\. 创建一个空的 Slice

```cpp
Slice() : data_(""), size_(0) {}
```

这个看起来很简单，就是把 `data_` 初始化为空，`size_` 初始化为 0。

2\. 从 char 字符串创建 Slice

```cpp
Slice(const char* d, size_t n) : data_(d), size_(n) {}
```

* `d`: char 字符串的起始地址；
* `n`: char 字符串的长度。

即把 `d[0,n-1]` 的内容初始化到 Slice。

3\. 从 std::string 字符串创建 Slice

```cpp
Slice(const std::string& s) : data_(s.data()), size_(s.size()) {}
```

* `s`: std::string 字符串。

这个和上一个没啥区别。

4\. 从 std::string_view 字符串创建 Slice

```cpp
#ifdef __cpp_lib_string_view
// Create a slice that refers to the same contents as "sv"
/* implicit */
Slice(std::string_view sv) : data_(sv.data()), size_(sv.size()) {}
#endif
```

std::string_view 是 C++17 中的新特性，就是一个只读的 std::string。

和上一个没啥区别。

5\. 从 char 字符串创建 Slice（自动计算长度）

```cpp
Slice(const char* s) : data_(s) { size_ = (s == nullptr) ? 0 : strlen(s); }
```

这个和上面第 2 个构造函数的唯一区别就是，无需你指定 char 字符串的长度，使用 `strlen()` 函数来计算。

6\. 从 SliceParts 创建一个 Slice

```cpp
Slice(const struct SliceParts& parts, std::string* buf);
```

这个是唯一一个相对复杂的构造函数，其定义在 `slice.cc` 文件中，如下：

```cpp
Slice::Slice(const SliceParts& parts, std::string* buf) {
  size_t length = 0;
  for (int i = 0; i < parts.num_parts; ++i) {
    length += parts.parts[i].size();
  }
  buf->reserve(length);

  for (int i = 0; i < parts.num_parts; ++i) {
    buf->append(parts.parts[i].data(), parts.parts[i].size());
  }
  data_ = buf->data();
  size_ = buf->size();
}
```

-

其中 SliceParts 就是一个 Slice 数组，里面存了很多个 Slice，其定义也在 `slice.h` 中，如下：

```cpp
struct SliceParts {
  SliceParts(const Slice* _parts, int _num_parts)
      : parts(_parts), num_parts(_num_parts) {}
  SliceParts() : parts(nullptr), num_parts(0) {}

  const Slice* parts;
  int num_parts;
};
```

* `_part`: 指向 Slice 数组的指针；
* `_num_parts`: Slice 数组的长度，即数组中 Slice 对象的个数。

-

现在再看上面的构造函数，就很明了了，这个构造函数，就是把 SliceParts 中的所有 Slice 拼接起来，搞了一个大的 Slice。

**要注意，代码中的 `data_ = buf->data();` 执行的是浅拷贝！所以，一旦 `buf` 失效，这个 Slice 也会失效！**





### 2.2. 其他方法

1\. 获取数据指针

```cpp
const char* data() const { return data_; }
```

2\. 获取数据长度

```cpp
size_t size() const { return size_; }
```

3\. 判断 Slice 是否为空

```cpp
bool empty() const { return size_ == 0; }
```

4\. 重载 `[]` 运算符

```cpp
char operator[](size_t n) const {
  assert(n < size());
  return data_[n];
}
```

使 Slice 可以像普通数组那样通过 `[]` 运算符使用索引访问元素。

5\. 清空 Slice

```cpp
void clear() {
  data_ = "";
  size_ = 0;
}
```

6\. 移除 Slice 中数据的前 n 个字符

```cpp
void remove_prefix(size_t n) {
  assert(n <= size());
  data_ += n;
  size_ -= n;
}
```

7\. 移除 Slice 中数据的后 n 个字符

```cpp
void remove_suffix(size_t n) {
  assert(n <= size());
  size_ -= n;
}
```

8\. 获取 Slice 中数据的 std::string 类型的拷贝

```cpp
std::string ToString(bool hex = false) const;
```



* `hex`: 当 hex 为 true 时，返回的 std::string 将进行 16 进制编码。

这里要注意的是，如果 hex 为 true，那么返回的 std::string 的长度将是 Slice 中数据长度的两倍。原因是原来是一个字符 1 个字节（8 位），用 16 进制编码，是将原来的数据每 4 位编程一个 16 进制值，所以返回的 std::string 长度将是原来数据长度的两倍。

这个函数的实现较为复杂，而且不是很重要，这里不列实现了，有兴趣可以自己去看源码。

9\. 获取 Slice 中数据的 std::string_view 类型的拷贝

```cpp
#ifdef __cpp_lib_string_view
  // Return a string_view that references the same data as this slice.
  std::string_view ToStringView() const {
    return std::string_view(data_, size_);
  }
#endif
```

这个比较简单了，因为没有像上一个方法那样提供 hex 参数。



10\. 判断一个 std::string 字符串是不是有效的 16 进制串

```cpp
bool DecodeHex(std::string* result) const;
```

这个函数检查 result 是不是一个有效的 16 进制串，即其中只含有 0-9A-F 字符，大小写均可。

如果是一个有效的 16 进制串，返回 true，否则返回 false。

其实现在 `slice.cc` 中，和之前的 16 进制编码实现一样，这个不是很重要，这里就不说了，有兴趣的可以自己去看看源码。



11\. 比较两个 Slice 中的数据

```cpp
int compare(const Slice& b) const;
```

其实现和声明都在 `slice.h` 头文件中，但是没写在一起，其实现如下：

```cpp
inline int Slice::compare(const Slice& b) const {
  assert(data_ != nullptr && b.data_ != nullptr);
  const size_t min_len = (size_ < b.size_) ? size_ : b.size_;
  int r = memcmp(data_, b.data_, min_len);
  if (r == 0) {
    if (size_ < b.size_)
      r = -1;
    else if (size_ > b.size_)
      r = +1;
  }
  return r;
}
```

从代码可以看到，其实就是两个 Slice 中数据的字典序比较，即：

* 返回值小于 0 如果 "\*this" <  "b"；
* 返回值等于 0 如果 "\*this" == "b"；
* 返回值大于 0 如果 "\*this" >  "b"；
* 如果一个 Slice a 中数据是另一个 Slice b 中数据的前缀，则 a < b，返回值小于 0。



12\. 判断一个 Slice 中数据的开头是否和另一个 Slice 中的数据一样

```cpp
bool starts_with(const Slice& x) const {
  return ((size_ >= x.size_) && (memcmp(data_, x.data_, x.size_) == 0));
}
```





13\. 判断一个 Slice 中数据的结尾是否和另一个 Slice 中的数据一样

```cpp
bool ends_with(const Slice& x) const {
  return ((size_ >= x.size_) &&
          (memcmp(data_ + size_ - x.size_, x.data_, x.size_) == 0));
}
```





14\. 比较两个 Slice 并返回第一个不同字符的位置

```cpp
size_t difference_offset(const Slice& b) const;
```

其实现和声明都在 `slice.h` 头文件中，但是没写在一起，其实现如下：

```cpp
inline size_t Slice::difference_offset(const Slice& b) const {
  size_t off = 0;
  const size_t len = (size_ < b.size_) ? size_ : b.size_;
  for (; off < len; off++) {
    if (data_[off] != b.data_[off]) break;
  }
  return off;
}
```

通过代码可以看到，`difference_offset()` 方法比较两个 Slice 中的数据。

如果两个 Slice 中的数据有不同，就返回不同位置的索引；如果两个 Slice 中的数据完全相同，则返回数据的长度。





## 3. 比较 Slice 的方法

`slice.h` 中还定义了两个运算符重载（在头文件最后的位置），用以比较两个 Slice。

```cpp
inline bool operator==(const Slice& x, const Slice& y) {
  return ((x.size() == y.size()) &&
          (memcmp(x.data(), y.data(), x.size()) == 0));
}
```

```cpp
inline bool operator!=(const Slice& x, const Slice& y) { return !(x == y); }
```



上面的代码写的还是很巧妙的，实际上主要重载的是 `==` 运算符，然后 `!=` 的重载用了 `==` 重载的结果。

这两个运算符重载后，两个 Slice 比较的是其中的具体数据。
