## Keyword dyn

### 描述： dyn是trait对象类型的前缀。

dyn关键字用于突出显示对相关trait的方法的调用是动态分配的。要以这种方式使用trait，它必须是“对象安全的”。

与泛型参数或impl Trait不同，编译器不知道传递的具体类型。因此，dyn Trait引用包含两个指针。一个指针指向数据(例如，一个结构体的实例)。另一个指针指向方法调用名到函数指针的映射(称为虚方法表或虚方法表)。

在运行时，当需要在dyn trait上调用一个方法时，会参考虚函数表来获得函数指针，然后调用该函数指针。

### 原文地址

[doc.rust-lang.org](https://doc.rust-lang.org/std/keyword.dyn.html)