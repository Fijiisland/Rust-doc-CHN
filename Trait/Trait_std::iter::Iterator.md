## Trait std::iter::Iterator

### 描述：此为std::iter模块的核心，一个处理迭代器的接口

此为主要的迭代器trait。

### Associated Types 涉及的类型

type Item

指代被迭代元素的类型。
    
### Required methods 需要实现的方法

fn next(&mut self) -> Option<Self::Item>

使迭代器前进并返回下一个值。

在迭代结束时返回None。一些个别迭代器用法会不停迭代，因此不断调用next()可能永远不会返回一个Some(Item):

```rust
let a = [1, 2, 3];

let mut iter = a.iter();

assert_eq!(Some(&1), iter.next());
assert_eq!(Some(&2), iter.next());
assert_eq!(Some(&3), iter.next());

// ... 最终迭代结束时返回None.
assert_eq!(None, iter.next());

// 更多的next调用可能永远返回一个None.
assert_eq!(None, iter.next());
assert_eq!(None, iter.next());
```

### Provided methods 提供的方法

fn size_hint(&self) -> (usize, Option<usize>)

返回一个迭代器剩余长度的边界元组。

特别地，此方法返回一个元组，第一个元素代表下界，第二个元素代表上界。

第二个元素为一个Option<usize>，如果它的值为None代表没有已知的上界，或者上界的值比usize还大。


### Implementation notes 实现笔记

迭代器的实现并不强制生成声明数量的元素。一个有bug的迭代器也许会生成比下界还少或者比上界还多的元素。

size_hint()主要用作优化，像为迭代器元素预留空间，但是在unsafe代码中的边界检查是不受信任的。

也就是说，实现需提供一个正确的估量，否则将违反trait的协议

此方法默认实现返回(0, None)，这对任意迭代器都是正确的。

#### Examples

基础例子：

```rust
let a = [1, 2, 3];
let iter = a.iter();

assert_eq!((3, Some(3)), iter.size_hint());
```

一个更复杂的例子：

```rust
let iter = (0..10).filter(|x| x % 2 == 0);
```
    
fn count(self) -> usize

消费迭代器，计算迭代次数并返回。

这个方法会重复调用next方法直至遇到None，然后返回它‘看见’Some值的次数。注意即使迭代器一个元素都不包含，这个方法也必须至少调用一次next。

### Overflow Behavior 溢出行为

count方法并不防止溢出，所以对元素数量超过usize::MAX的迭代器进行计数会造成错误以及程序panic。若启用了断言调试，那么可以保证出现一个panic。

### Panics 严重错误

此函数在迭代器元素数量超过usize::MAX时也许会造成panic。

#### Examples

基本用法：

```rust
let a = [1, 2, 3];
assert_eq!(a.iter().count(), 3);

let a = [1, 2 ,3 ,4 ,5];
assert_eq!(a.iter().count(), 5);
```    
    
fn last(self) -> Option<Self::Item>

消费迭代器，返回其末尾元素。

此方法评估迭代器知道迭代器返回None。此过程中，它将追踪当前的元素，当迭代器返回None后，last返回它看到的最后一个元素。

#### Examples

基础用法：

```rust
let a = [1, 2, 3];
assert_eq!(a.iter().last(), Some(&3));
```
    
fn nth(&mut self, n: usize) -> Option<Self::Item>

返回迭代器中第n个元素。

像大多数索引操作一样，它从0开始计数，所以nth(0)返回第一个值，nth(1)为第二个。

注意所有被遍历过的元素，以及最终被返回的元素，会被迭代器所消费。这意味着被遍历过的元素会被丢弃，因此，在同一个迭代器上调用nth(0)多次会返回不同的元素。

```rust
let a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let mut iter = a.iter();

iter.nth(6).unwrap();
println!("{:?}", iter);
```

此段代码输出为 Iter([8, 9, 10]),也就是说，被遍历的前五个元素以及返回的第六个元素均被消费且丢弃。

如果n大于等于迭代器的长度，nth()会返回None。

#### Examples

基本用法：

```rust
let a = [1, 2, 3];
assert_eq!(a.iter().nth(1), Some(&2));
```

调用多次造成消费：

```rust
let a = [1, 2, 3];

let mut iter = a.iter();

assert_eq!(iter.nth(1), Some(&2));
assert_eq!(iter.nth(1), None);
```

超出索引范围返回一个None：

```rust
let a = [1, 2, 3];
assert_eq!(a.iter().nth(10), None);
```