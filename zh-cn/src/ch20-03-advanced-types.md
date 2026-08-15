## 高级类型

Rust 类型系统有一些我们此前提到、但尚未讨论的功能。首先，我们会在考察 newtype 为何可作为有用类型的过程中，一般性地讨论 newtype。然后介绍类型别名——一种与 newtype 相似但语义略有不同的功能。我们还会讨论 `!` 类型和动态大小类型。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### 使用 Newtype 模式实现类型安全与抽象

本节假定你已经读过前面的[“使用 Newtype 模式实现外部 Trait”][newtype]<!-- ignore -->一节。newtype 模式除了能完成迄今讨论的任务外，也适用于其他用途，包括以静态方式确保不同值绝不会混淆，以及指出值的单位。示例 20-16 展示过用 newtype 表示单位的例子：回想一下，`Millimeters` 和 `Meters` 结构体用 newtype 包装了 `u32` 值。如果编写一个形参类型为 `Millimeters` 的函数，那么程序若意外尝试用 `Meters` 类型或普通 `u32` 值调用它，就无法通过编译。

我们也可以使用 newtype 模式抽象掉类型的一些实现细节：新类型可以公开一套不同于私有内部类型 API 的公共 API。

Newtype 还能隐藏内部实现。例如，可以提供一个 `People` 类型，包装 `HashMap<i32, String>`，后者把人员 ID 与姓名关联起来存储。使用 `People` 的代码只与我们提供的公共 API 交互，例如向 `People` 集合中添加姓名字符串的方法；这些代码无需知道我们在内部为姓名分配了 `i32` ID。newtype 模式是一种实现封装、隐藏实现细节的轻量方式；第 18 章的[“隐藏实现细节的封装”][encapsulation-that-hides-implementation-details]<!-- ignore -->一节讨论过这一点。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-type-synonyms-with-type-aliases"></a>

<a id="type-synonyms-and-type-aliases"></a>

### 类型同义词与类型别名

Rust 可以声明<em>类型别名(type alias)</em>，为现有类型提供另一个名称。为此使用 `type` 关键字。例如，可以像这样为 `i32` 创建别名 `Kilometers`：

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

现在，别名 `Kilometers` 是 `i32` 的<em>同义词(synonym)</em>；与示例 20-16 中创建的 `Millimeters` 和 `Meters` 类型不同，`Kilometers` 并不是一个独立的新类型。`Kilometers` 类型的值会得到与 `i32` 类型值相同的处理：

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

因为 `Kilometers` 和 `i32` 是同一种类型，所以可以将这两种类型的值相加，也可以把 `Kilometers` 值传给接收 `i32` 形参的函数。然而，使用这种方法无法获得前面讨论的 newtype 模式所带来的类型检查优势。换句话说，如果在某处混淆了 `Kilometers` 与 `i32` 值，编译器不会报错。

类型同义词的主要用途是减少重复。例如，我们可能有下面这样冗长的类型：

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

在整个代码中反复把这个长类型写进函数签名和类型标注，既麻烦又容易出错。想象一下，一个项目里到处都是示例 20-25 所示的代码。

<Listing number="20-25" caption="在多个位置使用长类型">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

类型别名通过减少重复，让这段代码更易管理。在示例 20-26 中，我们为冗长的类型引入名为 `Thunk` 的别名，并可用较短的别名 `Thunk` 替换该类型的所有用法。

<Listing number="20-26" caption="引入类型别名 `Thunk` 以减少重复">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

这段代码读写起来容易多了！为类型别名选择有意义的名称还有助于传达意图（<em>thunk</em> 指稍后才会求值的代码，因此很适合作为被存储闭包的名称）。

类型别名也常与 `Result<T, E>` 类型结合使用，以减少重复。以标准库的 `std::io` 模块为例。I/O 操作通常返回 `Result<T, E>`，用于处理操作失败的情况。这个库有一个表示所有可能 I/O 错误的 `std::io::Error` 结构体。`std::io` 中很多函数都会返回 `Result<T, E>`，其中 `E` 为 `std::io::Error`，例如 `Write` trait 中的这些函数：

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` 重复了许多次。因此，`std::io` 有如下类型别名声明：

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

因为该声明位于 `std::io` 模块中，所以可以使用完全限定的别名 `std::io::Result<T>`；也就是说，它是一个由 `std::io::Error` 填入 `E` 的 `Result<T, E>`。最终，`Write` trait 的函数签名如下所示：

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

类型别名带来两方面帮助：它既让代码更容易编写，<em>又</em>为整个 `std::io` 提供了一致接口。因为它只是别名，所以仍然是另一个 `Result<T, E>`；这意味着可以在它上面使用任何适用于 `Result<T, E>` 的方法，以及 `?` 运算符等特殊语法。

### 永不返回的 Never 类型

Rust 有一个名为 `!` 的特殊类型，在类型论术语中称为<em>空类型(empty type)</em>，因为它没有值。我们更愿意称它为 <em>never 类型(never type)</em>，因为当函数永不返回时，它会占据返回类型的位置。下面是一个例子：

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

这段代码读作“函数 `bar` 永不返回”。永不返回的函数称为<em>发散函数(diverging function)</em>。我们无法创建 `!` 类型的值，所以 `bar` 绝不可能返回。

但一种永远无法为其创建值的类型有什么用呢？回想猜数字游戏中的示例 2-5；我们在示例 20-27 中重现了其中一小部分。

<Listing number="20-27" caption="一个分支以 `continue` 结尾的 `match`">

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

当时我们略过了这段代码的一些细节。在第 6 章[“`match` 控制流结构”][the-match-control-flow-construct]<!-- ignore -->一节中，我们讨论过 `match` 的所有分支都必须返回相同类型。例如，下面的代码无法工作：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

这段代码中 `guess` 的类型必须既是整数<em>又</em>是字符串，但 Rust 要求 `guess` 只能有一种类型。那么，`continue` 返回什么呢？为什么在示例 20-27 中，我们可以从一个分支返回 `u32`，而另一个分支以 `continue` 结尾？

你可能已经猜到，`continue` 的值为 `!`。也就是说，Rust 计算 `guess` 的类型时会查看两个匹配分支：前一个具有 `u32` 值，后一个具有 `!` 值。因为 `!` 永远不可能有值，所以 Rust 判定 `guess` 的类型为 `u32`。

对这种行为的正式描述是：`!` 类型的表达式可以<em>强制转换(coerce)</em>为任何其他类型。允许以 `continue` 结束这个 `match` 分支，是因为 `continue` 不返回值，而是把控制权移回循环顶部；所以在 `Err` 情况下，我们永远不会给 `guess` 赋值。

Never 类型也适用于 `panic!` 宏。回想我们在 `Option<T>` 值上调用的 `unwrap` 函数，它要么产生一个值，要么 panic；其定义如下：

```rust,ignore
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

这段代码中发生的情况与示例 20-27 的 `match` 相同：Rust 看到 `val` 的类型是 `T`，`panic!` 的类型是 `!`，所以整个 `match` 表达式的结果为 `T`。这段代码能够工作，是因为 `panic!` 不产生值，而是结束程序。在 `None` 情况下，我们不会从 `unwrap` 返回值，所以这段代码是有效的。

最后一种具有 `!` 类型的表达式是循环：

```rust,ignore
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

这里的循环永不结束，所以表达式的值为 `!`。不过，如果包含 `break`，情况就不同了，因为循环会在执行到 `break` 时终止。

<a id="dynamically-sized-types-and-the-sized-trait"></a>

### 动态大小类型与 `Sized` Trait

Rust 需要了解有关类型的某些细节，例如要为特定类型的值分配多少空间。这使得类型系统的一个角落起初有些令人困惑：<em>动态大小类型(dynamically sized type)</em>的概念。这些类型有时简称 <em>DST</em>，或称为<em>无大小类型(unsized type)</em>；它们让我们能够使用只有在运行时才知道大小的值来编写代码。

让我们深入了解一种全书一直在使用、名为 `str` 的动态大小类型。没错，不是 `&str`，而是单独的 `str`，它是 DST。在许多情况下（例如存储用户输入的文本），直到运行时我们才知道字符串有多长。这意味着不能创建 `str` 类型的变量，也不能接收 `str` 类型的实参。来看下面这段无法工作的代码：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust 需要知道应为特定类型的任何值分配多少内存，而一种类型的所有值必须占用相同的内存量。如果 Rust 允许编写这段代码，这两个 `str` 值就需要占用相同空间。但它们的长度不同：`s1` 需要 12 字节存储空间，而 `s2` 需要 15 字节。这就是无法创建容纳动态大小类型的变量的原因。

那么该怎么办呢？在这种情况下，你已经知道答案：让 `s1` 和 `s2` 的类型成为字符串切片（`&str`），而不是 `str`。回想第 4 章[“字符串切片”][string-slices]<!-- ignore -->一节，切片数据结构只存储切片的起始位置和长度。因此，尽管 `&T` 是一个存储 `T` 所在内存地址的单一值，字符串切片却是<em>两个</em>值：`str` 的地址及其长度。这样，我们就能在编译时知道字符串切片值的大小：它是 `usize` 长度的两倍。也就是说，无论它引用的字符串有多长，我们始终知道字符串切片的大小。一般来说，这就是 Rust 使用动态大小类型的方式：它们带有一小块额外的元数据，用来存储动态信息的大小。动态大小类型的黄金法则是，必须始终把动态大小类型的值放在某种指针后面。

我们可以把 `str` 与各种指针结合起来，例如 `Box<str>` 或 `Rc<str>`。其实你已经见过这种方式，只不过针对的是另一种动态大小类型：trait。每个 trait 都是动态大小类型，可以用 trait 名称引用它。第 18 章的[“使用 Trait 对象抽象共同行为”][using-trait-objects-to-abstract-over-shared-behavior]<!-- ignore -->一节提到，要把 trait 用作 trait 对象，必须将其放在指针之后，例如 `&dyn Trait` 或 `Box<dyn Trait>`（`Rc<dyn Trait>` 也可以）。

为了处理 DST，Rust 提供了 `Sized` trait，用于确定某种类型的大小能否在编译时获知。凡是大小在编译时已知的类型，都会自动实现该 trait。此外，Rust 会隐式地为每个泛型函数添加 `Sized` bound。也就是说，像下面这样的泛型函数定义：

```rust,ignore
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

实际上会被当作我们写了：

```rust,ignore
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

默认情况下，泛型函数只适用于大小在编译时已知的类型。不过，可以使用下面的特殊语法放宽这项限制：

```rust,ignore
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

`?Sized` trait bound 表示“`T` 可能是 `Sized`，也可能不是”，这种写法覆盖了泛型类型必须在编译时具有已知大小的默认规则。具有这一含义的 `?Trait` 语法仅适用于 `Sized`，不适用于任何其他 trait。

还请注意，我们把形参 `t` 的类型从 `T` 改成了 `&T`。因为该类型可能不是 `Sized`，所以需要把它放在某种指针后面。这里选择了引用。

接下来，我们将讨论函数与闭包！

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits
