<!-- Old headings. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

<a id="the-match-control-flow-construct"></a>

## `match` 控制流结构

Rust 提供了极其强大的 `match` 控制流结构，可以把一个值与一系列模式比较，再
根据匹配的模式执行代码。模式可以由字面值、变量名、通配符等多种内容组成；
[第 19 章][ch19-00-patterns]<!-- ignore -->会介绍所有模式及其作用。`match` 的
强大之处来自模式的表达能力，也来自编译器会确认所有可能情况都得到处理。

可以把 `match` 表达式想象成硬币分类机：硬币沿轨道滑下，轨道上有大小不同的孔，
每枚硬币会从遇到的第一个大小合适的孔中落下。同样，值依次经过 `match` 中的每个
模式，在第一个“容得下”它的模式处落入相应代码块并执行。

说到硬币，就用它们演示 `match`！可以编写一个函数，接收面额未知的美国硬币，
像分类机一样判断是哪种硬币，并返回美分数，如示例 6-3 所示。

<Listing number="6-3" caption="枚举及以枚举变体作为模式的 `match` 表达式">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

分析 `value_in_cents` 中的 `match`。首先是 `match` 关键字，后跟表达式，本例是值
`coin`。这与 `if` 的条件表达式很相似，但有一项重要区别：`if` 的条件必须求值为
布尔值，这里却可以是任意类型。本例 `coin` 的类型是第一行定义的 `Coin` 枚举。

接下来是 `match` <em>分支(arm)</em>。每个分支分为模式和代码两部分。第一个分支的
模式是值 `Coin::Penny`，随后用 `=>` 运算符分隔模式与要运行的代码；这里的代码
只是值 `1`。各分支之间用逗号分隔。

执行 `match` 时，会按顺序将结果值与每个分支的模式比较。模式与值匹配时，执行
关联代码；不匹配则继续下一个分支，就像硬币分类机。分支数量不限，示例 6-3 有
四个分支。

每个分支关联的代码都是表达式，匹配分支的表达式结果就是整个 `match` 表达式的
返回值。

如果分支代码像示例 6-3 一样很短，只返回一个值，通常不使用花括号。如果需要运行
多行代码，就必须使用花括号，此时分支后的逗号可省略。例如，下面的代码每次收到
`Coin::Penny` 都打印“Lucky penny!”，但仍返回代码块最后的值 `1`：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

<a id="patterns-that-bind-to-values"></a>

### 绑定值的模式

匹配分支还有一项实用功能：可以绑定匹配值的组成部分，从枚举变体中提取值。

例如，修改一个枚举变体，让它保存数据。1999 至 2008 年，美国发行的 25 美分
硬币一面印有 50 个州各自的设计；其他硬币没有州图案，所以只有 25 美分硬币带有
这个额外值。示例 6-4 修改 `Quarter` 变体，使其包含一个 `UsState` 值。

<Listing number="6-4" caption="`Quarter` 变体还保存 `UsState` 值的 `Coin` 枚举">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

假设一位朋友想收集全部 50 个州的 25 美分硬币。按硬币类型整理零钱时，我们还会
报出每枚 25 美分硬币对应的州名；如果朋友没有，就能加入收藏。

在 `match` 表达式中，为匹配 `Coin::Quarter` 变体的模式添加变量 `state`。匹配
到 `Coin::Quarter` 时，`state` 会绑定该硬币的州值，然后可以在分支代码中使用：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

调用 `value_in_cents(Coin::Quarter(UsState::Alaska))` 时，`coin` 的值就是
`Coin::Quarter(UsState::Alaska)`。逐一比较分支，直到 `Coin::Quarter(state)` 才
匹配；此时 `state` 绑定为 `UsState::Alaska`。随后在 `println!` 中使用该绑定，
从 `Coin` 的 `Quarter` 变体中取出内部州值。

<!-- Old headings. Do not remove or links may break. -->

<a id="matching-with-optiont"></a>

### 匹配 `Option<T>`

上一节希望从 `Option<T>` 的 `Some` 情况中取出内部 `T` 值。也可以像处理 `Coin`
枚举一样，用 `match` 处理 `Option<T>`！这次比较的是它的变体，但 `match` 的工作
方式完全相同。

假设要编写接收 `Option<i32>` 的函数：如果内部有值就加 `1`；如果没有值，就返回
`None`，不尝试任何操作。借助 `match`，函数很容易编写，如示例 6-5 所示。

<Listing number="6-5" caption="对 `Option<i32>` 使用 `match` 表达式的函数">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

仔细分析第一次执行 `plus_one`。调用 `plus_one(five)` 时，函数体中的 `x` 为
`Some(5)`，随后与每个匹配分支比较：

```rust,ignore
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

`Some(5)` 不匹配 `None`，继续下一个分支：

```rust,ignore
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` 与 `Some(i)` 匹配吗？匹配！二者变体相同。`i` 绑定到 `Some` 包含的值，
因此取得 `5`。随后执行分支代码，把 `i` 加 `1`，创建内部总值为 `6` 的新 `Some`。

再看示例 6-5 对 `plus_one` 的第二次调用，其中 `x` 为 `None`。进入 `match` 并与
第一个分支比较：

```rust,ignore
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

匹配成功！没有值可加，程序停止比较，并返回 `=>` 右侧的 `None`。第一个分支已经
匹配，因此不会比较其他分支。

`match` 与枚举的组合适用于许多场景。Rust 代码中会经常看到这种模式：匹配枚举，
把变量绑定到内部数据，再据此执行代码。开始时稍显棘手，但习惯以后，你会希望
所有语言都有它。这一直是用户喜爱的特性。

### 匹配必须穷尽所有情况

`match` 还有一方面需要讨论：分支模式必须覆盖所有可能情况。下面这个
`plus_one` 版本存在缺陷，无法编译：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

代码没有处理 `None`，会导致错误。幸好 Rust 能发现它。尝试编译会得到：

```console
{{#include ../../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust 知道没有覆盖全部情况，甚至知道遗漏了哪个模式！Rust 中的匹配必须<em>穷尽
(exhaustive)</em>所有可能性，代码才有效。尤其对于 `Option<T>`，Rust 不允许忘记
显式处理 `None`，从而防止我们在值可能为空时仍假定值存在，让前面所说的十亿美元
错误无法发生。

### 全匹配模式与 `_` 占位符

使用枚举时，可以对少数特定值执行特殊操作，对其余值执行默认操作。设想一个游戏：
掷出 3 时玩家不移动，而是得到一顶漂亮的新帽子；掷出 7 时失去一顶帽子；其他
数字则让玩家在棋盘上前进相应格数。下面的 `match` 实现这种逻辑。为简化示例，
掷骰结果是硬编码值，其他逻辑用没有函数体的函数表示：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

前两个分支的模式是字面值 `3` 和 `7`。最后一个分支覆盖其余所有值，模式是名为
`other` 的变量；该分支把它传给 `move_player` 函数。

虽然没有列出 `u8` 的全部可能值，代码仍可编译，因为最后一个模式匹配所有未明确
列出的值。这个<em>全匹配模式(catch-all pattern)</em>满足 `match` 必须穷尽的要求。
全匹配分支必须放在最后，因为模式按顺序求值。如果提前放置，其他分支永远不会
运行；在全匹配之后添加分支时，Rust 会发出警告。

如果需要全匹配却不想<em>使用</em>匹配值，可以使用 `_`：它是匹配任意值但不绑定该值
的特殊模式。这告诉 Rust 不会使用该值，因此不会产生未使用变量警告。

修改游戏规则：现在掷出 3 或 7 之外的数字必须重掷。不再需要使用全匹配值，所以
把变量 `other` 改为 `_`：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

这个示例同样满足穷尽要求，因为最后一个分支显式忽略了所有其他值，没有遗漏。

最后再修改一次规则：如果掷出的不是 3 或 7，本回合不发生任何事情。可以用单元值
（[“元组类型”][tuples]<!-- ignore -->一节提到的空元组类型）作为 `_` 分支的代码：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

这里明确告诉 Rust，不会使用与前面模式都不匹配的其他值，也不想在这种情况下运行
任何代码。

[第 19 章][ch19-00-patterns]<!-- ignore -->还会深入介绍模式和匹配。目前先继续学习
`if let` 语法，它适用于 `match` 表达式稍显冗长的情况。

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
