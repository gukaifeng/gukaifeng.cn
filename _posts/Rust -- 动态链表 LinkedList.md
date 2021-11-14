---
title: Rust -- 动态链表 LinkedList
date: 2021-10-27 23:10:09
categories: [Technology]
tags: [Rust]
---



LinkedList 是 Rust 标准库中提供的双链表。

```rust
pub struct LinkedList<T> { /* fields omitted */ }
```

这里只介绍几个常用操作，更多内容请看文档 [LinkedList in std::collections - Rust](https://doc.rust-lang.org/stable/std/collections/struct.LinkedList.html)。

使用 LinkedList 需要先 `use std::collections::LinkedList;`。



<!--more-->

## 1. 新建 LinkedList

LinkedList 可以使用 `LinkedList::new()` 函数创建一个空的，也可以从现有的数组创建。

```rust
pub const fn new() -> LinkedList<T>
```

下面通过示例代码了解如何新建一个 LinkedList。

```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut list0 = LinkedList::from([1, 2, 3]);  // 元素类型由 Rust 推断
    let mut list1: LinkedList<u8> = LinkedList::from([1, 2, 3]);  // 元素类型指定为 u8
    
    let mut list2 = LinkedList::new();  // 元素类型由 Rust 根据后面的插入元素操作推断，注意必须在编译前有插入元素的操作，否则会编译错误，Rust 必须在编译前知道其中的元素类型。
    list2.push_back(1);
    list2.push_back(2);
    list2.push_back(3);
    
    let mut list3: LinkedList<u8> = LinkedList::new();  // 创建一个空的 LinkedList，指定元素类型为 u8。
}
```





## 2. 更新 LinkedList



### 2.1. 插入元素

```rust
pub fn push_back(&mut self, elt: T)
```

```rust
pub fn push_front(&mut self, elt: T)
```

`push_back()` 将一个元素追加到列表的后面。该操作的时间复杂度为 `O(1)`。

`push_front()` 将一个元素添加到列表的前面。该操作的时间复杂度为 `O(1)`。

### 2.2. 删除元素

```rust
pub fn pop_back(&mut self) -> Option<T>
```

```rust
pub fn pop_front(&mut self) -> Option<T>
```

`pop_back()` 从列表中移除最后一个元素并返回它（Option<T> 类型），如果为空则返回 `None`。该操作的时间复杂度为 `O(1)`。

`pop_front()` 从列表中移除第一个元素并返回它（Option<T> 类型），如果为空则返回 `None`。该操作的时间复杂度为 `O(1)`。





### 2.3. 清空链表



```rust
pub fn clear(&mut self)
```

这个比较简单，不举例子了，注意慎用。



### 2.4. 拼接链表

```rust
pub fn append(&mut self, other: &mut LinkedList<T>)
```

将 other 链表中的所有元素移到调用 `append()` 函数的列表的末尾。

这将重用来自 other 的所有结点，执行此操作后，other 为空。

时间复杂度与空间复杂度均为 `O(1)`。

示例代码如下：

```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut list1 = LinkedList::new();
    list1.push_back('a');

    let mut list2 = LinkedList::new();
    list2.push_back('b');
    list2.push_back('c');

    list1.append(&mut list2);

    let mut iter = list1.iter();
    assert_eq!(iter.next(), Some(&'a'));
    assert_eq!(iter.next(), Some(&'b'));
    assert_eq!(iter.next(), Some(&'c'));
    assert!(iter.next().is_none());

    assert!(list2.is_empty());
}
```





## 3. 访问 LinkedList



### 3.1. 检查一个元素是否在 LinkedList 中

```rust
pub fn contains(&self, x: &T) -> bool
where
    T: PartialEq<T>, 
```

如果 LinkedList 包含与给定值相等的元素，则返回 `true`。

示例代码如下：

```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut list: LinkedList<u32> = LinkedList::new();

    list.push_back(0);
    list.push_back(1);
    list.push_back(2);

    assert_eq!(list.contains(&0), true);
    assert_eq!(list.contains(&10), false);
}
```



### 3.2. 获取链表第一个元素

```rust
pub fn front(&self) -> Option<&T>
```

```rust
pub fn front_mut(&mut self) -> Option<&mut T>
```

`front()` 获取链表第一个元素的不可变引用（`Option<&T>` 类型），如果链表中没有元素则返回 `None`。

`front_mut()` 获取链表第一个元素的可变引用（`Option<&mut T>` 类型），如果链表中没有元素则返回 `None`。

示例代码如下：

```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut dl = LinkedList::new();
    assert_eq!(dl.front(), None);

    dl.push_front(1);
    assert_eq!(dl.front(), Some(&1));
}
```

```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut dl = LinkedList::new();
    assert_eq!(dl.front(), None);

    dl.push_front(1);
    assert_eq!(dl.front(), Some(&1));

    match dl.front_mut() {
        None => {}
        Some(x) => *x = 5,
    }
    assert_eq!(dl.front(), Some(&5));
}
```



### 3.3. 获取链表最后一个元素

```rust
pub fn back(&self) -> Option<&T>
```

```rust
pub fn back_mut(&mut self) -> Option<&mut T>
```

`back()` 获取链表最后一个元素的不可变引用（`Option<&T>` 类型），如果链表中没有元素则返回 `None`。

`back_mut()` 获取链表最后一个元素的可变引用（`Option<&mut T>` 类型），如果链表中没有元素则返回 `None`。

示例代码如下：

```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut dl = LinkedList::new();
    assert_eq!(dl.back(), None);

    dl.push_back(1);
    assert_eq!(dl.back(), Some(&1));
}
```

```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut dl = LinkedList::new();
    assert_eq!(dl.back(), None);

    dl.push_back(1);
    assert_eq!(dl.back(), Some(&1));

    match dl.back_mut() {
        None => {}
        Some(x) => *x = 5,
    }
    assert_eq!(dl.back(), Some(&5));
}
```



### 3.4. 遍历整个 LinkedList

遍历整个 ListedList 要使用迭代器。LinkedList 中只提供了前向迭代器，没有后向迭代器。

```rust
pub fn iter(&self) -> Iter<'_, T>
```

```rust
pub fn iter_mut(&mut self) -> IterMut<'_, T>
```

`iter()` 和 `iter_mut()` 都提供一个前向迭代器，区别已经体现在了函数名字上，前者提供的迭代器具有的是不可变引用，后者则是可变引用。 

示例代码如下：

```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut list: LinkedList<u32> = LinkedList::new();

    list.push_back(0);
    list.push_back(1);
    list.push_back(2);

    let mut iter = list.iter();
    assert_eq!(iter.next(), Some(&0));
    assert_eq!(iter.next(), Some(&1));
    assert_eq!(iter.next(), Some(&2));
    assert_eq!(iter.next(), None);
}
```



```rust
#![allow(unused)]

use std::collections::LinkedList;

fn main() {
    let mut list: LinkedList<u32> = LinkedList::new();

    list.push_back(0);
    list.push_back(1);
    list.push_back(2);

    for element in list.iter_mut() {
        *element += 10;
    }

    let mut iter = list.iter();
    assert_eq!(iter.next(), Some(&10));
    assert_eq!(iter.next(), Some(&11));
    assert_eq!(iter.next(), Some(&12));
    assert_eq!(iter.next(), None);
}
```





### 3.5. 检查 LinkedList 是否为空



```rust
pub fn is_empty(&self) -> bool
```

LinkedList 为空返回 `true`，否则返回 `false`。比较简单，不举例子了。

时间复杂度为 `O(1)`。



### 3.6. 获取 LinkedList 的长度

```rust
pub fn len(&self) -> usize
```

返回类型 `uszie`，就是 LinkedList 中结点的数量。