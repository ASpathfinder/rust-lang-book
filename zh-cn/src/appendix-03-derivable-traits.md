## 附录 C：可派生的 Trait

本书多处讨论过 `derive` 属性，可以把它应用于结构体或枚举定义。`derive` 属性会生成代码，为通过 `derive` 语法标注的类型实现具有自身默认实现的 trait。

本附录提供标准库中所有可与 `derive` 一起使用的 trait 参考。每一节都会介绍：

- 派生该 trait 会启用哪些运算符和方法
- `derive` 所提供的 trait 实现会做什么
- 实现该 trait 对类型意味着什么
- 允许或不允许实现该 trait 的条件
- 需要该 trait 的操作示例

如果想要不同于 `derive` 属性所提供的行为，请查阅各 trait 的[标准库文档](https://doc.rust-lang.org/std/index.html)<!-- ignore -->，了解如何手动实现它们。

这里列出的 trait 是标准库定义的 trait 中，唯一可以通过 `derive` 为你的类型实现的一组。标准库定义的其他 trait 没有合理的默认行为，所以需要根据想要实现的目标，以合理方式自行实现它们。

`Display` 是一个无法派生的 trait 示例，它负责面向最终用户的格式化。你应该始终考虑向最终用户显示类型的恰当方式。应当允许最终用户看到类型的哪些部分？哪些部分与他们相关？哪种数据格式对他们最有意义？Rust 编译器并不了解这些情况，所以无法提供恰当的默认行为。

本附录提供的可派生 trait 列表并不全面：库可以为自己的 trait 实现 `derive`，因此能与 `derive` 一起使用的 trait 列表实际上没有上限。实现 `derive` 需要使用过程宏，第 20 章的[“自定义 `derive` 宏”][custom-derive-macros]<!-- ignore -->一节介绍过这一点。

### 用于程序员输出的 `Debug`

`Debug` trait 在格式字符串中启用调试格式，通过在 `{}` 占位符中添加 `:?` 来表示。

`Debug` trait 允许为了调试而打印类型的实例，使你和其他使用该类型的程序员能够检查程序执行到特定时刻时的实例。

例如，使用 `assert_eq!` 宏时需要 `Debug` trait。如果相等断言失败，这个宏会打印作为实参传入的实例值，让程序员能够了解两个实例为何不相等。

### 用于相等比较的 `PartialEq` 与 `Eq`

`PartialEq` trait 允许比较某种类型的实例，检查它们是否相等，并启用 `==` 和 `!=` 运算符。

派生 `PartialEq` 会实现 `eq` 方法。在结构体上派生 `PartialEq` 时，只有<em>所有</em>字段都相等，两个实例才相等；只要<em>任意</em>字段不相等，实例就不相等。在枚举上派生时，每个变体与自身相等，而与其他变体不相等。

例如，使用 `assert_eq!` 宏时需要 `PartialEq` trait，因为它必须能够比较某种类型的两个实例是否相等。

`Eq` trait 没有任何方法。它的用途是表明：对于被标注类型的每个值，该值都与自身相等。`Eq` trait 只能应用于同时实现 `PartialEq` 的类型，但并非所有实现 `PartialEq` 的类型都能实现 `Eq`。浮点数类型就是一个例子：浮点数的实现规定，两个非数值（`NaN`）实例彼此不相等。

需要 `Eq` 的一种情况，是把值作为 `HashMap<K, V>` 的键，使 `HashMap<K, V>` 能够判断两个键是否相同。

### 用于顺序比较的 `PartialOrd` 与 `Ord`

`PartialOrd` trait 允许为了排序而比较某种类型的实例。实现 `PartialOrd` 的类型可以使用 `<`、`>`、`<=` 和 `>=` 运算符。只能把 `PartialOrd` trait 应用于同时实现 `PartialEq` 的类型。

派生 `PartialOrd` 会实现 `partial_cmp` 方法，该方法返回 `Option<Ordering>`；给定值无法产生顺序时，它就是 `None`。一种无法产生顺序、但同类型大多数值都能比较的值，是浮点数值 `NaN`。用任意浮点数和浮点数值 `NaN` 调用 `partial_cmp` 都会返回 `None`。

在结构体上派生时，`PartialOrd` 按各字段在结构体定义中的出现顺序比较字段值，从而比较两个实例。在枚举上派生时，枚举定义中较早声明的变体会被视为小于较晚列出的变体。

例如，`rand` crate 中的 `gen_range` 方法需要 `PartialOrd` trait；该方法会在范围表达式指定的范围内生成随机值。

`Ord` trait 让你能够知道：被标注类型的任意两个值之间都存在有效顺序。`Ord` trait 实现 `cmp` 方法，该方法返回 `Ordering` 而不是 `Option<Ordering>`，因为有效顺序始终存在。只能把 `Ord` trait 应用于同时实现 `PartialOrd` 和 `Eq` 的类型（而 `Eq` 又要求 `PartialEq`）。在结构体和枚举上派生时，`cmp` 的行为与通过 `PartialOrd` 派生的 `partial_cmp` 实现相同。

需要 `Ord` 的一种情况，是把值存入 `BTreeSet<T>`；这种数据结构根据值的排序顺序存储数据。

### 用于复制值的 `Clone` 与 `Copy`

`Clone` trait 允许显式创建值的深拷贝，复制过程可能涉及运行任意代码和复制堆数据。有关 `Clone` 的更多信息，请参阅第 4 章的[“变量与数据通过 Clone 交互”][variables-and-data-interacting-with-clone]<!-- ignore -->一节。

派生 `Clone` 会实现 `clone` 方法；为整个类型实现该方法时，它会对类型的每个部分调用 `clone`。这意味着要派生 `Clone`，类型中的所有字段或值也必须实现 `Clone`。

调用切片上的 `to_vec` 方法，是需要 `Clone` 的一种情况。切片不拥有其中包含的类型实例，但 `to_vec` 返回的向量需要拥有自己的实例，所以 `to_vec` 会对每一项调用 `clone`。因此，切片中存储的类型必须实现 `Clone`。

`Copy` trait 允许只复制存储在栈上的位来复制值，无需运行任意代码。有关 `Copy` 的更多信息，请参阅第 4 章的[“只在栈上的数据：Copy”][stack-only-data-copy]<!-- ignore -->一节。

`Copy` trait 不定义任何方法，以防程序员重载这些方法并违反“不运行任意代码”的假定。这样，所有程序员都可以假定复制值会非常快。

只要一种类型的所有组成部分都实现 `Copy`，就能在该类型上派生 `Copy`。实现 `Copy` 的类型也必须实现 `Clone`，因为实现 `Copy` 的类型拥有一个简单的 `Clone` 实现，它执行与 `Copy` 相同的任务。

很少有场景会要求 `Copy` trait；实现 `Copy` 的类型可以使用相关优化，意味着无需调用 `clone`，从而使代码更简洁。

凡是能通过 `Copy` 完成的事情，也都可以通过 `Clone` 完成，但代码可能更慢，或者必须在一些位置使用 `clone`。

### 用于把值映射到固定大小值的 `Hash`

`Hash` trait 允许使用哈希函数取得任意大小类型的实例，并把它映射为固定大小的值。派生 `Hash` 会实现 `hash` 方法。派生的 `hash` 方法实现会组合对类型各部分调用 `hash` 所得到的结果，这意味着要派生 `Hash`，所有字段或值也必须实现 `Hash`。

需要 `Hash` 的一种情况，是在 `HashMap<K, V>` 中存储键，以便高效存储数据。

### 用于默认值的 `Default`

`Default` trait 允许为类型创建默认值。派生 `Default` 会实现 `default` 函数。派生的 `default` 函数实现会对类型的每个部分调用 `default` 函数，这意味着要派生 `Default`，类型中的所有字段或值也必须实现 `Default`。

`Default::default` 函数常与第 5 章[“使用结构体更新语法从其他实例创建实例”][creating-instances-from-other-instances-with-struct-update-syntax]<!-- ignore -->一节讨论的结构体更新语法结合使用。可以自定义结构体的几个字段，再使用 `..Default::default()` 设置并使用其余字段的默认值。

例如，在 `Option<T>` 实例上使用 `unwrap_or_default` 方法时需要 `Default` trait。如果 `Option<T>` 是 `None`，`unwrap_or_default` 方法会返回 `Option<T>` 中所存类型 `T` 的 `Default::default` 结果。

[creating-instances-from-other-instances-with-struct-update-syntax]: ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[custom-derive-macros]: ch20-05-macros.html#custom-derive-macros
