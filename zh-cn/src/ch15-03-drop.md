## 使用 `Drop` Trait 在清理时运行代码

对智能指针模式而言，第二个重要的 trait 是 `Drop`，它让你能够自定义值即将离开作用域时发生的行为。可以为任何类型实现 `Drop` trait，其中的代码可用于释放文件、网络连接等资源。

我们在智能指针的语境下介绍 `Drop`，是因为实现智能指针时几乎总会用到 `Drop` trait 的功能。例如，当 `Box<T>` 被丢弃时，它会释放箱子所指向的堆空间。

在某些语言中，对于某些类型，程序员每次使用完这些类型的实例后，都必须调用代码来释放内存或资源，例如文件句柄、套接字和锁。如果程序员忘了，系统可能会因负载过高而崩溃。在 Rust 中，可以指定每当值离开作用域时运行某段特定代码，编译器会自动插入这些代码。因此，不必小心翼翼地在程序中每个不再使用某类型实例的地方放置清理代码，也仍然不会泄漏资源！

通过实现 `Drop` trait，可以指定值离开作用域时要运行的代码。`Drop` trait 要求实现一个名为 `drop` 的方法，它接收对 `self` 的可变引用。为了观察 Rust 何时调用 `drop`，我们暂时用 `println!` 语句来实现 `drop`。

示例 15-14 展示了一个 `CustomSmartPointer` 结构体。它唯一的自定义功能是在实例离开作用域时打印 `Dropping CustomSmartPointer!`，以显示 Rust 何时运行 `drop` 方法。

<Listing number="15-14" file-name="src/main.rs" caption="实现 `Drop` trait 的 `CustomSmartPointer` 结构体；我们会把清理代码放在这个 trait 中">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

`Drop` trait 包含在 prelude 中，所以不必将它引入作用域。我们为 `CustomSmartPointer` 实现 `Drop` trait，并为 `drop` 方法提供调用 `println!` 的实现。`drop` 方法的方法体中可以放置任何希望在该类型实例离开作用域时运行的逻辑。这里打印一些文本，以直观展示 Rust 何时调用 `drop`。

在 `main` 中，我们创建两个 `CustomSmartPointer` 实例，然后打印 `CustomSmartPointers created`。在 `main` 末尾，这些 `CustomSmartPointer` 实例会离开作用域，Rust 将调用放在 `drop` 方法中的代码，打印最后的消息。请注意，我们无需显式调用 `drop` 方法。

运行这个程序时，会看到以下输出：

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

实例离开作用域时，Rust 自动为我们调用了 `drop`，从而执行指定的代码。变量的丢弃顺序与创建顺序相反，因此 `d` 先于 `c` 被丢弃。这个示例旨在直观展示 `drop` 方法的工作方式；通常你会指定类型所需的清理代码，而不是打印一条消息。

<!-- Old headings. Do not remove or links may break. -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

遗憾的是，要禁用自动 `drop` 功能并不容易。通常也没有必要禁用 `drop`，因为 `Drop` trait 的要点就在于自动完成清理。不过，有时你可能希望提前清理某个值。例如，使用管理锁的智能指针时，可能希望强制调用释放锁的 `drop` 方法，让同一作用域中的其他代码能够取得该锁。Rust 不允许手动调用 `Drop` trait 的 `drop` 方法；如果想在值的作用域结束前强制丢弃它，必须调用标准库提供的 `std::mem::drop` 函数。

如示例 15-15 所示，尝试修改示例 15-14 中的 `main` 函数，手动调用 `Drop` trait 的 `drop` 方法是行不通的。

<Listing number="15-15" file-name="src/main.rs" caption="尝试手动调用 `Drop` trait 的 `drop` 方法以提前清理">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

尝试编译这段代码时，会得到以下错误：

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

这条错误消息指出，不允许显式调用 `drop`。错误消息使用了<em>析构函数(destructor)</em>一词，这是对清理实例的函数的通用编程术语。析构函数与创建实例的<em>构造函数(constructor)</em>相对应。Rust 中的 `drop` 函数是一种特定的析构函数。

Rust 不允许显式调用 `drop`，因为在 `main` 结束时，Rust 仍会自动对该值调用 `drop`。这会导致<em>双重释放(double free)</em>错误，因为 Rust 会尝试清理同一个值两次。

我们既不能禁用值离开作用域时自动插入的 `drop`，也不能显式调用 `drop` 方法。因此，如果需要强制提前清理某个值，就使用 `std::mem::drop` 函数。

`std::mem::drop` 函数不同于 `Drop` trait 中的 `drop` 方法。调用它时，要把希望强制丢弃的值作为实参传入。该函数位于 prelude 中，因此可以修改示例 15-15 中的 `main`，像示例 15-16 那样调用 `drop` 函数。

<Listing number="15-16" file-name="src/main.rs" caption="调用 `std::mem::drop`，在值离开作用域前显式丢弃它">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

运行这段代码会打印以下内容：

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

文本 ``Dropping CustomSmartPointer with data `some data`!`` 出现在 `CustomSmartPointer created` 与 `CustomSmartPointer dropped before the end of main` 之间，这表明 `drop` 方法的代码在那一刻被调用，以丢弃 `c`。

可以通过多种方式使用 `Drop` trait 实现中指定的代码，使清理既方便又安全；例如，可以用它创建自己的内存分配器！借助 `Drop` trait 和 Rust 的所有权系统，你无需记得清理，因为 Rust 会自动完成。

你也不必担心意外清理仍在使用的值所导致的问题：确保引用始终有效的所有权系统，也会保证只在值不再使用时调用一次 `drop`。

现在我们已经研究了 `Box<T>` 和智能指针的一些特性，接下来看看标准库中定义的其他几种智能指针。
