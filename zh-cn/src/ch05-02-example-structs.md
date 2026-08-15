## 使用结构体的示例程序

为了理解何时适合使用结构体，我们来编写一个计算矩形面积的程序。先使用单独变量，
再逐步重构，最终改用结构体。

使用 Cargo 新建名为 <em>rectangles</em> 的二进制项目。程序接收以像素为单位的矩形
宽度和高度，并计算矩形面积。示例 5-8 展示了项目 <em>src/main.rs</em> 中的一种实现。

<Listing number="5-8" file-name="src/main.rs" caption="根据单独的宽度和高度变量计算矩形面积">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

使用 `cargo run` 运行程序：

```console
{{#include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

这段代码把两个尺寸传给 `area` 函数，成功计算出矩形面积，但还可以让代码更清晰、
更易读。问题在 `area` 的签名中十分明显：

```rust,ignore
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

`area` 本应计算一个矩形的面积，但函数有两个形参，程序中没有任何地方表明它们
彼此相关。把宽度和高度组合起来会更易读、更易管理。第 3 章的
[“元组类型”][the-tuple-type]<!-- ignore -->一节已经讨论过一种做法：使用元组。

### 使用元组重构

示例 5-9 是使用元组的另一个版本。

<Listing number="5-9" file-name="src/main.rs" caption="用元组指定矩形的宽度和高度">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

从一个角度看，程序有所改进：元组增加了一点结构，现在只需传递一个实参。但从
另一个角度看，这个版本更不清晰：元组没有为元素命名，必须用索引访问各部分，
计算过程不够直观。

宽度与高度颠倒不会影响面积计算，但要在屏幕上绘制矩形时就会产生影响！我们必须
牢记 `width` 是元组索引 `0`，`height` 是索引 `1`。其他人使用代码时，要理解并
记住这一点更加困难。代码没有表达数据含义，因此更容易引入错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### 使用结构体重构

结构体通过为数据添加标签来赋予含义。可以把元组转换为结构体，为整体及各部分
分别命名，如示例 5-10 所示。

<Listing number="5-10" file-name="src/main.rs" caption="定义 `Rectangle` 结构体">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

这里定义了名为 `Rectangle` 的结构体，在花括号内定义 `width` 和 `height` 字段，
二者类型都是 `u32`。随后在 `main` 中创建宽 `30`、高 `50` 的具体实例。

`area` 函数现在只有一个名为 `rectangle` 的形参，其类型是 `Rectangle` 实例的
不可变借用。正如第 4 章所述，我们希望借用结构体而非取得所有权。这样 `main`
保留所有权，可以继续使用 `rect1`，所以函数签名和调用位置都使用 `&`。

`area` 访问 `Rectangle` 实例的 `width` 和 `height` 字段（访问借用的结构体实例
的字段不会移动字段值，所以经常会看到结构体借用）。现在的签名准确表达了意图：
使用 `Rectangle` 的宽和高计算其面积。宽高之间的关系得以体现，值也拥有描述性
名称，而不再使用 `0` 和 `1` 这样的元组索引，清晰度大有提升。

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### 通过派生特征添加实用功能

调试程序时，如果能打印 `Rectangle` 实例并查看所有字段值会很有帮助。示例 5-11
尝试像前几章一样使用 [`println!` 宏][println]<!-- ignore -->，但无法工作。

<Listing number="5-11" file-name="src/main.rs" caption="尝试打印 `Rectangle` 实例">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

编译代码会得到一条核心信息如下的错误：

```text
{{#include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

`println!` 宏可以使用多种格式。默认情况下，花括号要求它使用 `Display` 格式，
这种输出供最终用户直接阅读。此前见过的原始类型默认实现 `Display`，因为向用户
展示 `1` 或其他原始值通常只有一种合理方式。但结构体有更多展示可能：是否需要
逗号？是否打印花括号？是否显示所有字段？由于存在歧义，Rust 不会猜测我们的
意图，结构体也没有可供 `println!` 和 `{}` 占位符使用的 `Display` 实现。

继续阅读错误信息会看到这条有用提示：

```text
{{#include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

试一试！现在调用写成 `println!("rect1 is {rect1:?}");`。花括号内的 `:?` 指定
使用名为 `Debug` 的输出格式。`Debug` 特征让我们以对开发者有用的方式打印结构体，
以便在调试时查看其值。

编译修改后的代码。糟糕，仍然会报错：

```text
{{#include ../../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

但编译器再次给出有用提示：

```text
{{#include ../../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust <em>确实</em>提供了打印调试信息的功能，但必须显式选择为结构体启用它。为此，
在结构体定义前添加外部属性 `#[derive(Debug)]`，如示例 5-12 所示。

<Listing number="5-12" file-name="src/main.rs" caption="添加派生 `Debug` 特征的属性，并用调试格式打印 `Rectangle` 实例">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

现在运行程序不会报错，并会看到以下输出：

```console
{{#include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

很好！输出不算漂亮，却显示了这个实例所有字段的值，对调试很有帮助。结构体较大
时，更易读的输出很有用；可以在 `println!` 字符串中使用 `{:#?}` 代替 `{:?}`。
本例采用 `{:#?}` 会产生以下输出：

```console
{{#include ../../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

另一种用 `Debug` 格式打印值的方法是 [`dbg!` 宏][dbg]<!-- ignore -->。它取得表达式
的所有权（`println!` 取得引用），打印代码中调用 `dbg!` 的文件名、行号和表达式
结果，再归还该值的所有权。

> 注意：`dbg!` 把内容打印到标准错误流 `stderr`，而 `println!` 打印到标准输出流
> `stdout`。第 12 章的[“将错误写入标准错误而非标准输出”][err]<!-- ignore -->一节
> 会进一步讨论 `stderr` 和 `stdout`。

下面的示例同时关注赋给 `width` 字段的值和 `rect1` 整个结构体的值：

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

可以用 `dbg!` 包裹表达式 `30 * scale`。因为 `dbg!` 会归还表达式值的所有权，
`width` 得到的值与没有该调用时相同。我们不希望 `dbg!` 取得 `rect1` 的所有权，
所以在下一个调用中使用 `rect1` 的引用。输出如下：

```console
{{#include ../../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

第一部分输出来自 <em>src/main.rs</em> 第 10 行，调试表达式 `30 * scale`，结果为 `60`
（整数的 `Debug` 格式只打印值）。第 14 行的 `dbg!` 输出 `&rect1` 的值，也就是
`Rectangle` 结构体，并采用该类型美化后的 `Debug` 格式。分析代码行为时，`dbg!`
宏非常有用！

除了 `Debug`，Rust 还提供许多可配合 `derive` 属性使用的特征，为自定义类型增加
实用行为。[附录 C][app-c]<!-- ignore -->列出了这些特征及其行为。第 10 章会介绍
如何以自定义行为实现这些特征，以及如何创建自己的特征。除 `derive` 外还有许多
其他属性；更多信息参阅 [Rust 参考手册的“属性”一节][attributes]。

`area` 函数非常专用，只计算矩形面积。把这种行为与 `Rectangle` 结构体更紧密地
关联起来会很有帮助，因为它不适用于其他类型。下面继续重构，把 `area` 函数改为
定义在 `Rectangle` 类型上的 `area` 方法。

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.html
[println]: https://doc.rust-lang.org/std/macro.println.html
[dbg]: https://doc.rust-lang.org/std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: https://doc.rust-lang.org/reference/attributes.html
