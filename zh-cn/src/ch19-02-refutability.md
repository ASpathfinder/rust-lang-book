## 可反驳性：模式是否可能匹配失败

模式分为两种形式：可反驳与不可反驳。能够匹配传入的任何可能值的模式是<em>不可反驳(irrefutable)</em>的。例如，语句 `let x = 5;` 中的 `x` 可以匹配任何内容，因而不可能匹配失败。对某些可能值可能匹配失败的模式是<em>可反驳(refutable)</em>的。例如，表达式 `if let Some(x) = a_value` 中的 `Some(x)`；如果变量 `a_value` 中的值是 `None` 而不是 `Some`，`Some(x)` 模式就不会匹配。

函数形参、`let` 语句和 `for` 循环只能接受不可反驳模式，因为值不匹配时，程序无法执行任何有意义的操作。`if let` 和 `while let` 表达式以及 `let...else` 语句既接受可反驳模式，也接受不可反驳模式；但编译器会对不可反驳模式发出警告，因为按照定义，这些结构的目的是处理可能的失败：条件结构的功能正在于它能根据成功或失败执行不同操作。

通常不必担心可反驳与不可反驳模式之间的区别；不过，仍需熟悉<em>可反驳性(refutability)</em>这个概念，以便在错误消息中看到它时作出响应。在这些情况下，需要根据代码的预期行为，修改模式或使用该模式的结构。

下面看看在 Rust 要求不可反驳模式的位置尝试使用可反驳模式，以及反过来时会发生什么。示例 19-8 展示了一条 `let` 语句，但我们指定了可反驳模式 `Some(x)`。正如你可能预料的，这段代码无法编译。

<Listing number="19-8" caption="尝试在 `let` 中使用可反驳模式">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

如果 `some_option_value` 是 `None` 值，它就无法匹配模式 `Some(x)`，这意味着该模式可反驳。然而，`let` 语句只能接受不可反驳模式，因为代码无法对 `None` 值执行任何有效操作。在编译期，Rust 会指出我们在需要不可反驳模式的位置尝试使用了可反驳模式：

```console
{{#include ../../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

由于模式 `Some(x)` 没有（也无法）涵盖每个有效值，Rust 理所当然地产生编译错误。

如果在需要不可反驳模式的位置使用了可反驳模式，可以通过修改使用该模式的代码来修复：不再使用 `let`，而使用 `let...else`。这样，如果模式不匹配，花括号内的代码会处理该值。示例 19-9 展示了如何修复示例 19-8 中的代码。

<Listing number="19-9" caption="使用 `let...else` 及代码块配合可反驳模式，而不是使用 `let`">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

我们为代码提供了一条退路！这段代码完全有效，不过这意味着不能使用不可反驳模式而不收到警告。如果向 `let...else` 提供总会匹配的模式（例如 `x`），如示例 19-10 所示，编译器会发出警告。

<Listing number="19-10" caption="尝试在 `let...else` 中使用不可反驳模式">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust 会指出，将 `let...else` 与不可反驳模式配合使用没有意义，因为 `else` 永远不会到达：

```console
{{#include ../../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

基于这一原因，匹配分支必须使用可反驳模式，但最后一个分支除外，它应当用不可反驳模式匹配所有剩余值。Rust 允许在只有一个分支的 `match` 中使用不可反驳模式，但这种语法并没有特别大的用处，可以用更简单的 `let` 语句替代。

现在已经知道在哪里使用模式，以及可反驳模式与不可反驳模式的区别，下面介绍可用于创建模式的全部语法。
