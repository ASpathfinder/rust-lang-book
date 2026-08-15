## 面向对象语言的特征

编程社区对于一种语言必须具备哪些功能才能被视为面向对象，并没有共识。Rust 受到包括 OOP 在内的多种编程范式影响；例如，第 13 章探讨了来自函数式编程的功能。可以说，OOP 语言共享某些常见特征，即对象、封装和继承。下面分别了解这些特征的含义，以及 Rust 是否支持它们。

### 对象包含数据与行为

Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 合著的《设计模式：可复用面向对象软件的基础》（Addison-Wesley，1994 年）通常被称为“四人帮”一书，是一本面向对象设计模式目录。它这样定义 OOP：

> 面向对象程序由对象组成。<strong>对象(object)</strong>把数据以及操作这些数据的过程封装在一起。这些过程通常称为<strong>方法(method)</strong>或<strong>操作(operation)</strong>。

按照这个定义，Rust 是面向对象的：结构体和枚举拥有数据，`impl` 块为结构体和枚举提供方法。即使具有方法的结构体和枚举并不被<em>称为</em>对象，按照四人帮对对象的定义，它们也提供了相同的功能。

<a id="encapsulation-that-hides-implementation-details"></a>

### 隐藏实现细节的封装

通常与 OOP 相关的另一个方面是<em>封装(encapsulation)</em>，即使用对象的代码无法访问对象的实现细节。因此，与对象交互的唯一方式是通过其公开 API；使用对象的代码不应能够深入对象内部，直接改变数据或行为。这让程序员能够修改和重构对象内部，而不必修改使用该对象的代码。

第 7 章讨论过如何控制封装：可以使用 `pub` 关键字决定代码中的哪些模块、类型、函数和方法应当公开，而其他所有内容默认都是私有的。例如，可以定义一个 `AveragedCollection` 结构体，其中一个字段包含由 `i32` 值组成的向量。该结构体还可以有一个字段，保存向量中各值的平均数，这意味着每当有人需要平均数时，不必按需重新计算。换句话说，`AveragedCollection` 会替我们缓存计算出的平均数。示例 18-1 给出了 `AveragedCollection` 结构体的定义。

<Listing number="18-1" file-name="src/lib.rs" caption="维护整数列表及集合中各项平均数的 `AveragedCollection` 结构体">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

结构体标记为 `pub`，让其他代码能够使用它，但结构体内的字段仍保持私有。在这个例子中，这一点很重要，因为我们希望确保每当从列表中添加或移除一个值时，平均数也会更新。为此，我们在结构体上实现 `add`、`remove` 和 `average` 方法，如示例 18-2 所示。

<Listing number="18-2" file-name="src/lib.rs" caption="在 `AveragedCollection` 上实现公开方法 `add`、`remove` 和 `average`">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

公开方法 `add`、`remove` 和 `average` 是访问或修改 `AveragedCollection` 实例中数据的唯一方式。使用 `add` 方法向 `list` 添加项，或使用 `remove` 方法移除项时，这两个方法的实现都会调用私有方法 `update_average`，由它负责更新 `average` 字段。

我们让 `list` 和 `average` 字段保持私有，使外部代码无法直接向 `list` 字段添加项或从中移除项；否则，`list` 发生变化时，`average` 字段可能变得不同步。`average` 方法返回 `average` 字段中的值，让外部代码可以读取平均数，但不能修改它。

因为已经封装了 `AveragedCollection` 结构体的实现细节，所以将来可以轻松改变数据结构等方面。例如，可以为 `list` 字段使用 `HashSet<i32>`，而不是 `Vec<i32>`。只要公开方法 `add`、`remove` 和 `average` 的签名保持不变，使用 `AveragedCollection` 的代码就不需要修改。反之，如果把 `list` 设为公开，情况就不一定如此：`HashSet<i32>` 与 `Vec<i32>` 添加和移除项的方法不同，所以直接修改 `list` 的外部代码很可能必须改变。

如果封装是语言被视为面向对象所必需的特征，那么 Rust 满足这个要求。可以选择是否对代码的不同部分使用 `pub`，从而封装实现细节。

### 作为类型系统与代码共享机制的继承

<em>继承(inheritance)</em>是一种机制，对象可以通过它继承另一个对象定义中的元素，从而获得父对象的数据和行为，而无需再次定义。

如果语言必须具有继承才算面向对象，那么 Rust 就不是这样的语言。除非使用宏，否则无法定义一个继承父结构体字段和方法实现的结构体。

不过，如果你习惯把继承当作编程工具箱中的工具，可以根据最初选择继承的原因，在 Rust 中采用其他解决方案。

选择继承主要有两个原因。第一个是复用代码：可以为一个类型实现特定行为，而继承让你能够为另一个类型复用该实现。在 Rust 代码中，可以使用默认 trait 方法实现，以有限的方式做到这一点。示例 10-14 曾为 `Summary` trait 上的 `summarize` 方法添加默认实现。任何实现 `Summary` trait 的类型都可直接使用 `summarize` 方法，无需编写更多代码。这类似于父类拥有某个方法的实现，继承它的子类也拥有该方法实现。在实现 `Summary` trait 时，还可以重载 `summarize` 方法的默认实现，这类似于子类重载从父类继承的方法实现。

使用继承的另一个原因与类型系统有关：让子类型能够用于使用父类型的相同位置。这也称为<em>多态(polymorphism)</em>，即多个对象如果共享某些特征，就能在运行时彼此替换。

> ### 多态
>
> 对许多人而言，多态就是继承的同义词。但实际上，它是一个更通用的概念，指能够处理多种类型数据的代码。对于继承而言，这些类型通常是子类。
>
> Rust 改为使用泛型来抽象不同的可能类型，并使用 trait 约束来限制这些类型必须提供的内容。这有时称为<em>有界参数多态(bounded parametric polymorphism)</em>。

Rust 不提供继承，因而选择了另一组权衡。继承常常存在共享过多代码的风险。子类不一定应当共享父类的所有特征，但使用继承时却会如此。这可能降低程序设计的灵活性，还可能导致调用对子类没有意义的方法，或调用不适用于子类而引发错误的方法。此外，一些语言只允许<em>单继承(single inheritance)</em>（即子类只能继承一个类），进一步限制程序设计的灵活性。

基于这些原因，Rust 采用不同方法，使用特征对象代替继承，在运行时实现多态。下面看看特征对象如何工作。
