## Keyword as

as关键字有两个用法：

1. 类型转换

as最常用来将一个基础类型转为另一个基础类型，as还能将指针转换成地址，将地址转换为指针，将一个指针转换为其它指针。

基础类型转换：
    
```rust    
let thing1: u8 = 89.0 as u8;
assert_eq!(thing1 as char, 'Y');
```

2. 重命名一个import进来的名字

### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/keyword.as.html)