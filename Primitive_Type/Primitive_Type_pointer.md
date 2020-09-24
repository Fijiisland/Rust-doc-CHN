## Primitive Type pointer

### 描述：最原始，不安全的指针。\*const T以及\*mut T。

在此之前建议看看Module std::ptr的文档。

在Rust中操作raw pointer不太常见，通常仅限于某些模式。Raw pointer可以是非对齐或是null的。然而，当一个raw pointer被解引用时(使用*操作符),它必须是非空且对齐的。

通过一个raw pointer使用*ptr = data进行存储并对旧的值调用drop，所以如果类型有着drop绑定且内存还未初始化时必须调用write--否则在未初始化的内存会被drop。

使用null和null_mut函数创建null指针，并用\*const T和\*mut T类型的is_null方法检查是否为null。\*const T以及\*mut T类型也为指针的数学计算定义了offset方法。

Common ways to create raw pointers 创建raw指针的常用方法 

- 使用&来强制转换为一个指针

```rust
let my_num: i32 = 10;
// 语义：my_num_ptr是一个指向i32类型的const指针,const是指指针无法改动指向的数据
let my_num_ptr: *const i32 = &my_num; 
let mut my_speed: i32 = 88;
let my_speed_ptr: *mut i32 = &mut my_speed;
```

将指针指向被包装boxed的值，需对box进行解引用，并用&取其地址：

```rust
let my_mum: Box<i32> = Box::new(10);
let my_num_ptr: *const i32 = &*my_mum;
let mut my_speed: Box<i32> = Box::new(88);
let my_speed_ptr: *mut i32 = &mut *my_speed;
```

这些操作并不会取走原分配内存的所有权而且并不要求资源管理，但是在这些内存生命周期结束后禁止再使用它的raw指针。

***

- 消费一个box （Box<T>)

```rust
into_raw函数会消费一个box并返回其raw指针。此函数不会破坏内容T或释放内存：

let my_speed: Box<i32> = Box::new(88);
let my_speed: *mut i32 = Box::into_raw(my_speed);

// 将所有权从Box<T>中提取出来后，我们有义务在之后将它释放：
unsafe {
    drop(Box::from_raw(my_speed));
}
```

注意这里的drop调用是为了更清楚，它表明我们对提供的值工作已完成并且它应被释放。

***

- Get it from C.

```rust
extern crate libc;

use std::mem;

unsafe {
    let my_num: *mut i32 = libc::malloc(mem::size_of::<i32>()) as *mut i32;
    if my_num.is_null() {
        panic!("failed to allocate memory");
    }
    libc::free(my_num as *mut libc::c_void);
}
```

一般你不会在Rust中使用malloc和free，但是C语言的APIs经常会创造很多指针，因此是Rust中raw指针的一种来源。（可能是来自于Old-School程序员的习惯）

### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/primitive.pointer.html)