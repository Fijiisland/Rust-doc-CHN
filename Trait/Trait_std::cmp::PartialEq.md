# Trait std::cmp::PartialEq

### 描述：有关相等性比较的trait，这种比较是具有局部等价关系的。

对于不具备完全等价关系的类型，此trait允许局部等价性。比如，浮点数中NaN != NaN，所以浮点类型实现了PartialEq而不是Eq。

一般来说，此种等价性遵循：
 
 - 对称性：`a == b` 则有 `b == a`
 - 传递性：`a == b` 且 `b == c` 则有 `a == c`

注意，这些约束意味着，这个trait本身必须被对称且传递地实现：如果有 `T: PartialEq<U>` 且有 `U: PartialEq<V>` ，则推出 `U: PartialEq<T>` 和 `T: PartialEq<V>` 。

### 可继承

此trait可通过`#[derive]`来使用。当被继承于structs，若所有域均相等，那么两个实例相等；若任何两个域均不相等，那么两个实例一定不相等。当被继承于enums，每一个变体与自己相等并于其它变体不相等。

### 如何实现PartialEq

PartialEq trait仅要求目标类型实现其eq方法；ne方法是根据eq默认定义的。任何对ne的人为实现必须遵守eq与ne严格反对称的约束；也即，!(a == b)成立当且仅当a != b。

对PartialEq、PatialOrd以及Ord的实现必须是互容的。继承其中若干trait并人为实现其它trait很容易不小心造出互不相容的局面。

这有一个例子，一个域中若两本书的ISBN匹配则认为它们相同，即使其格式不同：

```rust
enum BookFormat { Paperback, Hardback, Ebook }
struct Book {
	isbn: i32,
	format: BookFormat,
}

impl PartialEq for Book {
	fn eq(&self, other: &Self) -> bool {
		self.isbn == other.isbn
	}
}

let b1 = Book { isbn: 3, format: BookFormat::Paperback };
let b2 = Book { isbn: 3, format: BookFormat::Ebook };
let b3 = Book { isbn: 10, format: BookFormat::Paperback };

assert!(b1 == b2);
assert!(b1 != b3);
```

### 如何比较两个不同的类型？

你可以进行比较的类型

```rust
#[derive(PartialEq)]
enum BookFormat {
	Paperback,
	Hardback,
	Ebook,
}
struct Book {
	isbn: i32,
	format: BookFormat,
}

// 实现 <Book> == <BookFormat> 的比较
impl PartialEq<BookFormat> for Book {
	fn eq(&self, other: &BookFormat) -> bool {
		self.format == *other
	}
}

// 实现 <BookFormat> == <Book> 的比较
impl PartialEq<Book> for BookFormat {
	fn eq(&self, other: &Book) -> bool {
		*self == other.format
	}
}

let b1 = Book { isbn: 3, format: BookFormat::Paperback };

assert!(b1 == BookFormat::Paperback);
assert!(BookFormat::Ebook != b1);
```