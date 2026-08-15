<a id="macros"></a>

## 宏

全书一直在使用 `println!` 之类的宏，但我们还没有全面探索宏是什么以及它如何工作。<em>宏(macro)</em>一词指 Rust 中的一系列功能——使用 `macro_rules!` 的声明宏，以及三种过程宏：

- 自定义 `#[derive]` 宏，用于指定通过结构体和枚举上的 `derive` 属性添加的代码
- 类属性宏，用于定义可在任何项上使用的自定义属性
- 类函数宏，看起来像函数调用，但对作为实参指定的 token 执行操作

我们会依次讨论这些宏，但先来看看，在已经有函数的情况下为什么还需要宏。

### 宏与函数的区别

从根本上说，宏是编写能够生成其他代码的代码的一种方式，这称为<em>元编程(metaprogramming)</em>。附录 C 会讨论 `derive` 属性，它能为你生成各种 trait 的实现。全书还使用过 `println!` 和 `vec!` 宏。所有这些宏都会<em>展开(expand)</em>，生成比手动编写部分更多的代码。

元编程有助于减少必须编写和维护的代码量，这也是函数的作用之一。不过，宏拥有一些函数不具备的额外能力。

函数签名必须声明函数形参的数量和类型。另一方面，宏可以接收数量可变的形参：可以用一个实参调用 `println!("hello")`，也可以用两个实参调用 `println!("hello {}", name)`。此外，宏会在编译器解释代码含义之前展开，因此宏可以为给定类型实现 trait。函数做不到这一点，因为它在运行时调用，而 trait 必须在编译时实现。

用宏而不是函数实现功能的缺点是，宏定义比函数定义更复杂，因为你编写的是能够生成 Rust 代码的 Rust 代码。由于这种间接性，宏定义通常比函数定义更难阅读、理解和维护。

宏与函数之间另一个重要区别是：必须先在文件中定义宏或将它引入作用域，<em>然后</em>才能调用；而函数可以定义在任何位置，也可以在任何位置调用。

<!-- Old headings. Do not remove or links may break. -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### 用于通用元编程的声明宏

Rust 中使用最广泛的宏形式是<em>声明宏(declarative macro)</em>。它们有时也称为“<em>示例宏(macros by example)</em>”、“`macro_rules!` 宏”，或者简称“宏”。声明宏的核心是允许你编写类似 Rust `match` 表达式的内容。第 6 章讨论过，`match` 表达式是一种控制结构：它接收表达式，把表达式的结果值与模式比较，然后运行与匹配模式关联的代码。宏也会把一个值同与特定代码相关联的模式进行比较：在这种情况下，值是传给宏的 Rust 字面源代码；模式与该源代码的结构进行比较；模式匹配时，与各模式关联的代码会替换传给宏的代码。所有这些都发生在编译期间。

定义宏时使用 `macro_rules!` 结构。我们通过考察 `vec!` 宏的定义，来探索如何使用 `macro_rules!`。第 8 章介绍了如何使用 `vec!` 宏创建含有特定值的新向量。例如，下面的宏创建一个包含三个整数的新向量：

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

还可以用 `vec!` 宏创建包含两个整数的向量，或者包含五个字符串切片的向量。函数无法做到相同的事情，因为事先不知道值的数量或类型。

示例 20-35 展示了 `vec!` 宏略加简化的定义。

<Listing number="20-35" file-name="src/lib.rs" caption="`vec!` 宏定义的简化版本">

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> 注意：标准库中 `vec!` 宏的实际定义包含预先分配正确内存量的代码。那段代码是一项优化，为了简化示例，这里没有包含它。

`#[macro_export]` 注解表示，只要定义该宏的 crate 被引入作用域，这个宏就应当可用。没有该注解，宏便无法被引入作用域。

接着，我们用 `macro_rules!` 和正在定义的宏名称开始宏定义，名称<em>不带</em>感叹号。在这里，名称 `vec` 后面是表示宏定义主体的花括号。

`vec!` 主体的结构与 `match` 表达式相似。这里有一个模式为 `( $( $x:expr ),* )` 的分支，后面是 `=>` 以及与该模式关联的代码块。如果模式匹配，就会生成关联的代码块。由于这是该宏中唯一的模式，所以只有一种有效的匹配方式；任何其他模式都会导致错误。更复杂的宏会有多个分支。

宏定义中的有效模式语法不同于第 19 章介绍的模式语法，因为宏模式匹配的是 Rust 代码结构，而不是值。下面逐一说明示例 20-29 中模式各部分的含义；完整的宏模式语法请参阅 [Rust 参考手册][ref]。

首先，我们使用一对圆括号包围整个模式。使用美元符号（`$`）在宏系统中声明一个变量，用于容纳与模式匹配的 Rust 代码。美元符号表明这是宏变量，而不是普通 Rust 变量。接下来是一对圆括号，捕获与其中模式匹配的值，以便在替换代码中使用。`$()` 内部是 `$x:expr`，它匹配任意 Rust 表达式，并将该表达式命名为 `$x`。

`$()` 后面的逗号表示，与 `$()` 中代码匹配的每个实例之间必须出现一个字面逗号分隔符。`*` 指定模式会匹配位于 `*` 之前的内容零次或多次。

用 `vec![1, 2, 3];` 调用这个宏时，`$x` 模式会分别与 `1`、`2` 和 `3` 三个表达式匹配三次。

现在来看与该分支关联的代码主体中的模式：`$()*` 内部的 `temp_vec.push()` 会为模式中匹配 `$()` 的每一部分生成一次，具体为零次或多次，取决于模式匹配了多少次。`$x` 会被每个匹配到的表达式替换。用 `vec![1, 2, 3];` 调用这个宏时，替换宏调用所生成的代码如下：

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

我们已经定义了一个宏，它可以接收任意数量、任意类型的实参，并生成代码来创建包含指定元素的向量。

如需进一步了解如何编写宏，请查阅在线文档或其他资源，例如由 Daniel Keep 发起、Lukas Wirth 延续编写的 [“The Little Book of Rust Macros”][tlborm]。

### 从属性生成代码的过程宏

第二种宏形式是<em>过程宏(procedural macro)</em>，它的行为更像函数（也属于过程的一种）。过程宏接收一些代码作为输入，对这些代码执行操作，并产生一些代码作为输出；它不像声明宏那样与模式匹配，再用其他代码替换原代码。过程宏分为三类：自定义 `derive` 宏、类属性宏和类函数宏，它们的工作方式都相似。

创建过程宏时，定义必须位于具有特殊 crate 类型的独立 crate 中。这是出于复杂的技术原因，我们希望未来能够消除这项限制。示例 20-36 展示了如何定义过程宏，其中 `some_attribute` 是使用某种具体宏类别的占位符。

<Listing number="20-36" file-name="src/lib.rs" caption="定义过程宏的示例">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

定义过程宏的函数接收 `TokenStream` 作为输入，并产生 `TokenStream` 作为输出。`TokenStream` 类型由 Rust 自带的 `proc_macro` crate 定义，表示一个 token 序列。这就是宏的核心：宏所操作的源代码构成输入 `TokenStream`，宏生成的代码构成输出 `TokenStream`。该函数上还附有一个属性，用来指定正在创建哪一类过程宏。同一个 crate 中可以包含多种过程宏。

下面看看不同种类的过程宏。我们先介绍自定义 `derive` 宏，再说明另外两种形式之间的小差异。

<!-- Old headings. Do not remove or links may break. -->

<a id="how-to-write-a-custom-derive-macro"></a>

<a id="custom-derive-macros"></a>

### 自定义 `derive` 宏

我们来创建一个名为 `hello_macro` 的 crate，它定义一个名为 `HelloMacro` 的 trait，trait 中只有一个名为 `hello_macro` 的关联函数。我们不要求用户为每种类型实现 `HelloMacro` trait，而是提供一个过程宏，让用户可以用 `#[derive(HelloMacro)]` 标注类型，从而获得 `hello_macro` 函数的默认实现。默认实现会打印 `Hello, Macro! My name is TypeName!`，其中 `TypeName` 是定义该 trait 的类型名称。换句话说，我们将编写一个 crate，使其他程序员能使用它编写示例 20-37 这样的代码。

<Listing number="20-37" file-name="src/main.rs" caption="使用过程宏时，我们的 crate 用户将能够编写的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

完成后，这段代码会打印 `Hello, Macro! My name is Pancakes!`。第一步是创建一个新的库 crate：

```console
$ cargo new hello_macro --lib
```

接下来，在示例 20-38 中定义 `HelloMacro` trait 及其关联函数。

<Listing file-name="src/lib.rs" number="20-38" caption="将与 `derive` 宏结合使用的简单 trait">

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

现在我们有了 trait 及其函数。此时，crate 用户可以像示例 20-39 那样实现该 trait，以获得所需功能。

<Listing number="20-39" file-name="src/main.rs" caption="如果用户手动实现 `HelloMacro` trait，代码会是什么样子">

```rust,ignore
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

然而，用户需要为想与 `hello_macro` 一起使用的每种类型编写实现块；我们希望免去他们这项工作。

此外，目前还无法为 `hello_macro` 函数提供会打印 trait 所实现类型名称的默认实现：Rust 不具备反射能力，因此无法在运行时查找类型名称。我们需要宏在编译时生成代码。

下一步是定义过程宏。在撰写本书时，过程宏需要位于自己的 crate 中。这项限制最终可能会被取消。组织 crate 与宏 crate 的惯例是：对于名为 `foo` 的 crate，自定义 `derive` 过程宏 crate 命名为 `foo_derive`。让我们在 `hello_macro` 项目内新建一个名为 `hello_macro_derive` 的 crate：

```console
$ cargo new hello_macro_derive --lib
```

这两个 crate 密切相关，所以我们在 `hello_macro` crate 的目录中创建过程宏 crate。如果修改 `hello_macro` 中的 trait 定义，也必须修改 `hello_macro_derive` 中的过程宏实现。这两个 crate 需要分别发布，使用它们的程序员必须把两者都添加为依赖并引入作用域。也可以让 `hello_macro` crate 把 `hello_macro_derive` 用作依赖，并重新导出过程宏代码。不过，我们现在组织项目的方式允许程序员在不需要 `derive` 功能时，仍然使用 `hello_macro`。

需要把 `hello_macro_derive` crate 声明为过程宏 crate。稍后你会看到，我们还需要 `syn` 和 `quote` crate 的功能，所以要把它们添加为依赖。请在 `hello_macro_derive` 的 <em>Cargo.toml</em> 文件中添加以下内容：

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

要开始定义过程宏，请把示例 20-40 中的代码放入 `hello_macro_derive` crate 的 <em>src/lib.rs</em> 文件。请注意，在添加 `impl_hello_macro` 函数的定义之前，这段代码无法编译。

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="大多数过程宏 crate 处理 Rust 代码时都需要的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

请注意，我们把代码分成了负责解析 `TokenStream` 的 `hello_macro_derive` 函数，以及负责转换语法树的 `impl_hello_macro` 函数；这让编写过程宏更加方便。对于你见到或创建的几乎每一个过程宏 crate，外层函数（这里是 `hello_macro_derive`）中的代码都相同。内层函数（这里是 `impl_hello_macro`）主体中指定的代码则会根据过程宏的用途而变化。

我们引入了三个新 crate：`proc_macro`、[`syn`][syn]<!-- ignore --> 和 [`quote`][quote]<!-- ignore -->。`proc_macro` crate 随 Rust 提供，因此无需把它添加到 <em>Cargo.toml</em> 的依赖中。`proc_macro` crate 是编译器提供的 API，使我们能从自己的代码中读取和操作 Rust 代码。

`syn` crate 把字符串形式的 Rust 代码解析成我们可以对其执行操作的数据结构。`quote` crate 再把 `syn` 数据结构转换回 Rust 代码。这些 crate 大大简化了我们可能想处理的各种 Rust 代码的解析工作：为 Rust 代码编写完整解析器绝非易事。

当库用户在类型上指定 `#[derive(HelloMacro)]` 时，会调用 `hello_macro_derive` 函数。这之所以可行，是因为我们在这里用 `proc_macro_derive` 标注了 `hello_macro_derive` 函数，并指定了与 trait 名称匹配的 `HelloMacro`；这是大多数过程宏遵循的惯例。

`hello_macro_derive` 函数首先把 `input` 从 `TokenStream` 转换为随后可以解释并执行操作的数据结构。这正是 `syn` 发挥作用之处。`syn` 中的 `parse` 函数接收 `TokenStream`，并返回表示已解析 Rust 代码的 `DeriveInput` 结构体。示例 20-41 展示了通过解析字符串 `struct Pancakes;` 得到的 `DeriveInput` 结构体中的相关部分。

<Listing number="20-41" caption="解析示例 20-37 中带有宏属性的代码后得到的 `DeriveInput` 实例">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

该结构体的字段表明，我们解析的 Rust 代码是一个标识符 `ident`（<em>identifier</em>，意为名称）为 `Pancakes` 的单元结构体。该结构体还有更多用于描述各种 Rust 代码的字段；有关详情，请查阅 [`syn` 的 `DeriveInput` 文档][syn-docs]。

稍后我们会定义 `impl_hello_macro` 函数，在其中构建想要包含的新 Rust 代码。但在此之前请注意，`derive` 宏的输出也是 `TokenStream`。返回的 `TokenStream` 会添加到 crate 用户编写的代码中，所以当用户编译自己的 crate 时，就能获得我们在修改后的 `TokenStream` 中提供的额外功能。

你可能已经注意到，如果这里对 `syn::parse` 函数的调用失败，我们会调用 `unwrap` 使 `hello_macro_derive` 函数 panic。过程宏出错时必须 panic，因为为了符合过程宏 API，`proc_macro_derive` 函数必须返回 `TokenStream`，而不是 `Result`。为简化示例，我们使用了 `unwrap`；在生产代码中，应使用 `panic!` 或 `expect`，提供更具体的出错信息。

现在已经有了把带注解的 Rust 代码从 `TokenStream` 转换为 `DeriveInput` 实例的代码，下面来生成在带注解类型上实现 `HelloMacro` trait 的代码，如示例 20-42 所示。

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="使用解析后的 Rust 代码实现 `HelloMacro` trait">

```rust,ignore
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

我们使用 `ast.ident` 取得包含带注解类型名称（标识符）的 `Ident` 结构体实例。示例 20-41 中的结构体表明，对示例 20-37 的代码运行 `impl_hello_macro` 函数时，取得的 `ident` 会有一个值为 `"Pancakes"` 的 `ident` 字段。因此，示例 20-42 中的变量 `name` 会包含一个 `Ident` 结构体实例，打印时会得到字符串 `"Pancakes"`，也就是示例 20-37 中结构体的名称。

`quote!` 宏让我们能够定义想要返回的 Rust 代码。编译器期望的内容不同于直接执行 `quote!` 宏的结果，因此需要将它转换为 `TokenStream`。我们通过调用 `into` 方法完成转换；该方法会消费这个中间表示，并返回所需 `TokenStream` 类型的值。

`quote!` 宏还提供了一些非常巧妙的模板机制：可以输入 `#name`，`quote!` 会用变量 `name` 中的值替换它。甚至可以执行一些类似普通宏的重复操作。请查阅 [`quote` crate 的文档][quote-docs]获取完整介绍。

我们希望过程宏为用户标注的类型生成 `HelloMacro` trait 实现，可以通过 `#name` 取得该类型。trait 实现有一个函数 `hello_macro`，其主体包含我们想提供的功能：打印 `Hello, Macro! My name is`，然后打印带注解类型的名称。

这里使用的 `stringify!` 宏内置于 Rust。它接收一个 Rust 表达式（例如 `1 + 2`），并在编译时把表达式转换为字符串字面量（例如 `"1 + 2"`）。这与 `format!` 或 `println!` 不同；后两者会先对表达式求值，再把结果转换为 `String`。输入 `#name` 有可能是应按字面形式打印的表达式，所以我们使用 `stringify!`。在编译时把 `#name` 转换为字符串字面量，也让 `stringify!` 省去了一次分配。

此时，在 `hello_macro` 和 `hello_macro_derive` 中执行 `cargo build` 都应成功完成。让我们把这些 crate 接入示例 20-37 的代码，看看过程宏的实际效果！在 <em>projects</em> 目录中用 `cargo new pancakes` 创建一个新的二进制项目。需要在 `pancakes` crate 的 <em>Cargo.toml</em> 中把 `hello_macro` 和 `hello_macro_derive` 添加为依赖。如果要把自己版本的 `hello_macro` 和 `hello_macro_derive` 发布到 [crates.io](https://crates.io/)<!-- ignore -->，它们会是普通依赖；否则，可以像下面这样把它们指定为 `path` 依赖：

```toml
{{#include ../../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

把示例 20-37 中的代码放入 <em>src/main.rs</em>，然后运行 `cargo run`：它应打印 `Hello, Macro! My name is Pancakes!`。过程宏生成的 `HelloMacro` trait 实现已经包含进来，无需 `pancakes` crate 自行实现；`#[derive(HelloMacro)]` 添加了 trait 实现。

接下来，让我们探索其他类型的过程宏与自定义 `derive` 宏有何不同。

### 类属性宏

<em>类属性宏(attribute-like macro)</em>与自定义 `derive` 宏相似，但它不是为 `derive` 属性生成代码，而是允许你创建新属性。它也更灵活：`derive` 只适用于结构体和枚举；属性还可以应用于函数等其他项。下面是使用类属性宏的例子。假设使用 Web 应用框架时，有一个名为 `route` 的属性用于标注函数：

```rust,ignore
#[route(GET, "/")]
fn index() {
```

这个 `#[route]` 属性将由框架定义为过程宏。宏定义函数的签名如下：

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

这里有两个 `TokenStream` 类型的形参。第一个用于属性内容，即 `GET, "/"` 部分。第二个是属性所附加项的主体；在这里，就是 `fn index() {}` 以及函数体的其余部分。

除此之外，类属性宏的工作方式与自定义 `derive` 宏相同：创建一个 crate 类型为 `proc-macro` 的 crate，并实现一个生成所需代码的函数！

### 类函数宏

<em>类函数宏(function-like macro)</em>定义看起来像函数调用的宏。与 `macro_rules!` 宏类似，它们比函数更灵活；例如，它们可以接收数量未知的实参。不过，`macro_rules!` 宏只能使用前面[“用于通用元编程的声明宏”][decl]<!-- ignore -->一节讨论的类 `match` 语法来定义。类函数宏接收一个 `TokenStream` 形参，其定义像另外两种过程宏一样，使用 Rust 代码操作该 `TokenStream`。类函数宏的一个例子是 `sql!` 宏，可以像下面这样调用：

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

这个宏会解析其中的 SQL 语句并检查语法是否正确，这项处理远比 `macro_rules!` 宏所能完成的复杂。`sql!` 宏的定义如下：

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

该定义与自定义 `derive` 宏的签名相似：接收圆括号内的 token，并返回想要生成的代码。

## 小结

呼！现在你的工具箱里已经有了一些可能不常使用的 Rust 功能，但你知道它们能用于非常具体的场景。我们介绍了若干复杂主题，这样当你在错误消息建议或他人代码中遇到它们时，就能够识别这些概念和语法。请把本章作为参考，用它指导你找到解决方案。

接下来，我们会把全书讨论的所有内容付诸实践，再完成一个项目！

[ref]: https://doc.rust-lang.org/reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
