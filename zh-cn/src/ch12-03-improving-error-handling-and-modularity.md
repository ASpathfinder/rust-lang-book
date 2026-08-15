## 通过重构改进模块化与错误处理

为了改进程序，我们会修复与程序结构和潜在错误处理方式有关的四个问题。首先，`main` 函数目前执行两项任务：解析实参和读取文件。随着程序增长，`main` 函数处理的独立任务会越来越多。函数承担的职责越多，就越难理解、越难测试，也越难在不破坏某个部分的情况下修改。最好把功能分开，让每个函数负责一项任务。

这个问题也与第二个问题有关：虽然 `query` 和 `file_path` 是程序的配置变量，但 `contents` 等变量用于执行程序逻辑。`main` 越长，需要引入作用域的变量就越多；作用域中的变量越多，就越难跟踪每个变量的用途。最好把配置变量组合到一个结构中，使其用途清晰明确。

第三个问题是，读取文件失败时，我们使用 `expect` 打印错误信息，但错误信息只会打印 `Should have been able to read the file`。读取文件可能以多种方式失败，例如文件可能不存在，或者我们没有权限打开它。目前，无论具体情况如何，所有错误都会打印相同信息，无法为用户提供任何有用信息！

第四，我们使用 `expect` 处理错误；如果用户运行程序时没有指定足够的实参，会收到 Rust 的 `index out of bounds` 错误，无法清楚说明问题。如果所有错误处理代码都集中在一个位置，那么未来维护者需要修改错误处理逻辑时，只需查看一个地方；这样会更好。把所有错误处理代码放在一个位置，还能确保打印的信息对最终用户有意义。

让我们通过重构项目解决这四个问题。

<!-- Old headings. Do not remove or links may break. -->

<a id="separation-of-concerns-for-binary-projects"></a>

### 分离二进制项目中的关注点

让 `main` 函数负责多项任务的组织问题，在许多二进制项目中都很常见。因此，当 `main` 函数开始变大时，许多 Rust 程序员认为把二进制程序的不同<em>关注点(concern)</em>拆分开很有帮助。这个过程包括以下步骤：

- 把程序拆分为 <em>main.rs</em> 和 <em>lib.rs</em> 文件，并把程序逻辑移到 <em>lib.rs</em>。
- 只要命令行解析逻辑仍然很少，就可以留在 `main` 函数中。
- 当命令行解析逻辑开始变得复杂时，将其从 `main` 函数提取到其他函数或类型中。

完成这个过程后，`main` 函数中保留的职责应该仅限于：

- 使用实参值调用命令行解析逻辑
- 设置其他所有配置
- 调用 <em>lib.rs</em> 中的 `run` 函数
- 如果 `run` 返回错误，则处理错误

这种模式旨在<em>分离关注点(separation of concerns)</em>：<em>main.rs</em> 处理程序运行，<em>lib.rs</em> 处理当前任务的所有逻辑。由于不能直接测试 `main` 函数，这种结构把所有程序逻辑移出 `main`，让你可以测试它们。`main` 函数中剩下的代码会少到足以通过阅读来验证正确性。让我们按照这个过程重新组织程序。

#### 提取实参解析器

我们会把解析实参的功能提取到一个由 `main` 调用的函数中。示例 12-5 展示了 `main` 函数新的开头，它会调用我们将在 <em>src/main.rs</em> 中定义的新函数 `parse_config`。

<Listing number="12-5" file-name="src/main.rs" caption="从 `main` 中提取 `parse_config` 函数">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

我们仍然把命令行实参收集到向量中，但不再于 `main` 函数内把索引 1 处的实参值赋给变量 `query`、把索引 2 处的实参值赋给变量 `file_path`，而是把整个向量传给 `parse_config` 函数。`parse_config` 函数随后包含判断哪个实参应放入哪个变量的逻辑，并把值传回 `main`。我们仍在 `main` 中创建 `query` 和 `file_path` 变量，但 `main` 不再负责确定命令行实参与变量的对应关系。

对于这个小程序，这种改动看起来可能有些过度，但我们正在以小步、渐进的方式重构。完成修改后，请再次运行程序，验证实参解析仍然正常。经常检查进度是一种好习惯，有助于在问题出现时找出原因。

#### 组合配置值

我们可以再迈出一小步，进一步改进 `parse_config` 函数。目前，我们返回一个元组，但随后又立即把元组拆成各个部分。这表明也许还没有找到正确的抽象。

另一个表明存在改进空间的迹象是 `parse_config` 名称中的 `config` 部分，它暗示返回的两个值彼此相关，都是一个配置值的组成部分。目前，除了把两个值组合成元组，数据结构没有传达这层含义；我们会把两个值放入同一个结构体，并为每个结构体字段赋予有意义的名称。这样，未来的代码维护者就更容易理解不同值之间的关系及其用途。

示例 12-6 展示了对 `parse_config` 函数的改进。

<Listing number="12-6" file-name="src/main.rs" caption="重构 `parse_config`，使其返回 `Config` 结构体的实例">

```rust,should_panic,noplayground
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

我们添加了名为 `Config` 的结构体，它定义了名为 `query` 和 `file_path` 的字段。现在，`parse_config` 的签名表明它返回 `Config` 值。在 `parse_config` 的函数体中，我们过去返回引用 `args` 中 `String` 值的字符串切片，现在则把 `Config` 定义为包含拥有所有权的 `String` 值。`main` 中的 `args` 变量是实参值的所有者，只允许 `parse_config` 函数借用它们；这意味着如果 `Config` 试图取得 `args` 中值的所有权，就会违反 Rust 的借用规则。

有多种方式可以管理 `String` 数据；最简单但效率略低的方式，是对值调用 `clone` 方法。这会完整复制数据，让 `Config` 实例拥有副本；与存储字符串数据的引用相比，需要更多时间和内存。不过，克隆数据也让代码非常直观，因为无需管理引用的生命周期；在这种情况下，牺牲少量性能来换取简单性是一项值得的取舍。

> ### 使用 `clone` 的取舍
>
> 由于 `clone` 会产生运行时成本，许多 Rust 程序员倾向于避免用它解决所有权问题。[第 13 章][ch13]会介绍如何在这类情况下使用更高效的方法。不过现在，为了继续推进，复制几个字符串没有问题，因为只会复制一次，而且文件路径和查询字符串都很短。初次实现时，拥有一个虽然效率稍低但能工作的程序，比尝试过度优化代码更好。随着 Rust 经验增长，从最高效的解决方案开始会变得更容易；但现在，调用 `clone` 完全可以接受。

我们更新了 `main`，让它把 `parse_config` 返回的 `Config` 实例放入名为 `config` 的变量；还更新了过去使用独立 `query` 和 `file_path` 变量的代码，改为使用 `Config` 结构体的字段。

现在，代码更清楚地表达了 `query` 与 `file_path` 相互关联，其用途是配置程序如何工作。任何使用这些值的代码都知道，应在 `config` 实例中根据用途命名的字段里查找它们。

#### 为 `Config` 创建构造函数

到目前为止，我们把负责解析命令行实参的逻辑从 `main` 提取出来，放入 `parse_config` 函数。这样做帮助我们看出 `query` 与 `file_path` 值彼此相关，而且代码应表达这种关系。随后添加 `Config` 结构体，为 `query` 和 `file_path` 的相关用途命名，也让 `parse_config` 函数能够以结构体字段名称返回这些值。

既然 `parse_config` 函数的用途是创建 `Config` 实例，就可以把 `parse_config` 从普通函数改为与 `Config` 结构体关联、名为 `new` 的函数。这项修改会让代码更加符合惯用写法。可以调用 `String::new` 创建 `String` 等标准库类型的实例。类似地，把 `parse_config` 改为与 `Config` 关联的 `new` 函数后，就能调用 `Config::new` 创建 `Config` 实例。示例 12-7 展示了所需修改。

<Listing number="12-7" file-name="src/main.rs" caption="把 `parse_config` 改为 `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

我们更新了 `main` 中调用 `parse_config` 的位置，改为调用 `Config::new`；还把 `parse_config` 重命名为 `new`，并将其移入 `impl` 块，使 `new` 函数与 `Config` 关联。请再次尝试编译这段代码，确保它能正常工作。

### 修复错误处理

现在来改进错误处理。回想一下，如果向量少于三项，尝试访问 `args` 向量索引 1 或索引 2 处的值会使程序 panic。尝试不带任何实参运行程序，结果如下：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

`index out of bounds: the len is 1 but the index is 1` 这一行是面向程序员的错误信息，无法帮助最终用户理解应该怎么做。现在就来修复它。

#### 改进错误信息

在示例 12-8 中，我们向 `new` 函数添加一项检查，在访问索引 1 和索引 2 之前验证切片足够长。如果切片不够长，程序会 panic 并显示更好的错误信息。

<Listing number="12-8" file-name="src/main.rs" caption="添加实参数量检查">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

这段代码与[示例 9-13 中编写的 `Guess::new` 函数][ch9-custom-types]类似；当 `value` 实参超出有效值范围时，我们会调用 `panic!`。这里不检查值的范围，而是检查 `args` 长度至少为 `3`，随后函数其余部分便可在该条件已经满足的假设下执行。如果 `args` 少于三项，这个条件会是 `true`，我们调用 `panic!` 宏立即结束程序。

向 `new` 添加这几行代码后，再次不带实参运行程序，看看现在错误是什么样子：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

这段输出更好了：现在有了合理的错误信息。不过，其中仍包含不想提供给用户的多余信息。也许示例 9-13 中使用的技术并不最适合这里：正如[第 9 章所讨论的][ch9-error-guidelines]，调用 `panic!` 更适合编程问题，而不是使用方式问题。我们将改用第 9 章学过的另一种技术——[返回 `Result`][ch9-result]，表明操作成功或发生错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### 返回 `Result` 而不是调用 `panic!`

我们可以改为返回 `Result` 值：成功时包含 `Config` 实例，出错时描述问题。还会把函数名从 `new` 改为 `build`，因为许多程序员期望 `new` 函数绝不失败。当 `Config::build` 与 `main` 通信时，可以使用 `Result` 类型表明出现问题。随后可以修改 `main`，把 `Err` 变体转换为对用户更实用的错误，而不带调用 `panic!` 时产生的 `thread 'main'` 和 `RUST_BACKTRACE` 等周边文本。

示例 12-9 展示了如何修改现在名为 `Config::build` 的函数返回值，以及为返回 `Result` 而需要修改的函数体。请注意，在下一示例更新 `main` 之前，这段代码无法编译。

<Listing number="12-9" file-name="src/main.rs" caption="从 `Config::build` 返回 `Result`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

`build` 函数返回一个 `Result`，成功时包含 `Config` 实例，出错时包含字符串字面量。错误值始终是具有 `'static` 生命周期的字符串字面量。

我们对函数体做了两项修改：当用户没有传入足够实参时，不再调用 `panic!`，而是返回 `Err` 值；并用 `Ok` 包裹 `Config` 返回值。这些修改让函数符合新的类型签名。

从 `Config::build` 返回 `Err` 值，使 `main` 函数能够处理 `build` 函数返回的 `Result` 值，并在出错时更干净地退出进程。

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### 调用 `Config::build` 并处理错误

为了处理错误情况并打印用户友好的信息，需要更新 `main` 来处理 `Config::build` 返回的 `Result`，如示例 12-10 所示。还会让 `panic!` 不再负责以非零错误码退出命令行工具，改为手动实现。非零退出状态是一项惯例，用于向调用程序的进程表明程序以错误状态退出。

<Listing number="12-10" file-name="src/main.rs" caption="如果构建 `Config` 失败，则以错误码退出">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

这个示例使用了尚未详细介绍的方法：标准库在 `Result<T, E>` 上定义的 `unwrap_or_else`。使用 `unwrap_or_else` 可以定义自定义的非 `panic!` 错误处理。如果 `Result` 是 `Ok` 值，该方法的行为与 `unwrap` 相似：返回 `Ok` 包裹的内部值。不过，如果值为 `Err`，该方法会调用闭包中的代码；闭包是我们定义并作为实参传给 `unwrap_or_else` 的匿名函数。[第 13 章][ch13]会更详细地介绍闭包。现在只需知道，`unwrap_or_else` 会把 `Err` 的内部值——这里是示例 12-9 中添加的静态字符串 `"not enough arguments"`——传给闭包中竖线之间的实参 `err`。闭包中的代码运行时便可使用 `err` 值。

我们添加了一条新的 `use` 语句，把标准库中的 `process` 引入作用域。发生错误时运行的闭包代码只有两行：打印 `err` 值，然后调用 `process::exit`。`process::exit` 函数会立即停止程序，并把传入的数字作为退出状态码返回。这与示例 12-8 中基于 `panic!` 的处理方式类似，但不再得到所有额外输出。来试试看：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

很好！这段输出对用户友好多了。

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-the-main-function"></a>

### 从 `main` 提取逻辑

配置解析重构完成后，接下来处理程序逻辑。正如[“分离二进制项目中的关注点”](#separation-of-concerns-for-binary-projects)所述，我们会提取一个名为 `run` 的函数，用来包含 `main` 函数中与设置配置或处理错误无关的所有现有逻辑。完成后，`main` 函数会变得简洁，易于通过检查验证；我们还能为其他所有逻辑编写测试。

示例 12-11 展示了提取 `run` 函数这一小步渐进式改进。

<Listing number="12-11" file-name="src/main.rs" caption="提取包含程序其余逻辑的 `run` 函数">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

现在，从读取文件开始，`run` 函数包含 `main` 中剩余的所有逻辑。`run` 函数接收 `Config` 实例作为实参。

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-errors-from-the-run-function"></a>

#### 从 `run` 返回错误

剩余程序逻辑已分离到 `run` 函数后，可以像示例 12-9 中的 `Config::build` 那样改进错误处理。发生问题时，`run` 函数不再通过调用 `expect` 允许程序 panic，而是返回 `Result<T, E>`。这样就能进一步把错误处理逻辑集中到 `main` 中，以用户友好的方式处理。示例 12-12 展示了需要对 `run` 的签名和函数体做出的修改。

<Listing number="12-12" file-name="src/main.rs" caption="修改 `run` 函数，使其返回 `Result`">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

这里做了三项重要修改。首先，把 `run` 函数的返回类型改为 `Result<(), Box<dyn Error>>`。该函数过去返回单元类型 `()`，我们保留它作为 `Ok` 情况下返回的值。

对于错误类型，我们使用特征对象 `Box<dyn Error>`（并通过文件顶部的 `use` 语句把 `std::error::Error` 引入作用域）。[第 18 章][ch18]会介绍特征对象。现在只需知道，`Box<dyn Error>` 表示函数会返回某种实现了 `Error` 特征的类型，但无需指定返回值具体是什么类型。这让我们可以灵活地在不同错误情况下返回可能具有不同类型的错误值。`dyn` 关键字是 <em>dynamic</em> 的缩写。

第二，按照[第 9 章][ch9-question-mark]讨论的内容，我们删除 `expect` 调用，改用 `?` 运算符。发生错误时，`?` 不会 `panic!`，而是从当前函数返回错误值，交由调用方处理。

第三，`run` 函数现在会在成功时返回 `Ok` 值。我们在签名中把 `run` 函数的成功类型声明为 `()`，因此需要把单元类型值包裹在 `Ok` 中。`Ok(())` 语法起初可能有些奇怪，但这是表明调用 `run` 仅为了其副作用、不需要返回值的惯用方式。

运行这段代码时，它可以编译，但会显示警告：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust 告诉我们，代码忽略了 `Result` 值，而该值可能表示发生了错误。但我们没有检查是否出现错误，编译器提醒我们可能本应在这里编写错误处理代码！现在来纠正这个问题。

#### 在 `main` 中处理 `run` 返回的错误

我们会使用与示例 12-10 中处理 `Config::build` 类似、但略有不同的技巧来检查并处理错误：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

我们使用 `if let` 而不是 `unwrap_or_else` 来检查 `run` 是否返回 `Err` 值，并在返回时调用 `process::exit(1)`。`run` 不会像 `Config::build` 那样返回我们想要 `unwrap` 的 `Config` 实例。由于 `run` 成功时返回 `()`，我们只关心是否检测到错误，不需要 `unwrap_or_else` 返回解包后的值——它只会是 `()`。

两种情况下，`if let` 与 `unwrap_or_else` 函数体都相同：打印错误并退出。

### 把代码拆分到库 crate 中

到目前为止，`minigrep` 项目看起来很不错！现在，我们会拆分 <em>src/main.rs</em> 文件，把部分代码放入 <em>src/lib.rs</em>。这样就能测试代码，并让 <em>src/main.rs</em> 承担更少职责。

让我们在 <em>src/lib.rs</em> 而不是 <em>src/main.rs</em> 中定义负责搜索文本的代码，这样我们（或其他使用 `minigrep` 库的人）就能在 `minigrep` 二进制程序以外的更多上下文中调用搜索函数。

首先，按照示例 12-13 在 <em>src/lib.rs</em> 中定义 `search` 函数签名，函数体调用 `unimplemented!` 宏。填充实现时，我们会更详细地解释该签名。

<Listing number="12-13" file-name="src/lib.rs" caption="在 <em>src/lib.rs</em> 中定义 `search` 函数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

我们在函数定义中使用 `pub` 关键字，把 `search` 指定为库 crate 公有 API 的一部分。现在，我们拥有了一个可以从二进制 crate 使用、也可以进行测试的库 crate！

接下来，需要把 <em>src/lib.rs</em> 中定义的代码引入 <em>src/main.rs</em> 中二进制 crate 的作用域并调用它，如示例 12-14 所示。

<Listing number="12-14" file-name="src/main.rs" caption="在 <em>src/main.rs</em> 中使用 `minigrep` 库 crate 的 `search` 函数">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

我们添加 `use minigrep::search` 行，把库 crate 中的 `search` 函数引入二进制 crate 的作用域。然后，在 `run` 函数中不再打印文件内容，而是调用 `search` 函数，把 `config.query` 值和 `contents` 作为实参传入。接着，`run` 使用 `for` 循环打印 `search` 返回的每一行查询匹配结果。这也是删除 `main` 函数中显示查询内容和文件路径的 `println!` 调用的好时机，使程序在没有错误时只打印搜索结果。

请注意，搜索函数会先把所有结果收集到返回的向量中，然后才进行打印。搜索大型文件时，这种实现可能会很晚才显示结果，因为它不会在找到结果时立即打印；第 13 章会讨论使用迭代器修复这个问题的一种可行方式。

呼！工作量真不少，但我们已经为未来的成功做好了准备。现在，处理错误容易得多，代码也更加模块化。从这里开始，几乎所有工作都会在 <em>src/lib.rs</em> 中完成。

让我们利用这项新获得的模块化能力，做一件过去代码难以完成、而新代码轻松就能做到的事：编写测试！

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
