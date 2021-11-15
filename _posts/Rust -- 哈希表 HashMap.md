---
title: Rust -- 哈希表 HashMap
date: 2021-10-27 23:07:53
updated: 2021-10-27 23:07:53
categories: [编程语言基础]
tags: [Rust,数据结构]
toc: true
---



哈希表非常常用，这篇文章不介绍什么是哈希表，只说下 Rust 标准库中提供的哈希表 HashMap。



Rust 中 HashMap 的声明如下：

```rust
pub struct HashMap<K, V, S = RandomState> 
{ /* fields omitted */ }
```



使用哈希表 HashMap 需要先 `use std::collections::HashMap;`。

这篇文章只会介绍几个基础用法，用以入门，详细内容请看官方文档 [HashMap in std::collections::hash_map - Rust](https://doc.rust-lang.org/stable/std/collections/hash_map/struct.HashMap.html)。

<!--more-->

## 1. 新建一个 HashMap



新建 HashMap 要使用 `HashMap::new()` 函数。

下面看代码：

```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let mut scores0 = HashMap::new();  // 新建一个空的 HashMap，类型由 Rust 根据下面的插入操作推断，注意如果在编译前没有插入操作，将编译失败，Rust 必须在编译前知道其元素的类型。
  
    scores0.insert(String::from("Blue"), 10);
    scores0.insert(String::from("Yellow"), 50);
  
    let mut scores1: HashMap<String, u8> = HashMap::new();  // 新建一个空的 HashMap，key 的类型为 String，值的类型为 u8。
  
}
```





## 2. 更新 HashMap





### 2.1. 插入键值对

插入键值对使用 `insert()` 函数：

```rust
pub fn insert(&mut self, k: K, v: V) -> Option<V>
```

`insert()` 将键值对插入 HashMap。如果 HashMap 没有待插入的键，就插入这个键值对，然后返回 `None`；若 HashMap 中已经存在了这个键，则更新其值，并返回旧的值（Option<T> 类型的值）。

下面看一个代码示例：

```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    assert_eq!(map.insert(37, "a"), None);
    assert_eq!(map.is_empty(), false);

    map.insert(37, "b");
    assert_eq!(map.insert(37, "c"), Some("b"));
    assert_eq!(map[&37], "c");
}
```





### 2.2. 根据旧值更新一个值



在哈希表中，根据旧值更新一个值是一个很常用的操作。

我们判断一个键是否在哈希表中，如果在就对其值做某些操作，如果不在就插入一个键值对。

我们先看一段代码：

```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map);
}
```

输出

```
{"world": 2, "hello": 1, "wonderful": 1}
```



上面的代码中用两个你可能没见过的函数：

一个是 `text.split_whitespace()`，顾名思义，这个函数把字符串 `text` 的内容按空格划分开了，这里不详细解释了，这不是这里的重点。另一个是 `map.entry(word).or_insert(0);`。

---



下面先介绍一下 HashMap 的  `entry()` 函数。

```rust
pub fn entry(&mut self, key: K) -> Entry<'_, K, V>
```

这个函数获取给定 key 在 HashMap 中的对应条目，你可以对这个条目原地操作，该条目可以是空的，也可以是已有的。

关于这个 `Entry` 类型，详细请查看文档 [Entry in std::collections::hash_map - Rust](https://doc.rust-lang.org/stable/std/collections/hash_map/enum.Entry.html)，这里只说其一个我们用到的函数 `or_insert()`。

```rust
pub fn or_insert(self, default: V) -> &'a mut V
```

如果 key 对应的条目是空的，则在 HashMap 中插入一个键值对，值就是参数 `default`，然后返回这个值的可变引用。如果 key 对应的条目不是空的，直接返回这个值的可变引用。

下面通过代码理解一下 `entry()` 和 `or_insert()`：

```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let mut map: HashMap<&str, u32> = HashMap::new();

    map.entry("poneyland").or_insert(3);
    assert_eq!(map["poneyland"], 3);

    *map.entry("poneyland").or_insert(10) *= 2;
    assert_eq!(map["poneyland"], 6);
}
```



---

现在再回头看 `let count = map.entry(word).or_insert(0);` 这个语句就很明了了：先拿到 `map` 中键 `word` 对应的条目，如果这个条目是空的，就插入一个键值对 `("word", 0)`，然后返回这个 `0` 的可变引用，如果这个条目不是空的，就直接返回这个值的可变引用。然后后面就可以通过这个可变引用来修改值了，注意为了赋值必须首先使用星号（`*`）解引用 `count`。



### 2.3. 删除键值对

删除键值对使用 `remove()` 函数：

```rust
pub fn remove<Q: ?Sized>(&mut self, k: &Q) -> Option<V>
where
    K: Borrow<Q>,
    Q: Hash + Eq, 
```

从 HashMap 中删除一个键，如果该键在 HashMap 中，则以 `Option<T>` 类型返回该键的值，否则返回 `None`。

下面看一段代码示例：

```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, "a");
    assert_eq!(map.remove(&1), Some("a"));
    assert_eq!(map.remove(&1), None);
}
```





### 2.4. 清空 HashMap

```rust
pub fn clear(&mut self)
```

这个比较简单，不举例子了，注意慎用。







## 3. 访问 HashMap



###  3.1. 使用 `get()` 获取一个 key 对应的值

你可以通过 `get()` 方法并提供对应的键来从 HashMap 中获取值。

```rust
pub fn get<Q: ?Sized>(&self, k: &Q) -> Option<&V>
where
    K: Borrow<Q>,
    Q: Hash + Eq, 
```

示例代码：

```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name);
}
```

`get()` 会返回 `Option<&T>`，如果哈希表中没有其参数提供的键，则返回 `None`。

与很多其他集合的 `get()` 一样，其得到的是不可变引用。想要可变引用使用 `get_mut()`，用法一致。

```rust
pub fn get_mut<Q: ?Sized>(&mut self, k: &Q) -> Option<&mut V>
where
    K: Borrow<Q>,
    Q: Hash + Eq, 
```



### 3.2. 遍历整个 HashMap



这个不需要额外的函数，直接从代码理解吧，看下面的 `for` 循环写法。



```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```

输出

```
Yellow: 50
Blue: 10
```

这个 `for` 循环会以任意顺序打印出 HashMap 中的每一个键值对。



### 3.3. 检查一个 key 是否存在于 HashMap 中

```rust
pub fn contains_key<Q: ?Sized>(&self, k: &Q) -> bool
where
    K: Borrow<Q>,
    Q: Hash + Eq, 
```

示例代码：

```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, "a");
    assert_eq!(map.contains_key(&1), true);
    assert_eq!(map.contains_key(&2), false);
}
```





### 3.4. 检查 HashMap 是否为空

```rust
pub fn is_empty(&self) -> bool
```



HashMap 为空返回 `true`，否则返回 `false`。比较简单，不举例子了。



### 3.5. 获取键值对个数

```rust
pub fn len(&self) -> usize
```

返回类型 `uszie`，就是 HashMap 中键值对的数量。



## 4. 扩展：HashMap 与所有权

对于像 `i32` 这样的实现了 `Copy` trait 的类型，其值可以拷贝进哈希 map。

对于像 `String` 这样拥有所有权的值，其值将被移动而 HashMap 会成为这些值的所有者。

```rust
#![allow(unused)]

use std::collections::HashMap;

fn main() {
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);  // 这里 field_name 和 field_value 不再有效，
}
```

如果将值的引用插入 HashMap，这些值本身将不会被移动进 HashMap。但是这些引用指向的值必须至少在 HashMap 有效时也是有效的。