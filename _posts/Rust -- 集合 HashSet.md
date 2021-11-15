---
title: Rust -- 集合 HashSet
date: 2021-10-27 23:08:06
updated: 2021-10-27 23:08:06
categories: [编程语言基础]
tags: [Rust]
toc: true
---

和大多编程语言一样，Rust 中的集合也是存 key 的，key 不可重复。



Rust 中 HashSet 的声明如下：

```rust
pub struct HashSet<T, S = RandomState> 
{ /* fields omitted */ }
```

使用集合 HashSet 需要先 `use std::collections::HashSet;`。

这篇文章只会介绍几个基础用法，用以入门，详细内容请看官方文档 [HashSet in std::collections - Rust](https://doc.rust-lang.org/stable/std/collections/struct.HashSet.html)。

另外要注意，Rust 中 HashSet 的实现，其实是把 HashMap 的键保留，值设成 `()` （但是在概念上，HashSet 把 HashMap 里的键称作值，HashMap 的值就相当于没有了）。建议先学习 [HashMap](https://gukaifeng.cn/archives/53/)。

<!--more-->



## 1. 新建一个 HashSet

新建 HashSet 要使用 `HashSet::new()` 函数。

```
#![allow(unused)]

use std::collections::HashSet;

fn main() {
    let mut set0: HashSet<i32> = HashSet::new();  // 新建一个空的 HashSet，元素类型为 i32
    
    let mut set1 = HashSet::new();  // 新建一个空的 HashSet，元素类型由 Rust 根据后面的插入操作推断，注意后面必须有插入操作代码，否则会编译错误，Rust 必须在编译前可以知道其中的元素应该是什么类型。
    set1.insert("abc");
}
```





## 2. 更新 HashSet

### 2.1. 插入 key

```rust
pub fn insert(&mut self, value: T) -> bool
```

向集合中添加一个值。  
如果集合中没有这个值，则返回 `true`。  
如果集合中存在这个值，则返回 `false`。

下面看示例代码：

```rust
#![allow(unused)]

use std::collections::HashSet;

fn main() {
    let mut set = HashSet::new();

    assert_eq!(set.insert(2), true);
    assert_eq!(set.insert(2), false);
    assert_eq!(set.len(), 1);
}
```





### 2.2. 删除 key

```rust
pub fn remove<Q: ?Sized>(&mut self, value: &Q) -> bool
where
    T: Borrow<Q>,
    Q: Hash + Eq, 
```

从集合中移除一个值，返回该值是否在集合中存在，存在返回 `true`，不存在返回 `false`。

下面看示例代码：

```rust
#![allow(unused)]

use std::collections::HashSet;

fn main() {
    let mut set = HashSet::new();

    set.insert(2);
    assert_eq!(set.remove(&2), true);
    assert_eq!(set.remove(&2), false);
}
```



### 2.3. 清空 HashSet

```rust
pub fn clear(&mut self)
```

这个比较简单，不举例子了，注意慎用。





## 3. 访问 HashSet



### 3.1. 遍历 HashSet

```rust
#![allow(unused)]

use std::collections::HashSet;

fn main() {
    let mut set = HashSet::new();
    
    set.insert(1);
    set.insert(2);
    set.insert(3);
    
    for i in &set {
        println!("{}", i);
    }
}
```

上面的 `for` 循环即可遍历 HashSet 中的所有元素，要注意遍历的顺序是任意的，并不是插入顺序。





### 3.2. 获取 HashSet 中一个 key 的引用

```rust
pub fn get<Q: ?Sized>(&self, value: &Q) -> Option<&T>
where
    T: Borrow<Q>,
    Q: Hash + Eq, 
```

返回对集合中值的引用（如果有的话），该引用等于给定值。注意返回的类型是 `Option<T>`，即如果给定的值不存在的话，会返回 `None`。

示例代码：

```rust
#![allow(unused)]

use std::collections::HashSet;

fn main() {
    let set: HashSet<_> = [1, 2, 3].iter().cloned().collect();
    assert_eq!(set.get(&2), Some(&2));
    assert_eq!(set.get(&4), None);
}
```







### 3.3. 检查一个 key 是否存在于 HashSet 中

```rust
pub fn contains<Q: ?Sized>(&self, value: &Q) -> bool
where
    T: Borrow<Q>,
    Q: Hash + Eq, 
```

示例代码：

```rust
#![allow(unused)]

use std::collections::HashSet;

fn main() {
    let set: HashSet<_> = [1, 2, 3].iter().cloned().collect();
    assert_eq!(set.contains(&1), true);
    assert_eq!(set.contains(&4), false);
}
```





### 3.4. 检查 HashSet 是否为空

```rust
pub fn is_empty(&self) -> bool
```



HashSet 为空返回 `true`，否则返回 `false`。比较简单，不举例子了。



### 3.5. 获取 HashSet 中元素个数

```rust
pub fn len(&self) -> usize
```

返回类型 `uszie`，就是 HashSet 中元素的数量。