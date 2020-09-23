此为Rust原始数组类型，表示为[T; N]，T代表数组元素类型，N代表非负的编译期常量数组尺寸。

有两种创建一个array的语法形式：

- 显式地定义一个含有各元素的列表：[x, y, z]
- 重复元素：[x; N]，产生一个有着N个x的拷贝的array，注意x的类型必须具备Copy语义


只要数组元素类型允许，大小在区间[0,32]的数组可实现以下traits:

- Debug
- IntoIterator (其为类型&[T; N]和&mut [T; N]实现)
- PartialEq, PartialOrd, Eq, Ord
- Hash
- AsRef, AsMut
- Borrow, BorrorMut
- Default

如果元素类型具备Copy语义，那么对应数组也具备Copy语义；Clone语义同理。

array支持切片，可在array上调用Primitive Type slice的方法。


### Examples：

```rust
let mut array: [i32; 3] = [0; 3];

array[1] = 1;
array[2] = 2;

assert_eq!([1, 2], &array[1..]);

for x in &array {
    print!("{} ", x);
}
```

要注意array本身不支持迭代：

```rust
let array: [i32, 3] = [0; 3];

for x in array { } // Wrong!
```

解决方案为将array强制转为一个slice类型：

```rust
for x in array.iter() { }
```

若array元素个数小于等于32，你可使用array引用的IntoIterator实现：

```rust
for x in &array
```

你可以使用切片模式来将元素移出一个array：

```rust
fn move_away(_: String) { /* 做一些不可描述的事 */ }

let [john, roa] = ["John".to_string(), "Roa".to_string()];
move_away(john);
move_away(roa); 
```