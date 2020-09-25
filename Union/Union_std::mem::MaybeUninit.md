## Union std::mem::MaybeUninit

### 描述：一种构建T类型的未初始化实例的包装类型。

一般来说，Rust的编译器认定，根据变量的类型要求，一个变量一定是被正确初始化了的。举个例子，一个类型为引用的变量一定是对齐切非空的。这些甚至在unsafe代码里，均是不变且恒定的规则。

因此，零初始化一个引用类型变量会造成即时的未定义行为，不管此引用是否曾被用来访问内存：

```rust
use std::mem::{self, MaybeUninit};

let x: &i32 = unsafe { mem::zeroed() };                       // 未定义行为

let x: &i32 = unsafe { MaybeUninit::zeroed().assume_init() }; // 未定义行为
```

类似地，完全未初始化的内存可能是任何内容，然而bool类型必须是true或false。因此，创建一个未初始化的bool也是未定义的行为：

```rust
use std::mem::{self, MaybeUninit};

let b: bool = unsafe { mem::uninitialized(); };               // 未定义行为

let b: bool = unsafe { MaybeUninit::uninit().assume_init() }; // 未定义行为
```

此外，未初始化的内存比较特殊，因为编译器清楚它并没有固定值。这使得在一个变量中包含未初始化的数据是未定义行为，即使这个变量类型为整型，否则它可以持有任意的固定位模式：

```rust
use std::mem::{self, MaybeUninit};

let x: i32 = unsafe { mem::uninitialize() };                  // 未定义行为

let x: i32 = unsafe { MaybeUninit::uninit().assume_init() };  // 未定义行为
```

Warning：需注意的是，Rust有关未初始化的整型的规则还不完整，但在结束之前，建议避免这种用法。

除此之外，谨记大多数类型除了在类型级别上被认为已初始化之外，还有额外的不变量。比如，一个被初始化为1的Vec<T>是被认定为已初始化的，因为Rust编译器唯一的要求是此Vec<T>的指针必须是非空的。创建像上述的一个Vec<T>不会造成立刻的未定义行为，但是会被可能是最安全的操作引发出未定义行为（包括drop行为）。

#### Examples：

MaybeUninit<T>被用作允许unsafe代码处理未初始化的数据。这将向Rust编译器表明-------此处的数据或许不会被初始化：

```rust
use std::mem::MaybeUninit;

// 创建一个显式未初始化引用。编译器清楚在一个MaybeUninit<T>中的数据有可能是非法的，因此这是UB：
let mut x = MaybeUninit::<&i32>::uinit();

// 为x设置一个合法值。
unsafe { x.as_mut_ptr().write(&0); }

//提取出已初始化的数据------这个操作仅在x被正确初始化情况下被允许。
let x = unsafe { x.assume_init() };
```

编译器便知道不需在此段代码上做任何不正确的假设或者优化。

你可以将MaybeUninit<T>想成类似Option<T>的存在，但是缺乏运行时追踪以及安全性检查。

### out-pointers 外部指针

你可以用MaybeUninit<T>实现"out-pointers"：我们不从函数返回数据，而是给函数传递一个指向一些内存(未初始化的)的指针，并将数据放到里面。当“控制负责存储返回数据的内存如何分配”这件事对调用者很重要时，并且你想避免不必要的移动，out-pointers会很有用：

```rust
use std::mem::MaybeUninit;

unsafe fn make_vec(out: *mut Vec<i32>) {
	// ‘write’并不会drop旧的内容，这很重要
	out.write(vec![1, 2 ,3]);
}

let mut v = MaybeUninit::uninit();
unsafe { make_vec(v.as_mut_ptr(); }
// 现在我们知道'v'已被初始化，这也确保vector得到正确的drop。
let v = unsafe { v.assume_init() };
assert_eq!(&v, &[1, 2, 3]);
```

### Initializing an array element-by-element: 逐元素地初始化一个数组

MaybeUninit<T>可用来逐元素地初始化一个大数组。

```rust
use std::mem::{self, MaybeUninit};

let data = {
	// 创建一个'MaybeUninit'的未初始化数组。'assume_init'是安全的，这是由
	// 于我们在此要求初始化的类型为很多个'MaybeUninits'，它们并不要求初始
	// 化。
	let mut data: [MaybeUninit<Vec<u32>>; 1000] = unsafe {
		MaybeUninit::uninit().assume_init()
	};
	
	// 丢弃一个'MaybeUninit'并无大碍。因此使用raw pointer，而非使
	// 用'ptr::write'进行赋值并不会导致旧的未初始化值被drop。同时若在此循环
	// 中出现了panic，说明产生了内存泄漏，但实际并无内存安全问题。
	for elem in &mut data[...] {
		*elem = MaybeUninit::new(vec![42]);
	}
	
	// 初始化工作已完成。现在将此数组转为已初始化类型。
	unsafe { mem::transmute<_, [Vec<u32>; 1000]>(data) }
};

assert_eq!(&data[0], &[42]);
```

你也可以操作部分初始化的数组，它在较低级数据结构中出现。

```rust
use std::mem::MaybeUninit;
use std::ptr;

// 创建一个元素为'MaybeUninit'的未初始化array.'assume_init'是安全的，这
// 是由于我们在此要求初始化的类型为很多个'MaybeUninits'，它们并不要求初始化。
let mut data: [MaybeUninit<String>; 1000] = unsafe { MaybeUninit::uninit().assume_init() };
// 计算我们已赋值元素的个数
let mut data_len: usize = 0;

for elem in &mut data[0..500] {
	*elem = MaybeUninit::new(String::from("hello"));
	data_len += 1;
}

// 对于array中的每一个元素，若它已被分配，我们将它drop掉
for elem in &mut data[0..data_len] {
	unsafe { ptr::drop_in_place(elem.as_mut_ptr()); }
}
```

### Initializing a struct field-by-field 逐字段地初始化一个struct

目前还不存在一种方法，其能支持创建指向MaybeUninit<Struct>中struct内部的一个字段的raw pointer或引用。这意味着，调用MaybeUninit::uninit::<Struct>()并对它内部的字段进行修改这种操作不太可能。

### Layout 布局

MaybeUninit<T>确保具有相同的大小，对齐以及ABI：

```rust
use std::mem::{MaybeUninit, size_of, align_of};
assert_eq!(size_of::<MaybeUninit<u64>>(), size_of::<u64>());
assert_eq!(align_of::<MaybeUninit<u64>>(), align_of<u64>());
```

然而，谨记包含MaybeUninit<T>的类型并不一定有着相同的布局。Rust一般不会确保一个像Foo<T>的字段顺序和一个Foo\<U\>相同，即使T与U有着相同的大小以及内存对齐。而且对于MaybeUninit<T>，任何的bit值均为合法，Rust编译器并不能进行non-zero/niche-filling优化，则可能导致更大的尺寸：

```rust
assert_eq!(size_of::<Option<bool>>(), 1);
assert_eq!(size_of::<Option<MaybeUninit<bool>>>(), 2);
```

如果类型T是FFI-safe的，then so is MaybeUninit<T> ???

### Implementations

pub const fn new(val: T) -> MaybeUninit<T>

通过给定值创建一个新的MaybeUninit<T>。对此函数的返回值调用assume_init方法是安全的。

注意drop一个MaybeUninit<T>并不会调用T的drop代码。若T已被初始化，确保将T drop掉是你的责任。

***

pub const fn uninit() -> MaybeUninit<T>

创建一个处于未初始化状态的MaybeUninit<T>。

注意drop一个MaybeUninit<T>并不会调用T的drop代码。若T已被初始化，确保将T drop掉是你的责任。

***

pub unsafe fn assume_init(self) -> T

此函数将假定目标MaybeUninit<T>容器内的值已经初始化，并将值从MaybeUninit<T>中取出返回。
注意它并不保证目标已经初始化，若对未初始化的目标操作会造成未定义行为。

#### Example:

```rust
use std::mem::MaybeUninit;

let mut x = MaybeUninit::<bool>::uninit();
unsafe { 
    x.as_mut_ptr().write(true); 
}
let x_init = unsafe { 
    x.assume_init() 
};
assert_eq!(x_init, true);
```

***

pub fn uninit_array<const LEN: usize>() -> [MaybeUninit<T>; LEN]

> 此为nightly版本的实验性API(maybe_uninit_uninit_array)

创建一个包含若干个MaybeUninit<T>的array，其处于未初始化状态。

如你想了解此API，请前往原文位置：[uninit_array](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html#method.uninit_array)

***

pub fn zeroed() -> MaybeUninit<T>

创建一个处于未初始化态的MaybeUninit<T>，且将其内存填充为0字节。这取决于T是否已做好被正确初始化的准备。例如，MaybeUninit<usize>::zeroed()已被初始化，但是MaybeUninit<&'static i32>::zeroed()并没有被初始化，因为引用禁止为null(若引用的内存被填充为0字节便为null)。

注意drop一个MaybeUninit<T>并不会调用T的drop代码。若T已被初始化，确保将T drop掉是你的责任。

#### Example

此方法的正确用法：将一个struct初始化为0，其中所有的字段均可持有bit-pattern 0作为一个合法值。(译注：本人对bit-pattern 0的理解为，若一个类型支持bit-pattern，那么0和1可以正确代表其一种‘值’，比如u8的bit-pattern 0对应的值为0；而bool类型的bit-pattern 0对应的值为false，而1对应false)

```rust
use std::mem::MaybeUninit;

let x = MaybeUninit::<(u8, bool)>::zeroed();
let x = unsafe { x.assume_init() };
assert_eq!(x, (0, false));
```

对此方法的错误使用：以0初始化一个struct，但其字段并不支持将0作为一个合法值。

```rust
use std::mem::MaybeUninit;

enum NotZero { One = 1, Two = 2 };

let x = MaybeUninit::<(u8, NotZero)>::zeroed();
let x = unsafe { x.assume_init() };
// 在pair内部，我们加入了一个'NotZero'，其并没有0值
// 此为未定义行为。
```

***

pub fn write(&mut self, val: T) -> &mut T

### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html)