## 模式可用的所有位置

模式会出现在 Rust 中的许多位置，而你可能在没有意识到的情况下已经大量使用它们！本节讨论所有能够使用模式的位置。

### `match` 分支

正如第 6 章所述，我们在 `match` 表达式的分支中使用模式。严格来说，`match` 表达式由 `match` 关键字、要匹配的值，以及一个或多个匹配分支组成；每个分支包含一个模式，以及值与该分支模式匹配时要运行的表达式，如下所示：

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

例如，下面是示例 6-5 中的 `match` 表达式，它匹配变量 `x` 中的 `Option<i32>` 值：

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

这个 `match` 表达式中的模式，是每个箭头左侧的 `None` 和 `Some(i)`。

`match` 表达式的一项要求是必须具有<em>穷尽性(exhaustive)</em>，即必须考虑 `match` 表达式中值的所有可能情况。确保涵盖每种可能性的一种方式，是让最后一个分支使用<em>全匹配模式(catch-all pattern)</em>：例如，与任何值都匹配的变量名永远不会匹配失败，因此可以涵盖余下所有情况。

特殊模式 `_` 会匹配任何内容，但从不绑定到变量，因此常用于最后一个匹配分支。例如，当希望忽略未明确指定的任何值时，`_` 模式会很有用。本章稍后的[“忽略模式中的值”][ignoring-values-in-a-pattern]<!-- ignore -->一节将更详细地介绍 `_` 模式。

### `let` 语句

本章之前，我们只明确讨论过在 `match` 和 `if let` 中使用模式；但实际上，也曾在其他位置使用模式，包括 `let` 语句。例如，考虑下面这个简单的 `let` 变量赋值：

```rust
let x = 5;
```

每次使用这样的 `let` 语句时，都是在使用模式，尽管你可能没有意识到！更正式地说，`let` 语句如下所示：

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

在 `let x = 5;` 这类语句中，PATTERN 位置上的变量名只是模式的一种特别简单的形式。Rust 把表达式与模式进行比较，并为找到的任何名称赋值。因此，在 `let x = 5;` 示例中，`x` 是一个模式，含义是“把在这里匹配的内容绑定到变量 `x`”。由于名称 `x` 构成整个模式，这个模式实际上意味着“无论值是什么，都把全部内容绑定到变量 `x`”。

为了更清楚地观察 `let` 的模式匹配特性，考虑示例 19-1；它在 `let` 中使用模式解构元组。

<Listing number="19-1" caption="使用模式解构元组，一次创建三个变量">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

这里把一个元组与模式进行匹配。Rust 把值 `(1, 2, 3)` 与模式 `(x, y, z)` 比较，发现值与模式匹配，也就是两者的元素数量相同。因此，Rust 把 `1` 绑定到 `x`，把 `2` 绑定到 `y`，把 `3` 绑定到 `z`。可以把这个元组模式看作其中嵌套了三个独立的变量模式。

如果模式中的元素数量与元组中的元素数量不匹配，整体类型就不匹配，并会得到编译错误。例如，示例 19-2 尝试把包含三个元素的元组解构为两个变量，这是行不通的。

<Listing number="19-2" caption="错误地构造模式，其中变量数与元组元素数不匹配">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

尝试编译这段代码会产生以下类型错误：

```console
{{#include ../../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

要修复这个错误，可以使用 `_` 或 `..` 忽略元组中的一个或多个值，正如[“忽略模式中的值”][ignoring-values-in-a-pattern]<!-- ignore -->一节将展示的那样。如果问题是模式中的变量太多，解决方案就是移除变量，使变量数量等于元组中的元素数量，从而让类型匹配。

### 条件 `if let` 表达式

第 6 章讨论过如何使用 `if let` 表达式，主要把它作为只匹配一种情况的等价 `match` 的简写。`if let` 还可以选择性地拥有对应的 `else`，其中包含 `if let` 的模式不匹配时要运行的代码。

示例 19-3 表明，还可以混合使用 `if let`、`else if` 和 `else if let` 表达式。这样比 `match` 表达式更灵活，因为在 `match` 中只能表达一个与模式比较的值。此外，Rust 不要求一系列 `if let`、`else if` 和 `else if let` 分支中的条件彼此相关。

示例 19-3 中的代码根据对多个条件的一系列检查，决定背景应使用什么颜色。在这个示例中，我们创建了一些具有硬编码值的变量；真实程序可能从用户输入中接收这些值。

<Listing number="19-3" file-name="src/main.rs" caption="混合使用 `if let`、`else if`、`else if let` 和 `else`">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

如果用户指定了最喜欢的颜色，就把该颜色用作背景。如果没有指定最喜欢的颜色，而今天是星期二，背景颜色就是绿色。否则，如果用户以字符串形式指定年龄，并且能成功把它解析为数字，则根据数字值使用紫色或橙色。如果这些条件都不适用，背景颜色就是蓝色。

这种条件结构让我们能够支持复杂需求。使用这里的硬编码值，示例会打印 `Using purple as the background color`。

可以看到，`if let` 也能像 `match` 分支一样引入遮蔽现有变量的新变量：`if let Ok(age) = age` 这一行引入一个新的 `age` 变量，其中包含 `Ok` 变体内部的值，并遮蔽原有的 `age` 变量。这意味着必须把 `if age > 30` 条件放在该代码块中，不能把两个条件合并成 `if let Ok(age) = age && age > 30`。要与 30 比较的新 `age` 只有在左花括号开启新作用域后才有效。

使用 `if let` 表达式的缺点是编译器不检查穷尽性，而对 `match` 表达式会检查。如果省略最后的 `else` 块，因而漏掉对某些情况的处理，编译器不会提醒这个潜在逻辑 bug。

### `while let` 条件循环

`while let` 条件循环的结构与 `if let` 相似，只要模式持续匹配，它就允许 `while` 循环一直运行。示例 19-4 展示了一个等待线程间消息的 `while let` 循环，不过这里检查的是 `Result`，而不是 `Option`。

<Listing number="19-4" caption="只要 `rx.recv()` 返回 `Ok`，就使用 `while let` 循环打印值">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

这个示例依次打印 `1`、`2`、`3`。`recv` 方法从通道接收端取出第一条消息，并返回 `Ok(value)`。第 16 章初次看到 `recv` 时，我们直接解包错误，或使用 `for` 循环把它当作迭代器处理。不过，正如示例 19-4 所示，也可以使用 `while let`，因为只要发送器存在，`recv` 方法每收到一条消息就返回一次 `Ok`；发送端断开连接后，它会产生 `Err`。

### `for` 循环

在 `for` 循环中，紧跟 `for` 关键字的值就是模式。例如，在 `for x in y` 中，`x` 是模式。示例 19-5 演示了如何在 `for` 循环中使用模式，作为循环的一部分解构（即拆开）元组。

<Listing number="19-5" caption="在 `for` 循环中使用模式解构元组">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

示例 19-5 中的代码会打印以下内容：

```console
{{#include ../../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

我们使用 `enumerate` 方法调整迭代器，让它产生一个值和该值的索引，并把两者放入元组。产生的第一个值是元组 `(0, 'a')`。这个值与模式 `(index, value)` 匹配时，`index` 为 `0`，`value` 为 `'a'`，从而打印输出的第一行。

### 函数形参

函数形参也可以是模式。示例 19-6 中的代码声明名为 `foo` 的函数，它接收一个名为 `x`、类型为 `i32` 的形参；现在看起来应该很熟悉。

<Listing number="19-6" caption="在形参中使用模式的函数签名">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

`x` 部分就是模式！与 `let` 一样，可以把函数实参中的元组与模式进行匹配。示例 19-7 在把元组传给函数时拆分其中的值。

<Listing number="19-7" file-name="src/main.rs" caption="使用形参解构元组的函数">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

这段代码会打印 `Current location: (3, 5)`。值 `&(3, 5)` 与模式 `&(x, y)` 匹配，所以 `x` 的值是 `3`，`y` 的值是 `5`。

由于闭包与函数相似，正如第 13 章所讨论的，也可以像在函数形参列表中那样，在闭包形参列表中使用模式。

至此，你已经见过模式的多种用法，但模式在每个可用位置的工作方式并不完全相同。在某些位置，模式必须不可反驳；另一些情况下，则可以被反驳。下面讨论这两个概念。

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
