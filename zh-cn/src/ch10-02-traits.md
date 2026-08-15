<!-- Old headings. Do not remove or links may break. -->

<a id="traits-defining-shared-behavior"></a>

## 使用特征定义共同行为

<em>特征(trait)</em>定义了某个特定类型拥有、并且可以与其他类型共享的功能。我们可以使用特征以抽象方式定义共同行为，也可以使用<em>特征约束(trait bound)</em>指定泛型类型可以是任何具有特定行为的类型。

> 注意：特征与其他语言中通常称为<em>接口(interface)</em>的功能类似，但存在一些差异。

### 定义特征

类型的行为由我们可以在该类型上调用的方法组成。如果能对不同类型调用相同的方法，这些类型就具有相同的行为。特征定义是一种把方法签名组合在一起的方式，用于定义达成某项目的所需的一组行为。

例如，假设有多个结构体，存放不同种类、不同数量的文本：`NewsArticle` 结构体存放在某个地点提交的新闻报道；`SocialPost` 最多可以包含 280 个字符，还附带表明它是新帖子、转帖还是对其他帖子的回复的元数据。

我们希望创建名为 `aggregator` 的媒体聚合器库 crate，用于显示可能存储在 `NewsArticle` 或 `SocialPost` 实例中的数据摘要。为此，需要从每种类型取得摘要；我们会通过对实例调用 `summarize` 方法来请求摘要。示例 10-12 展示了表达这种行为的公有 `Summary` 特征的定义。

<Listing number="10-12" file-name="src/lib.rs" caption="由 `summarize` 方法提供的行为组成的 `Summary` 特征">

```rust,noplayground
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

这里，我们使用 `trait` 关键字，后跟特征名称（本例中是 `Summary`）来声明特征。我们还把特征声明为 `pub`，这样依赖此 crate 的其他 crate 也能使用该特征，后面的几个例子会展示这一点。在花括号内，我们声明描述实现该特征的类型行为的方法签名，这里是 `fn summarize(&self) -> String`。

方法签名之后没有在花括号内提供实现，而是使用分号。实现这个特征的每种类型，都必须为方法体提供自己的自定义行为。编译器会强制确保任何具有 `Summary` 特征的类型都使用完全一致的签名定义了 `summarize` 方法。

一个特征的函数体中可以有多个方法：方法签名每行列出一个，并以分号结尾。

<a id="implementing-a-trait-on-a-type"></a>

### 为类型实现特征

既然已经定义了 `Summary` 特征所需的方法签名，就可以为媒体聚合器中的类型实现它。示例 10-13 展示了为 `NewsArticle` 结构体实现 `Summary` 特征的代码，它使用标题、作者和地点创建 `summarize` 的返回值。对于 `SocialPost` 结构体，我们把 `summarize` 定义为用户名后跟帖子的完整文本，并假定帖子内容已经限制在 280 个字符以内。

<Listing number="10-13" file-name="src/lib.rs" caption="为 `NewsArticle` 和 `SocialPost` 类型实现 `Summary` 特征">

```rust,noplayground
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

为类型实现特征与实现普通方法类似。区别在于，`impl` 之后要写希望实现的特征名，再使用 `for` 关键字，然后指定要为其实现特征的类型名称。在 `impl` 块内，放入特征定义所声明的方法签名。每个签名之后不再添加分号，而是使用花括号，并在方法体中填入希望特征方法针对此类型具备的具体行为。

现在，库已经为 `NewsArticle` 和 `SocialPost` 实现了 `Summary` 特征，crate 的用户就可以像调用普通方法一样，对 `NewsArticle` 和 `SocialPost` 实例调用特征方法。唯一的区别是，用户除了要把类型引入作用域，还必须把特征也引入作用域。下面是二进制 crate 使用 `aggregator` 库 crate 的示例：

```rust,ignore
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

这段代码会打印 `1 new post: horse_ebooks: of course, as you probably already know, people`。

依赖 `aggregator` crate 的其他 crate 也可以把 `Summary` 特征引入作用域，为自己的类型实现 `Summary`。需要注意一项限制：只有当特征或类型中的至少一方位于当前 crate 中时，我们才能为该类型实现该特征。例如，可以在 `aggregator` crate 中为 `SocialPost` 这样的自定义类型实现 `Display` 等标准库特征，因为 `SocialPost` 类型位于当前 `aggregator` crate 中。也可以在 `aggregator` crate 中为 `Vec<T>` 实现 `Summary`，因为 `Summary` 特征位于当前 `aggregator` crate 中。

但是，不能为外部类型实现外部特征。例如，不能在 `aggregator` crate 中为 `Vec<T>` 实现 `Display` 特征，因为 `Display` 和 `Vec<T>` 都定义在标准库中，不位于当前 `aggregator` crate。这项限制属于一种称为<em>一致性(coherence)</em>的属性，更具体地说，是<em>孤儿规则(orphan rule)</em>；之所以这样命名，是因为父类型并不存在于当前 crate。该规则确保他人的代码不会破坏你的代码，反之亦然。如果没有这项规则，两个 crate 可能为同一类型实现同一特征，而 Rust 不知道该使用哪个实现。

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### 使用默认实现

有时，为特征中的部分或全部方法提供默认行为很有用，而不必要求每种类型都实现所有方法。这样，为某个特定类型实现特征时，就可以保留或覆盖每个方法的默认行为。

在示例 10-14 中，我们为 `Summary` 特征的 `summarize` 方法指定了默认字符串，而不像示例 10-12 那样只定义方法签名。

<Listing number="10-14" file-name="src/lib.rs" caption="定义 `Summary` 特征，并为 `summarize` 方法提供默认实现">

```rust,noplayground
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

为了使用默认实现来概述 `NewsArticle` 实例，我们指定一个空 `impl` 块：`impl Summary for NewsArticle {}`。

尽管不再直接为 `NewsArticle` 定义 `summarize` 方法，但我们已经提供默认实现，并指定 `NewsArticle` 实现 `Summary` 特征。因此，仍然可以对 `NewsArticle` 实例调用 `summarize` 方法，如下所示：

```rust,ignore
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

这段代码会打印 `New article available! (Read more...)`。

创建默认实现不需要修改示例 10-13 中 `SocialPost` 的 `Summary` 实现。这是因为覆盖默认实现的语法，与实现没有默认实现的特征方法的语法相同。

默认实现可以调用同一特征中的其他方法，即使那些方法没有默认实现。这样，一个特征可以提供大量有用功能，而只要求实现者指定其中很小的一部分。例如，可以把 `Summary` 特征定义为包含必须实现的 `summarize_author` 方法，再定义一个具有默认实现、且会调用 `summarize_author` 方法的 `summarize` 方法：

```rust,noplayground
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

要使用这个版本的 `Summary`，为类型实现特征时只需定义 `summarize_author`：

```rust,ignore
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

定义 `summarize_author` 后，就可以对 `SocialPost` 结构体的实例调用 `summarize`，而 `summarize` 的默认实现会调用我们提供的 `summarize_author` 定义。由于我们已经实现 `summarize_author`，`Summary` 特征无需我们再编写任何代码，就为我们提供了 `summarize` 方法的行为。其用法如下：

```rust,ignore
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

这段代码会打印 `1 new post: (Read more from @horse_ebooks...)`。

请注意，在同一方法的覆盖实现中无法调用其默认实现。

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### 使用特征作为形参

既然知道如何定义和实现特征，我们就可以探索如何使用特征定义接受多种不同类型的函数。我们会使用示例 10-13 中为 `NewsArticle` 和 `SocialPost` 类型实现的 `Summary` 特征，定义一个 `notify` 函数。该函数会对其 `item` 形参调用 `summarize` 方法，而 `item` 属于某种实现了 `Summary` 特征的类型。为此，我们使用 `impl Trait` 语法：

```rust,ignore
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

我们没有为 `item` 形参指定具体类型，而是指定 `impl` 关键字和特征名称。这个形参接受任何实现了指定特征的类型。在 `notify` 的函数体中，可以对 `item` 调用来自 `Summary` 特征的任何方法，例如 `summarize`。我们可以调用 `notify` 并传入任意 `NewsArticle` 或 `SocialPost` 实例。使用 `String` 或 `i32` 等其他类型调用该函数的代码无法编译，因为这些类型没有实现 `Summary`。

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### 特征约束语法

`impl Trait` 语法适用于简单情形，但实际上是较长形式的语法糖，这种形式称为<em>特征约束(trait bound)</em>，写法如下：

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

这个较长形式与上一节中的例子等价，只是更加冗长。我们把特征约束放在泛型类型形参声明中，位于冒号之后、尖括号之内。

`impl Trait` 语法很方便，在简单情况下可以让代码更加简洁；完整的特征约束语法则可以在其他情况下表达更复杂的含义。例如，我们可以有两个实现 `Summary` 的形参。使用 `impl Trait` 语法时写作：

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

如果希望这个函数允许 `item1` 和 `item2` 具有不同类型（只要两种类型都实现 `Summary`），使用 `impl Trait` 很合适。但如果要强制两个形参具有相同类型，就必须使用特征约束：

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

把泛型类型 `T` 指定为 `item1` 和 `item2` 的类型，就会约束函数，使传给 `item1` 和 `item2` 的实参值必须具有相同的具体类型。

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>

#### 使用 `+` 语法指定多个特征约束

我们还可以指定多个特征约束。假设希望 `notify` 既对 `item` 使用显示格式，又调用 `summarize`：我们会在 `notify` 定义中指定 `item` 必须同时实现 `Display` 和 `Summary`。可以使用 `+` 语法做到这一点：

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

`+` 语法也适用于泛型类型上的特征约束：

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

指定这两个特征约束后，`notify` 的函数体就能调用 `summarize`，并使用 `{}` 格式化 `item`。

#### 使用 `where` 子句让特征约束更清晰

使用太多特征约束也有缺点。每个泛型都有自己的特征约束，因此具有多个泛型类型形参的函数，其函数名和形参列表之间可能包含大量特征约束信息，使函数签名难以阅读。为此，Rust 提供了另一种语法，在函数签名之后的 `where` 子句中指定特征约束。因此，不必写成：

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

而可以使用 `where` 子句：

```rust,ignore
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

这个函数的签名不再那么杂乱：函数名、形参列表和返回类型彼此相邻，与没有大量特征约束的函数类似。

### 返回实现了特征的类型

还可以在返回位置使用 `impl Trait` 语法，返回某种实现了特征的类型的值，如下所示：

```rust,ignore
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

使用 `impl Summary` 作为返回类型，可以指定 `returns_summarizable` 函数返回某种实现了 `Summary` 特征的类型，而不必说出具体类型。这里，`returns_summarizable` 返回 `SocialPost`，但调用此函数的代码无需知道这一点。

只通过类型所实现的特征指定返回类型，在第 13 章会介绍的闭包和迭代器语境中尤其有用。闭包和迭代器会创建只有编译器知道的类型，或者写出来非常长的类型。`impl Trait` 语法让你可以简洁地指定函数返回某种实现了 `Iterator` 特征的类型，而不必写出很长的类型。

不过，只有在返回单一类型时才能使用 `impl Trait`。例如，以下代码把返回类型指定为 `impl Summary`，但可能返回 `NewsArticle` 或 `SocialPost`，所以无法工作：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

由于编译器实现 `impl Trait` 语法的方式存在限制，不允许返回 `NewsArticle` 或 `SocialPost` 两者之一。第 18 章[“使用特征对象抽象共同行为”][trait-objects]一节会介绍如何编写具有这种行为的函数。

### 使用特征约束有条件地实现方法

通过在使用泛型类型形参的 `impl` 块上使用特征约束，可以有条件地为实现了指定特征的类型实现方法。例如，示例 10-15 中的 `Pair<T>` 类型始终实现 `new` 函数，以返回新的 `Pair<T>` 实例（回想第 5 章[“方法语法”][methods]一节，`Self` 是 `impl` 块所对应类型的类型别名，这里就是 `Pair<T>`）。但在下一个 `impl` 块中，只有内部类型 `T` 同时实现了支持比较的 `PartialOrd` 特征<em>以及</em>支持打印的 `Display` 特征，`Pair<T>` 才会实现 `cmp_display` 方法。

<Listing number="10-15" file-name="src/lib.rs" caption="根据特征约束有条件地为泛型类型实现方法">

```rust,noplayground
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

还可以有条件地为任何实现了另一个特征的类型实现某个特征。为满足特征约束的任意类型实现特征，称为<em>毯式实现(blanket implementation)</em>，Rust 标准库大量使用了这种实现。例如，标准库为任何实现了 `Display` 特征的类型实现 `ToString` 特征。标准库中的 `impl` 块与以下代码类似：

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

由于标准库提供了这个毯式实现，我们可以对任何实现了 `Display` 特征的类型调用 `ToString` 特征定义的 `to_string` 方法。例如，由于整数实现了 `Display`，可以像下面这样把整数转换成对应的 `String` 值：

```rust
let s = 3.to_string();
```

毯式实现会出现在特征文档的“Implementors”部分。

特征与特征约束让我们能够编写使用泛型类型形参来减少重复的代码，同时向编译器指定希望泛型类型具有特定行为。随后，编译器可以使用特征约束信息，检查代码中使用的所有具体类型是否提供了正确行为。在动态类型语言中，如果对没有定义某个方法的类型调用该方法，会在运行时得到错误。但 Rust 会把这些错误移到编译时，迫使我们在代码能够运行之前解决问题。此外，因为已经在编译时完成检查，无需编写在运行时检查行为的代码。这样既能提高性能，又无需放弃泛型的灵活性。

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax
