# Trait std::iter::IntoIterator

### 描述： 转换为一个迭代器Iterator.

通过为一个类型实现trait IntoIterator，你负责定义此类型如何被转为一个iterator。这对于描述某种集合的类型很常见。

实现IntoIterator的一个优点是，你的类型可与Rust的for循环语法配合运作。

#### Examples:

基本操作：

```rust
let v = vec![1, 2, 3];
let mut iter = v.into_iter();

assert_eq!(Some(1), iter.next());
assert_eq!(Some(2), iter.next());
assert_eq!(Some(3), iter.next());
assert_eq!(None, iter.next());
```

在为你的类型实现IntoIterator后：

```rust
// 一个简单的Vec<T> wrapper
#[derive(Debug)]
struct MyCollection(Vec<i32>);

impl MyCollection {
    fn new() -> MyCollection {
        MyCollection(Vec::new())
    }
    fn add(&mut self, elem: i32) {
        self.0.push(elem);
    }
}

impl IntoIterator for MyCollection {
    type Item = i32;
    type IntoIter = std::vec::IntoIter<Self::Item>;
    
    fn into_iter(self) -> Self::IntoIter {
        self.0.into_iter()
    }
}

let mut c = MyCollection::new();
c.add(0);
c.add(1);
c.add(2);

for (i, n) in c.into_iter().enumerate() {
    assert_eq!(i as i32, n);
}
```

### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html)