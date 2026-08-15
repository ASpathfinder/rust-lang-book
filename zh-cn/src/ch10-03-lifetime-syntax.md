<a id="validating-references-with-lifetimes"></a>

## 使用生命周期验证引用

生命周期是我们已经用过的另一种泛型。它不是确保类型具有我们想要的行为，而是确保引用在我们需要它们期间一直有效。

第 4 章[“引用与借用”][references-and-borrowing]一节中有一个细节尚未讨论：Rust 中的每个引用都有<em>生命周期(lifetime)</em>，即该引用有效的作用域。大多数情况下，生命周期会像类型一样被隐式推断。只有当多种类型都有可能时，我们才需要标注类型。类似地，当引用的生命周期可能存在多种不同关系时，必须标注生命周期。Rust 要求我们使用泛型生命周期形参标注这些关系，以确保运行时实际使用的引用一定有效。

标注生命周期甚至是大多数其他编程语言中都不存在的概念，因此可能会让人感到陌生。虽然本章不会涵盖生命周期的全部内容，但会讨论你可能遇到生命周期语法的常见方式，帮助你熟悉这个概念。

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-dangling-references-with-lifetimes"></a>

### 悬垂引用

生命周期的主要目标是防止悬垂引用；如果允许悬垂引用存在，程序就会引用预期数据之外的数据。考虑示例 10-16 中的程序，它包含一个外部作用域和一个内部作用域。

<Listing number="10-16" caption="尝试使用其所指值已经离开作用域的引用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> 注意：示例 10-16、10-17 和 10-23 声明变量时没有赋予初始值，因此变量名存在于外部作用域中。乍看之下，这似乎与 Rust 没有空值相矛盾。然而，如果尝试在赋值之前使用变量，就会得到编译时错误，这表明 Rust 的确不允许空值。

外部作用域声明了一个没有初始值、名为 `r` 的变量；内部作用域声明了一个初始值为 `5`、名为 `x` 的变量。在内部作用域中，我们尝试把 `r` 的值设为对 `x` 的引用。随后内部作用域结束，我们尝试打印 `r` 中的值。这段代码无法编译，因为在尝试使用 `r` 之前，`r` 所引用的值已经离开作用域。错误信息如下：

```console
{{#include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

错误信息指出变量 `x`“存活得不够久”。这是因为内部作用域在第 7 行结束时，`x` 会离开作用域。但 `r` 在外部作用域中仍然有效；由于其作用域更大，我们说它“存活得更久”。如果 Rust 允许这段代码工作，`r` 就会引用 `x` 离开作用域时已经释放的内存，对 `r` 的任何操作都无法正常工作。那么，Rust 如何判定这段代码无效呢？它使用借用检查器。

### 借用检查器

Rust 编译器带有<em>借用检查器(borrow checker)</em>，它会比较作用域，判断所有借用是否有效。示例 10-17 展示了与示例 10-16 相同的代码，但添加了标明变量生命周期的注释。

<Listing number="10-17" caption="标注 `r` 和 `x` 的生命周期，分别命名为 `'a` 和 `'b`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

这里，我们使用 `'a` 标注 `r` 的生命周期，使用 `'b` 标注 `x` 的生命周期。如你所见，内部的 `'b` 块远小于外部的 `'a` 生命周期块。编译时，Rust 会比较两个生命周期的大小，发现 `r` 的生命周期是 `'a`，但它引用的内存生命周期是 `'b`。程序被拒绝，是因为 `'b` 比 `'a` 短：引用的对象没有引用本身存活得久。

示例 10-18 修复了代码，使其不再包含悬垂引用，并且可以顺利编译。

<Listing number="10-18" caption="有效的引用，因为数据的生命周期比引用更长">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

这里，`x` 的生命周期是 `'b`，在此情形中比 `'a` 更大。这意味着 `r` 可以引用 `x`，因为 Rust 知道只要 `r` 中的引用有效，`x` 就始终有效。

现在你已经知道引用的生命周期在哪里，以及 Rust 如何分析生命周期以确保引用始终有效，接下来让我们探索函数形参和返回值中的泛型生命周期。

### 函数中的泛型生命周期

我们将编写一个返回两个字符串切片中较长者的函数。该函数接收两个字符串切片，返回一个字符串切片。实现 `longest` 函数后，示例 10-19 中的代码应该打印 `The longest string is abcd`。

<Listing number="10-19" file-name="src/main.rs" caption="调用 `longest` 函数找出两个字符串切片中较长者的 `main` 函数">

```rust,ignore
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

请注意，我们希望函数接收作为引用的字符串切片，而不是字符串，因为不希望 `longest` 函数取得形参的所有权。有关为何示例 10-19 使用这些形参的更多讨论，请参阅第 4 章[“作为形参的字符串切片”][string-slices-as-parameters]。

如果尝试按示例 10-20 所示实现 `longest` 函数，代码将无法编译。

<Listing number="10-20" file-name="src/main.rs" caption="返回两个字符串切片中较长者、但尚不能编译的 `longest` 函数实现">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

相反，我们会得到以下有关生命周期的错误：

```console
{{#include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

帮助文本显示，返回类型需要泛型生命周期形参，因为 Rust 无法判断返回的引用指向 `x` 还是 `y`。事实上，我们也不知道，因为这个函数体的 `if` 块返回对 `x` 的引用，而 `else` 块返回对 `y` 的引用！

定义这个函数时，我们不知道会传入哪些具体值，所以不知道会执行 `if` 还是 `else` 分支。我们也不知道传入引用的具体生命周期，因此无法像示例 10-17 和 10-18 那样查看作用域，判断返回的引用是否始终有效。借用检查器同样无法判断，因为它不知道 `x` 和 `y` 的生命周期与返回值生命周期之间的关系。为了修复错误，我们会添加定义引用之间关系的泛型生命周期形参，让借用检查器可以执行分析。

### 生命周期标注语法

<em>生命周期标注(lifetime annotation)</em>不会改变任何引用的存活时长。相反，它们描述多个引用的生命周期之间的关系，而不影响生命周期本身。正如签名指定泛型类型形参时函数可以接受任何类型一样，函数也可以通过指定泛型生命周期形参，接受具有任意生命周期的引用。

生命周期标注的语法有些特别：生命周期形参名必须以撇号（`'`）开头，并且通常全部小写、非常简短，就像泛型类型一样。大多数人使用名称 `'a` 作为第一个生命周期标注。我们把生命周期形参标注放在引用的 `&` 之后，并用空格将标注与引用的类型分隔开。

下面是几个例子：不带生命周期形参的 `i32` 引用、带有名为 `'a` 的生命周期形参的 `i32` 引用，以及同样具有生命周期 `'a` 的 `i32` 可变引用：

```rust,ignore
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

单独一个生命周期标注没有太大意义，因为标注旨在告诉 Rust 多个引用的泛型生命周期形参之间有何关系。让我们在 `longest` 函数的上下文中考察生命周期标注之间的关系。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-function-signatures"></a>

### 在函数签名中

要在函数签名中使用生命周期标注，需要像泛型类型形参一样，在函数名与形参列表之间的尖括号内声明泛型生命周期形参。

我们希望签名表达以下约束：只要两个形参都有效，返回的引用就有效。这就是形参生命周期与返回值之间的关系。我们把生命周期命名为 `'a`，然后将其添加到每个引用中，如示例 10-21 所示。

<Listing number="10-21" file-name="src/main.rs" caption="`longest` 函数定义，指定签名中的所有引用必须具有相同的生命周期 `'a`">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

把这段代码与示例 10-19 中的 `main` 函数配合使用时，应该能够编译并产生我们想要的结果。

现在，函数签名告诉 Rust：对于某个生命周期 `'a`，函数接收两个形参，它们都是至少存活 `'a` 这么久的字符串切片。函数签名还告诉 Rust，函数返回的字符串切片也至少存活 `'a` 这么久。实际上，这意味着 `longest` 函数所返回引用的生命周期，与函数实参所引用值的生命周期中较短的那个相同。我们希望 Rust 在分析代码时使用的正是这些关系。

请记住，在这个函数签名中指定生命周期形参时，并没有改变任何传入或返回值的生命周期。相反，我们指定借用检查器应拒绝不符合这些约束的所有值。请注意，`longest` 函数不需要确切知道 `x` 和 `y` 会存活多久，只需要有某个能够替代 `'a` 并满足此签名的作用域。

在函数中标注生命周期时，标注位于函数签名，而不是函数体中。生命周期标注会像签名中的类型一样，成为函数契约的一部分。让函数签名包含生命周期契约，可以简化 Rust 编译器的分析。如果函数的标注方式或调用方式存在问题，编译器错误就能更准确地指出代码相关部分和约束。相反，如果 Rust 编译器对我们预期的生命周期关系做更多推断，编译器可能只能指向距离问题根源许多步骤之外的代码使用处。

向 `longest` 传入具体引用时，用来替代 `'a` 的具体生命周期，是 `x` 作用域与 `y` 作用域重叠的部分。换句话说，泛型生命周期 `'a` 会取得等于 `x` 和 `y` 生命周期中较短者的具体生命周期。由于我们使用相同的生命周期形参 `'a` 标注了返回的引用，返回的引用也会在 `x` 和 `y` 生命周期中较短者的时长内有效。

让我们传入具有不同具体生命周期的引用，看看生命周期标注如何限制 `longest` 函数。示例 10-22 是一个直观的例子。

<Listing number="10-22" file-name="src/main.rs" caption="使用指向具有不同具体生命周期的 `String` 值的引用调用 `longest` 函数">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

在这个例子中，`string1` 在外部作用域结束之前有效，`string2` 在内部作用域结束之前有效，`result` 引用的内容在内部作用域结束之前有效。运行这段代码会发现借用检查器予以通过：代码可以编译并打印 `The longest string is long string is long`。

接下来，尝试一个表明 `result` 中引用的生命周期必须是两个实参生命周期中较短者的例子。我们把 `result` 变量的声明移到内部作用域之外，但仍在包含 `string2` 的作用域内为 `result` 赋值。然后，把使用 `result` 的 `println!` 移到内部作用域之外，也就是内部作用域已经结束之后。示例 10-23 中的代码无法编译。

<Listing number="10-23" file-name="src/main.rs" caption="尝试在 `string2` 离开作用域后使用 `result`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

尝试编译这段代码时，会得到以下错误：

```console
{{#include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

错误表明，为了让 `result` 对 `println!` 语句有效，`string2` 必须一直有效到外部作用域结束。Rust 之所以知道这一点，是因为我们使用相同的生命周期形参 `'a` 标注了函数形参和返回值的生命周期。

作为人类，我们可以查看代码并看出 `string1` 比 `string2` 长，因此 `result` 会包含对 `string1` 的引用。因为 `string1` 尚未离开作用域，对它的引用在 `println!` 语句处仍然有效。然而，编译器无法看出这个引用在此情形下有效。我们已经告诉 Rust，`longest` 函数所返回引用的生命周期，与传入引用的生命周期中较短者相同。因此，借用检查器不允许示例 10-23 中可能含有无效引用的代码。

请尝试设计更多实验，改变传给 `longest` 函数的引用值和生命周期，以及返回引用的使用方式。编译前先假设实验能否通过借用检查器，再检查自己的判断是否正确！

<!-- Old headings. Do not remove or links may break. -->

<a id="thinking-in-terms-of-lifetimes"></a>

### 关系

应该如何指定生命周期形参，取决于函数所做的事情。例如，如果把 `longest` 函数的实现改为始终返回第一个形参，而不是较长的字符串切片，就不需要为 `y` 形参指定生命周期。以下代码可以编译：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

我们为形参 `x` 和返回类型指定了生命周期形参 `'a`，但没有为形参 `y` 指定，因为 `y` 的生命周期与 `x` 或返回值的生命周期不存在任何关系。

从函数返回引用时，返回类型的生命周期形参需要与其中一个形参的生命周期形参匹配。如果返回的引用<em>并不</em>指向某个形参，它就必须指向在这个函数内部创建的值。然而，这会成为悬垂引用，因为函数结束时该值会离开作用域。考虑下面这个无法编译的 `longest` 函数实现尝试：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

这里，虽然为返回类型指定了生命周期形参 `'a`，但这个实现仍无法编译，因为返回值的生命周期与形参的生命周期完全无关。得到的错误信息如下：

```console
{{#include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

问题在于，`result` 在 `longest` 函数结束时离开作用域并被清理，而我们还试图从函数返回对 `result` 的引用。没有任何生命周期形参能够改变这个悬垂引用，Rust 也不会允许创建悬垂引用。在这种情况下，最好的修复方式是返回拥有所有权的数据类型，而不是引用，这样调用函数便负责清理该值。

归根结底，生命周期语法用于连接函数的各种形参和返回值的生命周期。它们连接起来后，Rust 就有足够的信息允许内存安全的操作，并禁止会创建悬垂指针或以其他方式违反内存安全的操作。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-struct-definitions"></a>

### 在结构体定义中

到目前为止，我们定义的结构体都存放拥有所有权的类型。也可以定义存放引用的结构体，但在这种情况下，需要为结构体定义中的每个引用添加生命周期标注。示例 10-24 中名为 `ImportantExcerpt` 的结构体存放一个字符串切片。

<Listing number="10-24" file-name="src/main.rs" caption="存放引用、因此需要生命周期标注的结构体">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

这个结构体只有一个字段 `part`，用于存放字符串切片，也就是引用。与泛型数据类型一样，我们在结构体名之后的尖括号内声明泛型生命周期形参名，以便在结构体定义的函数体中使用生命周期形参。这个标注意味着，`ImportantExcerpt` 实例不能比其 `part` 字段中存放的引用活得更久。

这里的 `main` 函数创建了一个 `ImportantExcerpt` 结构体实例，其中存放了对变量 `novel` 所拥有 `String` 第一句话的引用。`novel` 中的数据在创建 `ImportantExcerpt` 实例之前就已存在。此外，直到 `ImportantExcerpt` 离开作用域之后，`novel` 才会离开作用域，所以 `ImportantExcerpt` 实例中的引用有效。

### 生命周期省略

你已经知道每个引用都有生命周期，也知道需要为使用引用的函数或结构体指定生命周期形参。然而，我们在示例 4-9 中曾有一个函数，它没有生命周期标注却能编译；示例 10-25 再次展示了它。

<Listing number="10-25" file-name="src/lib.rs" caption="示例 4-9 中定义的函数；尽管形参和返回类型都是引用，它仍然无需生命周期标注即可编译">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

这个函数无需生命周期标注就能编译，其原因与历史有关：在 Rust 的早期版本（1.0 之前）中，这段代码无法编译，因为每个引用都需要显式生命周期。当时，函数签名必须写成：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

编写了大量 Rust 代码后，Rust 团队发现，Rust 程序员会在某些特定情形下一再输入相同的生命周期标注。这些情形是可预测的，并遵循少数几个确定的模式。开发者把这些模式编入编译器，让借用检查器可以在这些情形下推断生命周期，不再需要显式标注。

这段 Rust 历史与我们有关，因为未来可能会出现更多确定模式并被加入编译器。到那时，需要的生命周期标注可能会更少。

编入 Rust 引用分析中的这些模式称为<em>生命周期省略规则(lifetime elision rule)</em>。它们不是程序员需要遵守的规则，而是编译器会考虑的一组特定情形；如果代码符合这些情形，就无需显式写出生命周期。

省略规则并不会提供完整推断。如果 Rust 应用规则后，引用的生命周期仍然存在歧义，编译器不会猜测剩余引用应具有什么生命周期。编译器会给出错误，而你可以通过添加生命周期标注来解决。

函数或方法形参上的生命周期称为<em>输入生命周期(input lifetime)</em>，返回值上的生命周期称为<em>输出生命周期(output lifetime)</em>。

没有显式标注时，编译器使用三条规则推断引用的生命周期。第一条规则适用于输入生命周期，第二条和第三条适用于输出生命周期。如果编译器应用完三条规则后，仍有引用的生命周期无法推断，就会停止并报错。这些规则既适用于 `fn` 定义，也适用于 `impl` 块。

第一条规则是：编译器为每个引用形参分配一个生命周期形参。换句话说，只有一个形参的函数获得一个生命周期形参：`fn foo<'a>(x: &'a i32)`；有两个形参的函数获得两个不同的生命周期形参：`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`；依此类推。

第二条规则是：如果恰好只有一个输入生命周期形参，该生命周期会被分配给所有输出生命周期形参：`fn foo<'a>(x: &'a i32) -> &'a i32`。

第三条规则是：如果有多个输入生命周期形参，但其中一个是 `&self` 或 `&mut self`（因为这是方法），则 `self` 的生命周期会被分配给所有输出生命周期形参。这条规则减少了所需符号，让方法更容易读写。

让我们假装自己是编译器，应用这些规则来推断示例 10-25 中 `first_word` 函数签名内引用的生命周期。签名最初没有任何与引用关联的生命周期：

```rust,ignore
fn first_word(s: &str) -> &str {
```

接着，编译器应用第一条规则，也就是每个形参都有自己的生命周期。照例将其称为 `'a`，现在签名变成：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

由于恰好只有一个输入生命周期，第二条规则适用。第二条规则把唯一输入形参的生命周期分配给输出生命周期，所以签名现在变成：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

现在，这个函数签名中的所有引用都有了生命周期，编译器无需程序员在函数签名中标注生命周期，即可继续分析。

再看一个例子，这次使用刚开始在示例 10-20 中处理时不带生命周期形参的 `longest` 函数：

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

应用第一条规则：每个形参取得自己的生命周期。这次有两个形参而不是一个，所以有两个生命周期：

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

可以看出，第二条规则不适用，因为存在多个输入生命周期。第三条规则也不适用，因为 `longest` 是函数而不是方法，所以所有形参都不是 `self`。应用完三条规则后，仍未推断出返回类型的生命周期。这正是尝试编译示例 10-20 中的代码时出错的原因：编译器应用了生命周期省略规则，但仍无法推断签名中所有引用的生命周期。

由于第三条规则实际上只适用于方法签名，接下来会在该语境中考察生命周期，看看为什么第三条规则使我们不必频繁在方法签名中标注生命周期。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-method-definitions"></a>

### 在方法定义中

为带生命周期的结构体实现方法时，使用的语法与示例 10-11 中的泛型类型形参相同。生命周期形参应在哪里声明和使用，取决于它们是与结构体字段相关，还是与方法形参和返回值相关。

结构体字段的生命周期名称始终需要在 `impl` 关键字之后声明，并在结构体名称之后使用，因为这些生命周期是结构体类型的一部分。

在 `impl` 块中的方法签名内，引用可能与结构体字段中引用的生命周期相关，也可能相互独立。此外，生命周期省略规则往往使方法签名不再需要生命周期标注。让我们使用示例 10-24 中定义的 `ImportantExcerpt` 结构体来看几个例子。

首先，使用名为 `level` 的方法。它唯一的形参是对 `self` 的引用，返回值是 `i32`，并不是对任何内容的引用：

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

`impl` 之后的生命周期形参声明及其在类型名称之后的使用是必需的，但由于第一条省略规则，不需要标注对 `self` 的引用的生命周期。

下面是应用第三条生命周期省略规则的例子：

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

这里有两个输入生命周期，因此 Rust 应用第一条生命周期省略规则，分别为 `&self` 和 `announcement` 分配生命周期。然后，由于其中一个形参是 `&self`，返回类型取得 `&self` 的生命周期，所有生命周期都得到了确定。

### 静态生命周期

我们还需要讨论一个特殊生命周期：`'static`。它表示受影响的引用<em>可以</em>存活整个程序运行期间。所有字符串字面量都具有 `'static` 生命周期，可以像下面这样标注：

```rust
let s: &'static str = "I have a static lifetime.";
```

这个字符串的文本直接存储在程序的二进制文件中，始终可用。因此，所有字符串字面量的生命周期都是 `'static`。

你可能会在错误信息中看到使用 `'static` 生命周期的建议。但在把引用的生命周期指定为 `'static` 之前，请考虑该引用是否真的存活于程序的整个生命周期，以及你是否希望如此。大多数时候，建议使用 `'static` 生命周期的错误信息源于尝试创建悬垂引用，或可用生命周期不匹配。在这些情况下，解决方案是修复这些问题，而不是指定 `'static` 生命周期。

<!-- Old headings. Do not remove or links may break. -->

<a id="generic-type-parameters-trait-bounds-and-lifetimes-together"></a>

## 泛型类型形参、特征约束与生命周期

让我们简要看看如何在一个函数中同时指定泛型类型形参、特征约束和生命周期！

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

这是示例 10-21 中返回两个字符串切片中较长者的 `longest` 函数。不过，它现在多了一个名为 `ann`、泛型类型为 `T` 的形参；按照 `where` 子句的指定，任何实现了 `Display` 特征的类型都可以填充 `T`。这个额外形参会使用 `{}` 打印，因此需要 `Display` 特征约束。由于生命周期是一种泛型，生命周期形参 `'a` 与泛型类型形参 `T` 的声明会出现在同一列表中，位于函数名之后的尖括号内。

## 总结

本章涵盖了大量内容！现在你已经了解泛型类型形参、特征和特征约束，以及泛型生命周期形参，可以编写适用于许多不同情形且没有重复的代码了。泛型类型形参让代码可以应用于不同类型。特征和特征约束确保即使类型是泛型的，它们仍具有代码所需的行为。你还学会了使用生命周期标注，确保灵活的代码不会含有悬垂引用。而所有这些分析都发生在编译时，不会影响运行时性能！

信不信由你，本章讨论的主题还有更多内容值得学习：第 18 章会讨论特征对象，它是使用特征的另一种方式。还有一些涉及生命周期标注的复杂情形，只有在非常高级的场景中才会用到；对于这些内容，请阅读 [Rust 参考手册][reference]。不过接下来，你会学习如何在 Rust 中编写测试，确保代码按预期工作。

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: https://doc.rust-lang.org/reference/trait-bounds.html
