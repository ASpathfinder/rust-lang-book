## 改进 I/O 项目

有了迭代器方面的新知识，我们可以使用迭代器改进第 12 章的 I/O 项目，让代码中的一些位置更清晰、更简洁。让我们看看迭代器如何改进 `Config::build` 和 `search` 函数的实现。

### 使用迭代器移除 `clone`

示例 12-6 添加了一些代码，它接收 `String` 值切片，通过索引访问切片并克隆其中的值，创建 `Config` 结构体实例，从而让 `Config` 拥有这些值。示例 13-17 再次展示了示例 12-23 中的 `Config::build` 函数实现。

<Listing number="13-17" file-name="src/main.rs" caption="再次展示示例 12-23 中的 `Config::build` 函数">

```rust,ignore
{{#rustdoc_include ../../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

当时，我们说不必担心效率不高的 `clone` 调用，因为将来会删除它们。现在时候到了！

这里需要 `clone`，是因为形参 `args` 中有一个包含 `String` 元素的切片，但 `build` 函数并不拥有 `args`。为了返回 `Config` 实例的所有权，必须克隆 `Config` 的 `query` 和 `file_path` 字段值，使 `Config` 实例能够拥有其值。

掌握迭代器的新知识后，可以修改 `build` 函数，让它接收迭代器的所有权作为实参，而不是借用切片。我们会使用迭代器功能，取代检查切片长度和索引特定位置的代码。由于迭代器会访问这些值，这将更清楚地表达 `Config::build` 函数在做什么。

一旦 `Config::build` 取得迭代器的所有权，并停止使用会产生借用的索引操作，就可以把迭代器中的 `String` 值移入 `Config`，而不再调用 `clone` 和进行新分配。

#### 直接使用返回的迭代器

打开 I/O 项目的 <em>src/main.rs</em> 文件，它应该如下所示：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

首先，把示例 12-24 中 `main` 函数的开头改为示例 13-18 中的代码，这次使用迭代器。在同时更新 `Config::build` 之前，代码无法编译。

<Listing number="13-18" file-name="src/main.rs" caption="把 `env::args` 的返回值传给 `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

`env::args` 函数返回迭代器！我们不再把迭代器的值收集到向量中，再把切片传给 `Config::build`；现在直接把 `env::args` 返回的迭代器所有权传给 `Config::build`。

接下来，需要更新 `Config::build` 的定义。把 `Config::build` 的签名改成示例 13-19 中的样子。代码仍然无法编译，因为还需要更新函数体。

<Listing number="13-19" file-name="src/main.rs" caption="更新 `Config::build` 的签名，使其接收迭代器">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

标准库中 `env::args` 函数的文档表明，其返回的迭代器类型是 `std::env::Args`，该类型实现 `Iterator` 特征并返回 `String` 值。

我们更新了 `Config::build` 函数的签名，让形参 `args` 使用带有特征约束 `impl Iterator<Item = String>` 的泛型类型，而不是 `&[String]`。第 10 章[“使用特征作为形参”][impl-trait]一节讨论过这种 `impl Trait` 语法的用法；它表示 `args` 可以是实现 `Iterator` 特征并返回 `String` 项的任意类型。

由于我们取得 `args` 的所有权，并会通过遍历修改 `args`，可以在 `args` 形参的声明中添加 `mut` 关键字，使其可变。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### 使用 `Iterator` 特征方法

接下来修复 `Config::build` 的函数体。由于 `args` 实现了 `Iterator` 特征，我们知道可以对它调用 `next` 方法！示例 13-20 更新了示例 12-23 中的代码，改为使用 `next` 方法。

<Listing number="13-20" file-name="src/main.rs" caption="修改 `Config::build` 的函数体，使用迭代器方法">

```rust,ignore,noplayground
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

请记住，`env::args` 返回值中的第一个值是程序名。我们希望忽略它并取得下一个值，所以先调用 `next`，不对返回值做任何处理。然后再次调用 `next`，取得要放入 `Config` 的 `query` 字段的值。如果 `next` 返回 `Some`，就使用 `match` 提取该值；如果返回 `None`，说明提供的实参不足，我们会提前返回 `Err` 值。对于 `file_path` 值，也执行相同操作。

<!-- Old headings. Do not remove or links may break. -->

<a id="making-code-clearer-with-iterator-adapters"></a>

### 使用迭代器适配器让代码更清晰

还可以在 I/O 项目的 `search` 函数中利用迭代器；示例 13-21 再次展示了示例 12-19 中的实现。

<Listing number="13-21" file-name="src/lib.rs" caption="示例 12-19 中 `search` 函数的实现">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

可以使用迭代器适配器方法，以更简洁的方式编写代码。这样还能避免使用可变的中间 `results` 向量。函数式编程风格倾向于尽量减少可变状态，使代码更清晰。移除可变状态还可能支持未来的增强，让搜索并行进行，因为无需管理对 `results` 向量的并发访问。示例 13-22 展示了这项修改。

<Listing number="13-22" file-name="src/lib.rs" caption="在 `search` 函数的实现中使用迭代器适配器方法">

```rust,ignore
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

回想一下，`search` 函数的目的是返回 `contents` 中包含 `query` 的所有行。与示例 13-16 中的 `filter` 类似，这段代码使用 `filter` 适配器，只保留 `line.contains(query)` 返回 `true` 的行。随后使用 `collect` 把匹配行收集到另一个向量中。简单多了！也可以随意对 `search_case_insensitive` 函数做出相同修改，改用迭代器方法。

要进一步改进，可以删除 `collect` 调用，并把返回类型改为 `impl Iterator<Item = &'a str>`，让 `search` 函数返回迭代器，使该函数本身成为迭代器适配器。请注意，还需要更新测试！修改前后分别使用 `minigrep` 工具搜索大型文件，观察行为差异。修改前，程序会等到收集所有结果后才打印；修改后，由于 `run` 函数中的 `for` 循环能够利用迭代器的惰性，会在找到每一行匹配结果时立即打印。

<!-- Old headings. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### 在循环与迭代器之间选择

接下来很自然的问题是，在自己的代码中应该选择哪种风格，以及为什么：示例 13-21 中的原始实现，还是示例 13-22 中使用迭代器的版本（假设返回前收集所有结果，而不是返回迭代器）。大多数 Rust 程序员更喜欢迭代器风格。一开始它稍难掌握，但熟悉各种迭代器适配器及其作用后，迭代器可能更容易理解。代码不再忙于处理循环和构建新向量的各种细节，而是专注于循环的高层目标。这会抽象掉一些常见代码，使这段代码独有的概念更容易看清，例如迭代器中每个元素都必须通过的筛选条件。

但这两种实现真的等价吗？直觉可能会认为更底层的循环更快。接下来讨论性能。

[impl-trait]: ch10-02-traits.html#traits-as-parameters
