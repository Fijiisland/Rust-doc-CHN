# Module std::cmp

### 描述：用于排序和比较的功能。

此模块包含各种为排序与比较值服务的工具，概括地说：

- [Eq](https://doc.rust-lang.org/std/cmp/trait.Eq.html)和[PartialEq](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)是允许你分别定义值之间的局部和完全相等性的traits。实现它们需重载==, 以及!=操作符。
- [Ord](https://doc.rust-lang.org/std/cmp/trait.Ord.html)和[PartialOrd](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)是允许你分别定义值之间的局部和完全有序的traits。实现它们需重载<, <=, >以及>=操作符。
- [Ordering](https://doc.rust-lang.org/std/cmp/enum.Ordering.html)是由[Ord](https://doc.rust-lang.org/std/cmp/trait.Ord.html)和[PartialOrd](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)中的主要函数返回的一种enum类型，描述一种顺序。
- [Reverse](https://doc.rust-lang.org/std/cmp/struct.Reverse.html)是一个struct，它让你能轻松将一种顺序颠倒。
- [max](https://doc.rust-lang.org/std/cmp/fn.max.html)和[min](https://doc.rust-lang.org/std/cmp/fn.min.html)是构建于[Ord](https://doc.rust-lang.org/std/cmp/trait.Ord.html)的函数，让你能在两个值中找出最大值和最小值。