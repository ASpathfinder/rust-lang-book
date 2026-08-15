## 高级函数与闭包

本节探讨与函数和闭包相关的一些高级功能，包括函数指针和返回闭包。

### 函数指针

我们已经讨论过如何把闭包传给函数；其实也可以把普通函数传给函数！当你想传递已经定义的函数，而不是定义新闭包时，这项技术很有用。函数会强制转换为 `fn` 类型（小写的 <em>f</em>），不要把它与 `Fn` 闭包 trait 混淆。`fn` 类型称为<em>函数指针(function pointer)</em>。通过函数指针传递函数，可以把函数用作其他函数的实参。

指定形参为函数指针的语法与闭包相似，如示例 20-28 所示。这里定义了函数 `add_one`，它给形参加 1。函数 `do_twice` 接收两个形参：一个函数指针，指向任何接收 `i32` 形参并返回 `i32` 的函数；以及一个 `i32` 值。`do_twice` 函数调用函数 `f` 两次，每次向它传入 `arg` 值，然后将两次函数调用的结果相加。`main` 函数以 `add_one` 和 `5` 为实参调用 `do_twice`。

<Listing number="20-28" file-name="src/main.rs" caption="使用 `fn` 类型接收函数指针作为实参">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

这段代码打印 `The answer is: 12`。我们指定 `do_twice` 中的形参 `f` 是一个接收一个 `i32` 类型形参并返回 `i32` 的 `fn`。然后便可在 `do_twice` 函数体中调用 `f`。在 `main` 中，可以把函数名称 `add_one` 作为第一个实参传给 `do_twice`。

与闭包不同，`fn` 是类型而不是 trait，因此我们直接把 `fn` 指定为形参类型，而不是声明一个以某个 `Fn` trait 为 trait bound 的泛型类型参数。

函数指针会实现全部三种闭包 trait（`Fn`、`FnMut` 和 `FnOnce`），这意味着凡是期望闭包的函数，都可以向它传递函数指针作为实参。最佳做法是使用泛型类型和某个闭包 trait 编写函数，使函数既能接收普通函数，也能接收闭包。

不过，有一种情况会希望只接收 `fn` 而不接收闭包：与不具备闭包的外部代码交互时。C 函数可以接收函数作为实参，但 C 没有闭包。

作为既可以使用内联定义的闭包、也可以使用具名函数的例子，我们来看标准库 `Iterator` trait 所提供的 `map` 方法。要用 `map` 方法把数字向量转换为字符串向量，可以像示例 20-29 那样使用闭包。

<Listing number="20-29" caption="通过 `map` 方法使用闭包将数字转换为字符串">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

也可以把具名函数而不是闭包作为实参传给 `map`。示例 20-30 展示了这种写法。

<Listing number="20-30" caption="通过 `map` 方法使用 `String::to_string` 函数将数字转换为字符串">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

请注意，因为存在多个名为 `to_string` 的可用函数，所以必须使用[“高级 Trait”][advanced-traits]<!-- ignore -->一节介绍的完全限定语法。

这里使用的是 `ToString` trait 中定义的 `to_string` 函数；标准库为所有实现 `Display` 的类型实现了这个 trait。

回想第 6 章[“枚举值”][enum-values]<!-- ignore -->一节，我们定义的每个枚举变体名称也会成为初始化函数。可以把这些初始化函数用作实现闭包 trait 的函数指针，这意味着可以将初始化函数指定为接收闭包的方法的实参，如示例 20-31 所示。

<Listing number="20-31" caption="通过 `map` 方法使用枚举初始化函数，从数字创建 `Status` 实例">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

这里，我们使用 `Status::Value` 的初始化函数，为调用 `map` 的范围内每个 `u32` 值创建 `Status::Value` 实例。有些人偏爱这种风格，有些人则偏爱使用闭包。两者会编译为相同代码，因此请选择对你而言更清晰的风格。

### 返回闭包

闭包由 trait 表示，这意味着不能直接返回闭包。在大多数想返回 trait 的情况下，可以改用实现该 trait 的具体类型作为函数返回值。然而，对于闭包通常无法这样做，因为它们没有可供返回的具体类型；例如，如果闭包捕获了其作用域中的任何值，就不允许把函数指针 `fn` 用作返回类型。

通常，你会改用第 10 章学过的 `impl Trait` 语法。使用 `Fn`、`FnOnce` 和 `FnMut`，可以返回任何函数类型。例如，示例 20-32 中的代码可以顺利编译。

<Listing number="20-32" caption="使用 `impl Trait` 语法从函数返回闭包">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

不过，正如第 13 章[“推断与标注闭包类型”][closure-types]<!-- ignore -->一节所说，每个闭包也都有自己独一无二的类型。如果需要处理多个签名相同但实现不同的函数，就需要为它们使用 trait 对象。看看编写示例 20-33 这样的代码会发生什么。

<Listing file-name="src/main.rs" number="20-33" caption="创建一个闭包的 `Vec<T>`，这些闭包由返回 `impl Fn` 类型的函数定义">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

这里有两个函数 `returns_closure` 和 `returns_initialized_closure`，两者都返回 `impl Fn(i32) -> i32`。请注意，尽管它们实现相同类型，所返回的闭包却不同。如果尝试编译，Rust 会告诉我们这种写法无法工作：

```text
{{#include ../../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

错误消息告诉我们，每当返回 `impl Trait` 时，Rust 都会创建一个唯一的<em>不透明类型(opaque type)</em>：我们既看不到 Rust 为我们构造内容的细节，也无法猜出 Rust 将生成的类型并亲自写出来。因此，尽管这些函数返回的闭包实现了同一个 trait——`Fn(i32) -> i32`，Rust 为它们分别生成的不透明类型仍各不相同。（这类似于第 17 章[“`Pin` 类型与 `Unpin` Trait”][future-types]<!-- ignore -->一节看到的情况：即使不同 async 块的输出类型相同，Rust 也会为它们生成不同的具体类型。）这个问题的解决方案我们已经见过几次：可以像示例 20-34 那样使用 trait 对象。

<Listing number="20-34" caption="创建一个闭包的 `Vec<T>`，这些闭包由返回 `Box<dyn Fn>` 的函数定义，因而具有相同类型">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

这段代码可以顺利编译。有关 trait 对象的更多信息，请参阅第 18 章的[“使用 Trait 对象抽象共同行为”][trait-objects]<!-- ignore -->一节。

接下来，让我们看看宏！

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[future-types]: ch17-03-more-futures.html
[trait-objects]: ch18-02-trait-objects.html
