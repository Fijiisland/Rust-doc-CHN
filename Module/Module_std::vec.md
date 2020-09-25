## Module std::vec

### 描述： 一个毗邻且可增长的数组类型，存放基于堆分配的内容，写作Vec<T>

Vectors有着O(1)的索引时间复杂度，均摊复杂度为O(1)的push(在尾部)以及O(1)的pop(从尾部)。

Vectors会确保永不分配超过isize::MAX个字节的空间。

#### Examples

你可以用new来显式地创建一个Vec<T>：

```rust
let v: Vec<i32> = Vec::new();
```

或是用vec!宏：

```rust
let v: Vec<i32> = vec![];
let v = vec![1, 2, 3, 4, 5];
let v = vec![0; 10];
```

你可用push将值放到一个vector尾部(需要时会对vector扩容)

```rust
let mut v = vec![1, 2];
v.push(3);
```

将值pop出来：

```rust
let mut v = vec![1, 2];
let two = v.pop();
```

Vectors支持索引(通过Index和IndexMut traits实现)

```rust
let mut v = vec![1, 2, 3];
let three = v[2];
v[1] = v[1] + 5;
```

### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/vec/index.html)