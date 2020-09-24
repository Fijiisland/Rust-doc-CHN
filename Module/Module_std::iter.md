## Module std::iter

### 描述： 模组定义：可组合的外部迭代。

如果你发现自己拥有某种类型的集合，并且需要对该集合的元素执行操作，那么很快就会遇到“迭代器”。迭代器在惯用的Rust代码中被大量使用。

让我们先谈谈这个模块的结构:

### Organization：

这个模块主要是由类型来组织的：

- Traits是核心部分：traits定义了存在着什么样的迭代器iterators，以及你怎样利用它们。这些traits的方法值得用一些时间来学习。
- Functions提供一些有用的方法来创建一些基础迭代器iterators。
- Structs经常作为属于这个模块的traits的各种方法的返回类型。一般你会希望去查看创建struct的方法而不是struct本身，这个说法的解释在之后的'Implementing Iterator'块中。

### Iterator

模块Iter的核心是Iterator trait，Iterator trait的核心长这个样子：

trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

一个迭代器有一个方法next，当它被调用时，返回一个Option<Item>。只要元素存在，方法next会返回一个Some(Item),一旦元素都被遍历之后，next返回一个None来表明迭代结束。单个迭代器可选择继续迭代，所以再次调用next可能会，也可能不会最终再次返回Some(Item)。

trait Iterator的完整定义同时包含若干其它方法，但是它们都是在next基础上开发的默认方法。

迭代器iterators也是可组合的，通常将它们连起来做更复杂形式的处理，这部分留在下面的Adapters块来解释。

### The three forms of iteration

有三种常用方法来从一个集合创建迭代器：

- iter()，迭代的类型为&T
- iter_mut()，迭代的类型为&mut T
- into_iter()，迭代的类型为T

在适当的情况下，标准库中的各种东西可以实现上面三个接口中的一个或多个。

### Implementing Iterator

创建一个你自己的迭代器iterator涉及两个步骤：首先创建一个struct来持有迭代器的状态，然后为这个struct实现Iterator trait。这便是为什么在Iter模块里有这么多structs的原因：每一个迭代器iterator和迭代器适配器iterator adapter都有一个struct。

我们试着写一个叫做Counter的迭代器，它能从1数到5:

```rust
// 第一步，创建一个struct：
struct Counter {
    count: usize,
}

// 我们希望从1开始数，也就是初始持有数0的状态，可以用new方法来辅助
// 并不是严格要求这么写，但比较方便
impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

// 为Counter实现Iterator trait
impl Iterator for Counter {
    // 我们用类型usize来计数
    type Item = uszie;
    
    // 只需实现next方法
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}

// 现在可以使用它了
let mut counter = Counter::new();
assert_eq!(counter.next(), Some(1));
assert_eq!(counter.next(), Some(2));
assert_eq!(counter.next(), Some(3));
assert_eq!(counter.next(), Some(4));
assert_eq!(counter.next(), Some(5));
assert_eq!(counter.next(), None);
```

### For Loops and IntoIterator

Rust的for循环实际上是为迭代器准备的语法糖，这有个基础例子：

```rust
let values = vec![1, 2, 3 ,4 ,5];

for x in values {
    println!("{}", x);
}
```

你会注意到一个细节：我们并没有在vector上调用方法来生成一个迭代器，那是为何？

这是由于，标准库里有一个叫IntoIterator的trait用来将容器转换为一个迭代器。这个trait有一个叫做into_iter的接口，它将实现了IntoIterator的东西转为一个迭代器。让我们康康Rust编译器是如何把上面这段语法糖代码展开的：

#### Rust's' de-sugar code:

```rust
let values = vec![1, 2, 3, 4, 5];
{
    let result = match IntoIterator::into_iter(values) {
        mut iter => loop {
            let next;
            match iter.next() {
                Some(val) => next = val,
                None => break,
            };
            let x = next;
            let () = { println!("{}", x); };
        },
    };
} 
```

在这段代码中，我们调用values的into_iter方法，并匹配其返回的迭代器，并不断调用其next方法直到next返回值为None，此时迭代结束。

有一个很微妙的地方：标准库对于IntoIterator的实现比较有趣：

```rust
impl<I: Iterator> IntoIterator for I
```

换句话说，所有Iterators均通过返回它们自己实现了IntoIterator，这意味着两件事：

1. 如果你写了一个Iterator，你可将它用于for循环
2. 如果你正在写一个集合，为它实现IntoIterator可让你的集合用于for循环

### Adapters

取用一个Iterator并返回另一个Iterator的函数通常叫做‘iterator adapters迭代器适配器’，它们是'适配器模式'的一种形式。

常见的iterator adapters迭代器适配器包括map，take以及filter，你可查看它们的文档已了解更多。

如果一个迭代器适配器引起了panics，迭代器会处于一种不确定状态（但却是内存安全的）。这种状态并不保证在Rust不同版本之间保持一致，所以你应该避免拾取从已经panic的迭代器的返回值。

### Laziness 惰性

迭代器（以及迭代器适配器）是惰性的，这意味仅创建一个迭代器并不能完成所有工作。直到你调用它的next之前，没有任何有效操作发生。这有时会造成困惑。举个例子，map方法对它所遍历的每个元素调用一个闭包：

```rust
let v = vec![1, 2, 3, 4, 5];
v.iter().map(|x| println!("{}",x));
```

当我们仅创建一个迭代器，而不是使用它时，这不会打印任何值。此时编译器会警告我们这种行为：

==warning: unused result that must be used: iterators are lazy and do nothing unless consumed.==

符合Rust的用法是用for_each代替或者for循环：

```rust
let v = vec![1, 2, 3, 4, 5];
v.iter().for_each(|x| println!("{}", x));
// or
for x in &v {
    println!("{}", x);
}
```

### Infinity

迭代器不必是有限的。这有个是无限迭代器的右开区间range：

```rust
let numbers = 0..;
```

通常使用迭代器适配器take将一个无限迭代器转为一个有限的：

```rust
let numbers = 0..;
let five_numbers = numbers.take(5);

for number in five_numbers {
    println!("{}", number);
}
```

这将打印0到4

你得记住，对于无限迭代器的方法，甚至是那些能在有限时间里数学地确定结果的方法，都可能无法终结。一个特例是，像min这样的方法，在普通情形下要求遍历迭代器中每一个元素，似乎不能在无限迭代器上顺利返回结果：

```rust
let ones = std::iter::repeat(1);
let least = ones.min().unwrap(); // Oh no!An infinite loop!
```

### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/iter/index.html)