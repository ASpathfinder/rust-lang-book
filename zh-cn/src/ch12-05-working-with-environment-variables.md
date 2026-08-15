## 使用环境变量

我们会为 `minigrep` 二进制程序添加额外功能：一个用户可以通过环境变量启用的不区分大小写搜索选项。可以把这项功能做成命令行选项，要求用户每次使用时都输入它；但改用环境变量后，用户只需设置一次，该终端会话中的所有搜索就都不区分大小写。

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### 为不区分大小写的搜索编写失败测试

首先，向 `minigrep` 库添加新的 `search_case_insensitive` 函数，当环境变量有值时调用它。我们会继续遵循 TDD 过程，所以第一步仍然是编写失败测试。我们将为新的 `search_case_insensitive` 函数添加新测试，并把旧测试从 `one_result` 重命名为 `case_sensitive`，明确两个测试之间的区别，如示例 12-20 所示。

<Listing number="12-20" file-name="src/lib.rs" caption="为即将添加的不区分大小写函数添加新的失败测试">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

请注意，我们还编辑了旧测试的 `contents`。添加的新行文本为 `"Duct tape."`，其中有一个大写 <em>D</em>；执行区分大小写的搜索时，它不应该匹配查询 `"duct"`。这样修改旧测试，有助于确保不会意外破坏已经实现的区分大小写搜索功能。该测试现在应该通过，并且在我们开发不区分大小写搜索期间持续通过。

不区分大小写搜索的新测试使用 `"rUsT"` 作为查询。在即将添加的 `search_case_insensitive` 函数中，查询 `"rUsT"` 应该匹配包含大写 <em>R</em> 的 `"Rust:"` 行，也应匹配 `"Trust me."` 行，尽管二者的大小写都与查询不同。这是失败测试；由于尚未定义 `search_case_insensitive` 函数，它无法编译。你可以像示例 12-16 中对 `search` 函数所做的一样，添加一个始终返回空向量的骨架实现，看看测试先编译、再失败。

### 实现 `search_case_insensitive` 函数

示例 12-21 中的 `search_case_insensitive` 函数与 `search` 函数几乎相同。唯一的区别是，我们会把 `query` 和每个 `line` 转换为小写，使输入实参无论原本大小写如何，在检查行是否包含查询时都具有相同大小写。

<Listing number="12-21" file-name="src/lib.rs" caption="定义 `search_case_insensitive` 函数，在比较前把查询和行转换为小写">

```rust,noplayground
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

首先，把 `query` 字符串转换为小写，存入同名的新变量，遮蔽原来的 `query`。必须对查询调用 `to_lowercase`，这样无论用户的查询是 `"rust"`、`"RUST"`、`"Rust"` 还是 `"rUsT"`，我们都会把它当成 `"rust"`，不受大小写影响。虽然 `to_lowercase` 可以处理基本 Unicode，但并非百分之百准确。如果编写真实应用，还需要在这里做更多工作；但本节主题是环境变量而不是 Unicode，所以到此为止。

请注意，`query` 现在是 `String` 而不是字符串切片，因为调用 `to_lowercase` 会创建新数据，而不是引用现有数据。以查询 `"rUsT"` 为例：该字符串切片不包含可供使用的小写 `u` 或 `t`，所以必须分配包含 `"rust"` 的新 `String`。现在把 `query` 作为实参传给 `contains` 方法时，需要添加一个 `&`，因为 `contains` 的签名定义为接收字符串切片。

接着，对每个 `line` 添加 `to_lowercase` 调用，把所有字符转换为小写。现在 `line` 和 `query` 都转换为小写，无论查询采用什么大小写，都能找到匹配项。

来看看这个实现能否通过测试：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

很好！测试通过了。现在从 `run` 函数调用新的 `search_case_insensitive` 函数。首先，向 `Config` 结构体添加配置选项，用于在区分和不区分大小写的搜索之间切换。添加这个字段会导致编译器错误，因为我们尚未在任何地方初始化它：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

我们添加了存放布尔值的 `ignore_case` 字段。接下来，需要让 `run` 函数检查 `ignore_case` 字段的值，并据此决定调用 `search` 还是 `search_case_insensitive` 函数，如示例 12-22 所示。这段代码仍然无法编译。

<Listing number="12-22" file-name="src/main.rs" caption="根据 `config.ignore_case` 中的值调用 `search` 或 `search_case_insensitive`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

最后，需要检查环境变量。处理环境变量的函数位于标准库的 `env` 模块中，该模块已经在 <em>src/main.rs</em> 顶部引入作用域。我们会使用 `env` 模块的 `var` 函数，检查名为 `IGNORE_CASE` 的环境变量是否已设为任意值，如示例 12-23 所示。

<Listing number="12-23" file-name="src/main.rs" caption="检查名为 `IGNORE_CASE` 的环境变量是否具有任意值">

```rust,ignore,noplayground
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

这里，我们创建新变量 `ignore_case`。为了设置其值，我们调用 `env::var` 函数，并向它传入环境变量 `IGNORE_CASE` 的名称。如果该环境变量被设为任意值，`env::var` 函数会返回成功的 `Ok` 变体，其中包含环境变量值；如果环境变量未设置，则返回 `Err` 变体。

我们对 `Result` 使用 `is_ok` 方法来检查环境变量是否设置；如果设置，程序就应执行不区分大小写的搜索。如果 `IGNORE_CASE` 环境变量没有设为任何值，`is_ok` 会返回 `false`，程序执行区分大小写的搜索。我们不关心环境变量的<em>值</em>，只关心它是否设置，因此检查 `is_ok`，而不使用 `unwrap`、`expect` 或见过的其他 `Result` 方法。

我们把 `ignore_case` 变量中的值传给 `Config` 实例，使 `run` 函数能够读取该值，并按照示例 12-22 中的实现决定调用 `search_case_insensitive` 还是 `search`。

来试试看！先在未设置环境变量的情况下，以查询 `to` 运行程序；它应该匹配所有包含全小写单词 <em>to</em> 的行：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

看起来仍然能正常工作！现在把 `IGNORE_CASE` 设为 `1`，使用相同查询 `to` 运行程序：

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

如果使用 PowerShell，需要分别用命令设置环境变量并运行程序：

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

这会使 `IGNORE_CASE` 在 shell 会话剩余期间持续存在。可以使用 `Remove-Item` cmdlet 取消设置：

```console
PS> Remove-Item Env:IGNORE_CASE
```

我们应该得到可能包含大写字母的 <em>to</em> 匹配行：

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

太棒了，我们也得到了包含 <em>To</em> 的行！现在，`minigrep` 程序可以执行由环境变量控制的不区分大小写搜索。你也已经知道如何管理通过命令行实参或环境变量设置的选项。

有些程序允许使用实参<em>和</em>环境变量设置同一项配置。在这种情况下，程序会决定其中哪一个优先。作为自主练习，请尝试通过命令行实参或环境变量控制是否区分大小写。如果程序运行时一个设为区分大小写、另一个设为忽略大小写，请决定应由命令行实参还是环境变量优先。

`std::env` 模块还包含许多处理环境变量的实用功能：请查看其文档，了解可用内容。
