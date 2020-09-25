## Struct std::sync::mpsc::TryIter

### 描述： 一个毗邻且可增长的数组类型，写作Vec<T>读作'Vector'

#### Examples

```rust
let mut vec = Vec::new();
vec.push(1);
vec.push(2);

assert_eq!(vec.len(), 2);
assert_eq!(vec[0], 1);

assert_eq!(vec.pop(), Some(2));
assert_eq!(vec.len(), 1);

vec[0] = 7;
assert_eq!(vec[0], 7);

vec.extend([1, 2, 3].iter().copied());

for x in &vec {
    println!("{}", x);
}
assert_eq!(vec, [7, 1, 2, 3]);
```



### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/sync/mpsc/struct.TryIter.html)