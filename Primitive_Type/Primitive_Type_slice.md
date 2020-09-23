### 描述： 一个连续序列的具有动态大小的视图。这里的连续意味着元素被布局使得每个元素与其相邻元素的距离相同。
<br/>
更具体地说，Slices是一块内存的视图，表征为一个指针以及一个长度。

```rust
// 对一个Vec切片
let vec = vec![1, 2, 3];
let int_slice = &vec[..];
// 强制一个array转为一个slice
let str_slice: &[&str] = &["one", "two", "three"];
```

Slices类型是可变且共享的。共享的slice类型为&[T]，同时可变slice的类型是&mut [T]，这里T代表元素类型。举个例子，你可以更改可变slice指向的一块内存：

```rust
let mut x = [1, 2 ,3];
let x = &mut x[..];     // 取得x的全切片
x[1] = 7;
assert_eq!(x, &[1, 7, 3]);
```

## Implementations 方法实现

pub const fn len(&self) -> usize

返回slice的元素数量
    
### Examples

```rust 
    let a = [1, 2, 3];
    assert_eq!(a.len(), 3);
```

---
    
pub const is_empty(&self) -> bool

当slice长度为0时返回true

---
    
pub fn first(&self) -> Option<&T>

返回slice的第一个元素，若slice为空返回None
    
### Examples
    
```rust
let v = [10, 40, 30];
assert_eq!(Some(&10), v.first());

let w: &[i32] = &[];
assert_eq!(None, w.first());
```    

---
    
pub fn first_mut(&mut self) -> Option<&mut T>

返回指向slice第一个元素的可变指针，若slice为空返回None
    
### Examples

 ```rust   
let x = &mut[0, 1, 2];

if let Some(first) = x.first_mut() {
    *first = 5;
}

assert_eq!(x, &[5, 1, 2]);
```     

---

pub fn get\<I\>(&self, index: I) -> Option<&<I as SliceIndex<[T]>>::Output>
            where I: SliceIndex<[T]>
            
根据index的类型，返回一个元素的引用或者子切片subslice

- 如果是一个positon，返回此处的元素，如果超出范围返回None
- 如果是一个range，返回与range对应的subslice，超出范围返回None

### Examples

```rust 
let v = [10, 40, 30];
assert_eq!(Some(&40), v.get(1));
assert_eq!(Some(&[10, 40][..]), v.get(0..2));
assert_eq!(None, v.get(3));
assert_eq!(None, v.get(0..4));
```

---
    
pub fn get_mut\<I\>(&mut self, index: I) -> Option<&mut <I as SliceIndex[T]>::Output>
            where I: SliceIndex<[T]>
            
根据index的类型，返回一个元素的可变引用或者可变子切片subslice

### Examples

```rust 
let x = &mut [0, 1, 2];

if let Some(elem) = x.get_mut(1) {
    *elem = 42;
}
assert_eq!(x, &[0, 42, 2]);
```

---
    
pub fn get_unchecked\<T\r>(&self, index: I) -> &<I as SliceIndex<[T]>>::Output
            where I: SliceIndex<[T]>
            
返回元素或子切片的引用，不做边界检查。此方法一般用于performance critical的场景

若用户通过此方法查找一个超出边界的索引，即使返回的引用不被使用，也会造成未定义行为。

### Examples

```rust 
let x = &[1, 2, 4];
unsafe {
    assert_eq!(x.get_unchecked(1), &2);
}
```

---  
    
pub fn get_unchecked_mut\<T\>(&mut self, index: I) -> &mut <I as SliceIndex<[T]>>::Output
            where I: SliceIndex<[T]>
            
返回元素或子切片的可变引用，不做边界检查。此方法一般用于performance critical的场景

### Examples

```rust 
let x = &mut [1, 2, 4];

unsafe {
    let elem = x.get_unchecked_mut(1);
    *elem = 13;
}
assert_eq!(x, &[1, 13, 4]);
```   

---
    
pub fn as_ptr(&self) -> *const T

返回指向slice缓冲区的原始指针

调用者必须确保slice的生命周期长于此函数返回的指针，否则此指针最终可能会指向垃圾。

调用者必须确保指针所指向的内存从未被该指针，或者任何由该指针派生的指针写入。若你需要改动slice里的内容，请使用as_mut_ptr。

修改slice所引用的容器可能会造成其缓冲区被重分配，这会导致任何指向它的指针非法。

### Examples

```rust 
let x = &[1, 2 ,4];
let x_ptr = x.as_ptr();

unsafe {
    for i in 0..x.len() {
        assert_eq!(x.get_unchecked(i), &*x_ptr.get(i));
    }
}
```

---  
    
pub fn as_mut_ptr(&mut self) -> *mut T

返回指向slice缓冲区的不安全可变指针

调用者必须确保slice的生命周期长于此函数返回的指针，否则此指针最终可能会指向垃圾。

修改slice所引用的容器可能会造成其缓冲区被重分配，这会导致任何指向它的指针非法。

### Examples

```rust 
let x = &mut [1, 2, 4];
let x_ptr = x.as_mut_ptr();

unsafe {
    for i in 0..x.len() {
        *x_ptr.get(i) += 2;
    }
}
```