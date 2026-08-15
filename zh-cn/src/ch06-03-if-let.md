## 使用 `if let` 和 `let...else` 实现简洁控制流

`if let` 语法把 `if` 和 `let` 组合起来，以更简短的方式处理匹配某个模式的值并
忽略其他值。示例 6-6 匹配变量 `config_max` 中的 `Option<u8>`，但只想在值为
`Some` 变体时执行代码。

<Listing number="6-6" caption="只关心值为 `Some` 时执行代码的 `match`">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

如果值为 `Some`，模式会把内部值绑定到变量 `max`，再打印它。我们不想对 `None`
做任何事情，但为了满足 `match`，处理一个变体后仍必须添加 `_ => ()`，这是恼人的
样板代码。

可以改用 `if let` 写得更短。以下代码与示例 6-6 的 `match` 行为相同：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

`if let` 接收由等号分隔的模式和表达式。它与 `match` 的工作方式相同：表达式传给
`match`，模式作为第一个分支。本例模式是 `Some(max)`，`max` 绑定到 `Some` 内部
的值；随后可以像在对应匹配分支中一样，在 `if let` 块中使用 `max`。只有值与模式
匹配时，块中的代码才会运行。

使用 `if let` 意味着更少输入、更少缩进和更少样板代码，但也会失去 `match` 强制
执行的穷尽检查，无法确保没有遗漏任何情况。选择 `match` 还是 `if let`，取决于
具体场景，以及用穷尽检查换取简洁是否合适。

换句话说，可以把 `if let` 看成一种<em>语法糖(syntactic sugar)</em>：它表示匹配一个
模式时运行代码、忽略所有其他值的 `match`。

`if let` 还可以包含 `else`。`else` 对应的代码块，与等价 `match` 表达式中 `_`
分支的代码块相同。回想示例 6-4 的 `Coin` 定义，其中 `Quarter` 变体还保存一个
`UsState`。如果想在报出 25 美分硬币所属州的同时，统计见到的所有非 25 美分
硬币，可以使用 `match`：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

也可以使用 `if let` 和 `else`：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## 使用 `let...else` 保持“顺利路径”

一种常见模式是：值存在时执行某项计算，否则返回默认值。继续使用带有 `UsState`
值的硬币示例。假如要根据 25 美分硬币上的州有多古老来说一句有趣的话，可以为
`UsState` 添加检查州年代的方法：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

然后像示例 6-7 一样，用 `if let` 匹配硬币类型，在条件体内引入 `state` 变量。

<Listing number="6-7" caption="在 `if let` 内嵌套条件，检查某州在 1900 年是否已经存在">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

这能完成任务，却把主要工作推入 `if let` 语句体内；工作更复杂时，顶层分支之间
的关系可能难以理解。也可以利用表达式产生值的性质，让 `if let` 生成 `state`
或提前返回，如示例 6-8 所示（`match` 也能实现类似做法）。

<Listing number="6-8" caption="使用 `if let` 生成值或提前返回">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

不过这种写法也有些难读：`if let` 的一个分支产生值，另一个分支直接从函数返回。

为了更好地表达这种常见模式，Rust 提供了 `let...else`。它与 `if let` 很相似：
左侧是模式，右侧是表达式；但没有 `if` 分支，只有 `else` 分支。模式匹配时，模式
中的值绑定在外层作用域中；不匹配时，程序进入 `else` 分支，而该分支必须从函数
返回。

示例 6-9 展示了用 `let...else` 代替 `if let` 后，示例 6-8 的写法。

<Listing number="6-9" caption="使用 `let...else` 阐明函数中的控制流">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

这样，函数主体可以保持在“<em>顺利路径(happy path)</em>”上，不会像 `if let` 那样让两个
分支拥有明显不同的控制流。

如果程序逻辑用 `match` 表达过于冗长，请记住 Rust 工具箱中还有 `if let` 和
`let...else`。

## 小结

现在已经了解如何用枚举创建自定义类型，让值属于列举值集合中的一种；也看到标准
库的 `Option<T>` 如何借助类型系统避免错误。枚举值包含数据时，可以根据需要处理
的情况数量，使用 `match` 或 `if let` 提取并使用这些数据。

Rust 程序现在可以用结构体和枚举表达问题领域中的概念。在 API 中使用自定义类型
可以保证类型安全：编译器会确保函数只取得它所期望类型的值。

为了向用户提供组织良好、易于使用且只暴露必要内容的 API，下面转向 Rust 模块。
