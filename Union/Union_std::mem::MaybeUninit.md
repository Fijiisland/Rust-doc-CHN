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

assume_init()

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