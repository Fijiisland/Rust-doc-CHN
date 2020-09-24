## Keyword type

### 描述： 为一个已知的类型定义一个别名

语法为：type Name = ExistingType;

#### Examples:

注意关键字type并不会创建一个新的类型：

```rust
type Meters = u32;
type Kilograms = u32;

let m: Meters = 3;
let k: Kilograms = 3;

assert_eq!(m, k);
```

在traits里，type关键字用来声明一个关联的类型：

```rust
trait Iterator {
    // 关联类型声明
    type Item;
    fn next(&mut self) -> Option<Self::item>;
}

struct Once<T>(Option<T>);

impl<T> Iterator for Once<T> {
    // 关联类型定义
    type Item = T;
    fn next(&mut self) -> Option<Self::Item> {
        self.0.take()
    }
}
```