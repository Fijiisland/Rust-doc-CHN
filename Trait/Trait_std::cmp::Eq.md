# Trait std::cmp::Eq

### 描述：有关相等性比较的trait，这种比较是具有完全等价关系的。

这意味着，除了 `a == b` 和 `a != b` 严格遵循反对称性，以下等式必须成立：(对于所有a,b,c)：

- 自反性：`a == a`；
- 对称性：`a == b` 则有 `b == a`
- 传递性：`a == b` 且 `b == c`， 则有`a == c`

此特性不能由编译器检查，因此Eq能推出PartialEq，并且无额外的方法。

### 可继承的

此trait可以通过`#[derive]`来使用。当被继承时，由于Eq并无额外的方法，它仅仅告知编译器，这是一个全等关系而不是部分等价。注意derive策略要求所有域均为Eq。

### 如何实现Eq

如果不使用derive策略，那就具体说明你的类型实现Eq，并且无方法实现：

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
impl Eq for Book {}
```