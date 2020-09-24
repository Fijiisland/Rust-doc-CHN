## Module std::ptr

### 描述: 通过raw pointer进行人工内存管理

### Safety 安全性

此模块中的许多函数将raw pointer作为参数并对它们进行读与写。为了安全，这些被传递的指针必须是合法(valid)的。一个指针是否合法取决于它被用来进行的操作(读或写)，以及被其访问的内存大小(也即是，有多少个字节被读/写)。大多数函数用\*mut T和\*const T指针类型来访问一个单一的值，这种情况下，文档省略了大小并隐式假设它有size_of::<T>()个字节。

目前为止，Rust暂时没有对于“合法性”的明确规则，为“合法性”提供的保证比较少：

- 一个null空指针永远不是合法的，就连访问[size zero](https://doc.rust-lang.org/nomicon/exotic-sizes.html#zero-sized-types-zsts)也不合法。
- 所有指针(null指针除外)对于size zero的访问均为合法。
- 要使指针有效，必须(并不是充分条件)使指针具备可解引用性：从指针处开始的，被给定尺寸的内存范围必须包含在一个单独分配对象的边界里。
- 由此模块中函数执行的所有访问均为非原子操作。就原子操作而言，其用于线程之间的同步。这意味着，从不同的线程执行两条并发的，指向同一位置的访存操作是未定义行为，除非两个访存操作仅对内存进行只读操作。注意这将显式地包含[read_volatile](https://doc.rust-lang.org/std/ptr/fn.read_volatile.html)以及[write_volatile](https://doc.rust-lang.org/std/ptr/fn.write_volatile.html)：Volatile访问不能被用作线程间同步。
- 只要底层对象是活跃的且无任何引用(仅raw pointer)被用于访问相同的内存，那么将一个引用转换为一个指针是合法的。

这些公理，搭配对于offset方法(用于指针算数)的谨慎使用，足以在unsafe代码里正确实现很多有用的东西。当alising规则被确定时，最终会提供健壮的保证。


### Alignment 对齐

像上文规则那样定义的合法raw pointers，对它们进行正确的内存对齐不是特别必要(这里的"正确"对齐取决于指针数据的类型，换句话说，\*const T必须对齐于mem::align_of::<T>())。然而，大多函数要求它们的参数是被正确对齐，并在它们的文档里显式声明了此要求的。对此，比较严重的异常为：[read_unaligned](https://doc.rust-lang.org/std/ptr/fn.read_unaligned.html)和[write_unaligned](https://doc.rust-lang.org/std/ptr/fn.write_unaligned.html)。

当一个函数要求正确对齐，即使访问的尺寸为0，它也会这么做，换句话说，即使内存并没有被触碰过。这种情况下考虑使用[NonNull::dangling](https://doc.rust-lang.org/std/ptr/struct.NonNull.html#method.dangling)。


### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/ptr/index.html)