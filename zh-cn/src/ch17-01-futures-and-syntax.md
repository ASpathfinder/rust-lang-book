## Future 与 Async 语法

Rust 异步编程的关键要素是 <em>future</em>，以及 Rust 的 `async` 和 `await` 关键字。

<em>Future</em> 是一种现在可能尚未就绪、但会在未来某个时刻就绪的值。（许多语言中都有相同概念，有时采用 <em>task</em> 或 <em>promise</em> 等其他名称。）Rust 提供 `Future` trait 作为构建模块，使不同的异步操作能够用不同的数据结构实现，同时拥有共同接口。在 Rust 中，future 是实现 `Future` trait 的类型。每个 future 都保存自己的进度信息，以及“就绪”对它意味着什么。

可以把 `async` 关键字应用于代码块和函数，表明它们能够被中断并恢复。在 async 块或 async 函数中，可以使用 `await` 关键字来<em>等待 future(await a future)</em>，即等待它就绪。在 async 块或函数内，每个等待 future 的位置，都可能成为该代码块或函数暂停与恢复之处。检查 future 的值是否已可用的过程称为<em>轮询(polling)</em>。

C# 和 JavaScript 等其他语言也使用 `async` 和 `await` 关键字进行异步编程。如果你熟悉这些语言，可能会注意到 Rust 处理这些语法的方式存在一些显著差异。正如我们将看到的，这些差异有充分理由！

编写异步 Rust 时，大多数时候会使用 `async` 和 `await` 关键字。Rust 会把它们编译为使用 `Future` trait 的等价代码，就像把 `for` 循环编译为使用 `Iterator` trait 的等价代码。由于 Rust 提供了 `Future` trait，在需要时也能为自己的数据类型实现它。本章将看到的许多函数都会返回具有各自 `Future` 实现的类型。本章末尾会重新讨论该 trait 的定义，更深入地了解其工作方式；但这些细节目前已经足够我们继续前进。

这些内容可能还显得有些抽象，所以来编写第一个异步程序：一个小型 Web 抓取器。我们从命令行传入两个 URL，并发获取两者，并返回先完成者的结果。这个示例会包含不少新语法，但不用担心，我们会在过程中解释所需的一切。

<a id="our-first-async-program"></a>

## 第一个异步程序

为了让本章专注于学习 async，而不是忙于处理生态系统中的各个组成部分，我们创建了 `trpl` crate（`trpl` 是 “The Rust Programming Language” 的缩写）。它重新导出了所需的所有类型、trait 和函数，主要来自 [`futures`][futures-crate]<!-- ignore --> 和 [`tokio`][tokio]<!-- ignore --> crate。`futures` crate 是 Rust 官方用于异步代码实验的场所，`Future` trait 最初实际上就是在那里设计的。Tokio 是目前 Rust 中使用最广泛的异步运行时，尤其适用于 Web 应用程序。还有其他优秀的运行时，它们可能更适合你的用途。`trpl` 在底层使用 `tokio` crate，是因为后者经过充分测试且应用广泛。

在某些情况下，`trpl` 还会重命名或包装原始 API，让你专注于与本章有关的细节。如果希望了解这个 crate 的作用，建议查看[其源代码][crate-source]。你可以看到每一项重新导出来自哪个 crate；我们还留下了大量注释，解释这个 crate 所做的事情。

创建一个名为 `hello-async` 的新二进制项目，并把 `trpl` crate 添加为依赖：

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

现在，可以使用 `trpl` 提供的各个部分编写第一个异步程序。我们将构建一个小型命令行工具：获取两个网页，提取各自的 `<title>` 元素，并打印率先完成整个过程的页面标题。

### 定义 page_title 函数

首先编写一个函数，它接收页面 URL 作为形参，向该 URL 发出请求，并返回 `<title>` 元素的文本（见示例 17-1）。

<Listing number="17-1" file-name="src/main.rs" caption="定义一个异步函数，从 HTML 页面获取 title 元素">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

首先定义名为 `page_title` 的函数，并用 `async` 关键字标记它。接着使用 `trpl::get` 函数获取传入的任意 URL，并添加 `await` 关键字等待响应。为了取得 `response` 的文本，我们调用它的 `text` 方法，并再次用 `await` 关键字等待。这两个步骤都是异步的。对于 `get` 函数，需要等待服务器发回响应的第一部分，其中包含 HTTP 标头、cookie 等内容，并且可能与响应体分开发送。尤其当响应体很大时，全部内容到达可能需要一些时间。由于必须等待<em>完整</em>响应到达，`text` 方法也是异步的。

必须显式等待这两个 future，因为 Rust 中的 future 是<em>惰性(lazy)</em>的：除非使用 `await` 关键字要求它们工作，否则它们什么也不会做。（事实上，如果不使用一个 future，Rust 会显示编译器警告。）这可能让你想起第 13 章[“使用迭代器处理一系列项”][iterators-lazy]<!-- ignore -->一节对迭代器的讨论。除非调用迭代器的 `next` 方法（直接调用，或通过 `for` 循环、`map` 等在底层使用 `next` 的方法），否则迭代器什么也不做。同样，除非显式要求 future 工作，否则它们也什么都不做。这种惰性使 Rust 能够避免在确实需要之前运行异步代码。

> 注意：这不同于第 16 章[“使用 spawn 创建新线程”][thread-spawn]<!-- ignore -->一节中使用 `thread::spawn` 时看到的行为；在那里，传给另一个线程的闭包会立即开始运行。它也不同于许多其他语言处理 async 的方式。不过，这一点对于 Rust 提供性能保证至关重要，就像迭代器的惰性一样。

得到 `response_text` 后，可以使用 `Html::parse` 把它解析为 `Html` 类型的实例。现在得到的不再是原始字符串，而是一种可用于把 HTML 当作更丰富数据结构处理的数据类型。具体来说，可以使用 `select_first` 方法查找给定 CSS 选择器的第一个实例。传入字符串 `"title"`，便会得到文档中的第一个 `<title>` 元素（如果存在的话）。由于可能没有任何匹配元素，`select_first` 返回 `Option<ElementRef>`。最后，使用 `Option::map` 方法；如果 `Option` 中存在项，它让我们能够处理该项，否则什么也不做。（这里也可以使用 `match` 表达式，但 `map` 更符合惯用写法。）在提供给 `map` 的函数体中，对 `title` 调用 `inner_html` 取得其内容，即一个 `String`。全部完成后，得到 `Option<String>`。

请注意，Rust 的 `await` 关键字位于所等待表达式的<em>后面</em>，而不是前面。也就是说，它是一个<em>后缀(postfix)</em>关键字。如果你用过其他语言中的 `async`，这可能与习惯不同，但在 Rust 中，它让方法链使用起来更加方便。因此，可以修改 `page_title` 的函数体，把 `trpl::get` 和 `text` 函数调用串联起来，并在两者之间使用 `await`，如示例 17-2 所示。

<Listing number="17-2" file-name="src/main.rs" caption="使用 `await` 关键字进行链式调用">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

至此，我们成功编写了第一个异步函数！在向 `main` 添加代码来调用它之前，先进一步讨论刚才编写的内容及其含义。

Rust 看到用 `async` 关键字标记的<em>代码块</em>时，会把它编译为一种实现 `Future` trait、独一无二的匿名数据类型。Rust 看到用 `async` 标记的<em>函数</em>时，会把它编译为一个非异步函数，其函数体是一个 async 块。异步函数的返回类型，是编译器为该 async 块创建的匿名数据类型。

因此，编写 `async fn` 等价于编写一个返回其返回类型之 <em>future</em> 的函数。对编译器而言，示例 17-1 中 `async fn page_title` 这样的函数定义，大致等价于以下非异步函数：

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

逐一分析转换后版本的各个部分：

- 它使用了第 10 章[“作为形参的 Trait”][impl-trait]<!-- ignore -->一节讨论过的 `impl Trait` 语法。
- 返回值实现 `Future` trait，其关联类型为 `Output`。请注意，`Output` 类型是 `Option<String>`，与 `page_title` 的 `async fn` 版本原本的返回类型相同。
- 原函数体中调用的所有代码都包裹在 `async move` 块中。请记住，代码块是表达式。整个代码块就是函数返回的表达式。
- 如前所述，这个 async 块产生 `Option<String>` 类型的值。该值与返回类型中的 `Output` 类型相匹配。这与之前见过的其他代码块一样。
- 新函数体之所以是 `async move` 块，是因为它使用 `url` 形参的方式。（本章稍后会进一步讨论 `async` 与 `async move`。）

现在可以在 `main` 中调用 `page_title`。

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### 使用运行时执行异步函数

首先取得单个页面的标题，如示例 17-3 所示。遗憾的是，这段代码暂时无法编译。

<Listing number="17-3" file-name="src/main.rs" caption="从 `main` 中调用 `page_title` 函数，传入用户提供的实参">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

我们采用第 12 章[“接收命令行实参”][cli-args]<!-- ignore -->一节中获取命令行实参的相同模式。然后把 URL 实参传给 `page_title` 并等待结果。由于 future 产生的值是 `Option<String>`，我们使用 `match` 表达式打印不同消息，以分别处理页面是否包含 `<title>` 的情况。

`await` 关键字只能在异步函数或 async 块中使用，而 Rust 不允许把特殊的 `main` 函数标记为 `async`。

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main` 不能标记为 `async`，是因为异步代码需要<em>运行时(runtime)</em>：一个管理异步代码执行细节的 Rust crate。程序的 `main` 函数可以<em>初始化</em>运行时，但它本身<em>不是</em>运行时。（稍后会进一步了解原因。）每个执行异步代码的 Rust 程序，都至少有一个设置运行时来执行 future 的位置。

大多数支持 async 的语言都捆绑了运行时，但 Rust 没有。相反，它提供许多不同的异步运行时，每种运行时都有适合目标用例的不同权衡。例如，拥有许多 CPU 核心和大量 RAM 的高吞吐量 Web 服务器，与只有单个核心、少量 RAM 且无法进行堆分配的微控制器，需求截然不同。提供这些运行时的 crate 通常还会提供文件或网络 I/O 等常用功能的异步版本。

在这里以及本章余下部分，我们将使用 `trpl` crate 的 `block_on` 函数。它接收一个 future 作为实参，并阻塞当前线程，直到该 future 运行完成。在幕后，调用 `block_on` 会使用 `tokio` crate 设置一个运行时，用来运行传入的 future（`trpl` crate 的 `block_on` 行为与其他运行时 crate 的 `block_on` 函数相似）。future 完成后，`block_on` 返回该 future 产生的值。

可以把 `page_title` 返回的 future 直接传给 `block_on`；future 完成后，可以像示例 17-3 中尝试的那样，对得到的 `Option<String>` 进行匹配。然而，在本章的大多数示例（以及现实世界的大多数异步代码）中，我们要做的不只一次异步函数调用，因此会改为传入一个 `async` 块，并显式等待 `page_title` 调用的结果，如示例 17-4 所示。

<Listing number="17-4" caption="使用 `trpl::block_on` 等待 async 块" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

运行这段代码，会得到我们最初期待的行为：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

呼——终于有了可以工作的异步代码！不过，在添加让两个站点相互竞速的代码之前，先短暂把注意力转回 future 的工作方式。

每个<em>等待点(await point)</em>，即代码中每个使用 `await` 关键字的位置，都代表一个把控制权交还给运行时的地方。为了做到这一点，Rust 需要跟踪 async 块涉及的状态，以便运行时可以开始其他工作，并在准备好再次尝试推进第一个工作时返回。这是一个不可见的<em>状态机(state machine)</em>，就好像你编写了类似下面的枚举，在每个等待点保存当前状态：

```rust
{{#rustdoc_include ../../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

不过，手动编写在各状态间转换的代码既繁琐又容易出错，尤其是在以后需要为代码添加更多功能和更多状态时。幸运的是，Rust 编译器会自动为异步代码创建并管理状态机数据结构。围绕数据结构的普通借用和所有权规则仍然全部适用；令人欣慰的是，编译器也会替我们检查这些规则，并提供有用的错误消息。本章稍后会逐一研究其中一些情况。

归根结底，必须由某种东西执行这个状态机，而它就是运行时。（这也解释了为什么研究运行时时可能会看到<em>执行器(executor)</em>一词：执行器是运行时中负责执行异步代码的部分。）

现在你就能理解，为什么示例 17-3 中编译器阻止我们把 `main` 本身设为异步函数。如果 `main` 是异步函数，就需要其他东西管理 `main` 所返回 future 的状态机，但 `main` 是程序的起点！因此，我们在 `main` 中调用 `trpl::block_on` 函数，设置运行时并运行 `async` 块返回的 future，直到它完成。

> 注意：一些运行时提供宏，让你<em>可以</em>编写异步 `main` 函数。这些宏会把 `async fn main() { ... }` 改写为普通的 `fn main`，完成与示例 17-4 中手动操作相同的事情：调用一个像 `trpl::block_on` 那样运行 future 直至完成的函数。

现在把这些部分组合起来，看看如何编写并发代码。

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### 让两个 URL 并发竞速

在示例 17-5 中，我们使用从命令行传入的两个不同 URL 调用 `page_title`，并选择率先完成的 future，让两者相互竞速。

<Listing number="17-5" caption="为两个 URL 调用 `page_title`，观察哪一个率先返回" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

首先为用户提供的每个 URL 调用 `page_title`，把得到的 future 保存为 `title_fut_1` 和 `title_fut_2`。请记住，它们此时还什么也不会做，因为 future 是惰性的，我们尚未等待它们。然后把这些 future 传给 `trpl::select`，后者返回一个值，表明传入的哪一个 future 率先完成。

> 注意：在底层，`trpl::select` 建立在 `futures` crate 中定义的、更通用的 `select` 函数之上。`futures` crate 的 `select` 函数能完成许多 `trpl::select` 无法完成的事情，但也带来一些额外复杂性，我们暂时可以跳过。

任意一个 future 都可能合理地“获胜”，所以返回 `Result` 没有意义。`trpl::select` 返回的是之前未见过的类型 `trpl::Either`。`Either` 类型与 `Result` 有些相似，因为它有两种情况。不过与 `Result` 不同，`Either` 并未内置成功或失败的概念，而是使用 `Left` 和 `Right` 表示“二者之一”：

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

如果第一个实参获胜，`select` 函数返回包含该 future 输出的 `Left`；如果第二个 future 实参获胜，则返回包含其输出的 `Right`。这与调用函数时实参出现的顺序相符：第一个实参位于第二个实参的左侧。

我们还更新了 `page_title`，让它同时返回传入的 URL。这样，即使率先返回的页面没有可解析的 `<title>`，仍然可以打印有意义的消息。得到这些信息后，最后更新 `println!` 输出，表明哪个 URL 率先完成，以及该 URL 对应网页的 `<title>`（如果存在）是什么。

现在，你已经构建了一个可以工作的小型 Web 抓取器！选择一对 URL 并运行这个命令行工具。你可能会发现有些网站始终比其他网站快，而在另一些情况下，速度更快的网站会在不同运行之间发生变化。更重要的是，你已经学会了使用 future 的基础知识，现在可以深入探索 async 还能做些什么。

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
