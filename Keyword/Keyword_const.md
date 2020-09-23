## 描述： 编译期常量以及确定性的函数。

有时一个确定的值会贯穿于整个程序不断被使用，自然地，不断copy它会变得不方便。除此之外，将它作为一个变量并将它带到各个需要它的函数里，不太可能也不太可取。综上所述，const关键字为代码复制提供一种方便的替代方案。

```rust
const THING: u32 = 0xABAD1DEA;

let foo = 123 + THING;
```
常量必须被显式注明类型，不像let，你不能忽略其类型并让编译器来判断。任何常量值都可以被定义为一个const，实际上，大部分情况都适合使用常量(常量函数除外)。

常量唯一被允许的生命周期为'static，举个例子，若你想定义一个常量字符串，它看起来是这样：

```rust
const WORDS: &'static str = "hello rust!";
```

多亏Rust语法糖，你一般不用这么显式地使用'static关键字：

```rust
const WORDS: &str = "hello convenience!";
```

const项看起来相当像static项，这会造成“什么时候该用哪个”的困扰。让我们简化这个问题：const项是内联的，也就是无论它们在哪被使用，编译期Rust会在对应的地方将const的名字直接替换成其值。而static的特点是，static变量指向内存中的一个单独分配的位置，这导致任何对此static变量的访问均是在访问此内存位置，也即共享(shared)的。这意味着，不像常量，static变量无析构器，且在整个代码中表现得像一个单独的值(在某内存角落做一个安静的美男子，不会被copy)；

在Rust中，常量和static变量的命名遵守SCREAMING_SNAKE_CASE。

const关键字还被用于raw pointer并与mut关键字配合使用，像是\*const T和\*mut T。这部分的详细解释位于Primitive Type pointer中。