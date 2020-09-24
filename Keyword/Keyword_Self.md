## Keyword Self

### 描述： trait或impl块中的实现类型，或类型定义中的当前类型

在类型定义中：

```rust
struct Node {
    elem: i32,
    // 此处的‘Self’指代‘Node’
    next: Option<Box<Self>>,
}
```

在一个impl块中：

```rust
struct Foo(i32);

impl Foo {
    fn new() -> Self {
        // 此处的‘Self’即为‘Foo’的别名
        Self(0)
    }
}

assert_eq!(Foo::new().0, Foo(0).0);
```

泛型参数是隐式的Self：

```rust
struct Wrap<T> {
    elem: T,
}

impl<T> Wrap<T> {
    fn new(elem: T) -> Self {
        Self { elem }
    }
}
```

在一个trait定义及与其相关联的impl块中：

```rust
trait Example {
    fn example() -> Self;
}

struct Foo(i32);

impl Example for Foo {
    fn example() -> Self {
        Self(42)
    }
}

assert_eq!(Foo::example().0, Foo(42).0);
```

### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/keyword.Self.html)