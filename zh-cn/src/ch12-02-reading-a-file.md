## 读取文件

现在添加读取 `file_path` 实参所指定文件的功能。首先，需要一个用于测试的示例文件：我们会使用一个包含少量多行文本、其中一些单词重复出现的文件。示例 12-3 中 Emily Dickinson 的诗就很合适！在项目根目录创建名为 <em>poem.txt</em> 的文件，并输入诗歌《I’m Nobody! Who are you?》。

<Listing number="12-3" file-name="poem.txt" caption="Emily Dickinson 的诗是一个很好的测试用例">

```text
{{#include ../../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

文本准备好后，编辑 <em>src/main.rs</em>，添加读取文件的代码，如示例 12-4 所示。

<Listing number="12-4" file-name="src/main.rs" caption="读取第二个实参所指定文件的内容">

```rust,should_panic,noplayground
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

首先使用 `use` 语句引入标准库中相关的部分：需要 `std::fs` 来处理文件。

在 `main` 中，新语句 `fs::read_to_string` 接收 `file_path`，打开该文件，并返回包含文件内容的 `std::io::Result<String>` 类型值。

之后，我们再次添加临时 `println!` 语句，在读取文件后打印 `contents` 的值，以便检查程序目前是否正常工作。

让我们使用任意字符串作为第一个命令行实参（因为还没有实现搜索部分），并以 <em>poem.txt</em> 文件作为第二个实参来运行代码：

```console
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

很好！代码读取并打印了文件内容。不过，代码还有一些缺陷。目前，`main` 函数承担了多项职责：通常，如果每个函数只负责一个概念，函数会更清晰，也更容易维护。另一个问题是错误处理还可以做得更好。程序仍然很小，所以这些缺陷问题不大；但随着程序增长，要干净利落地修复它们会变得更加困难。开发程序时尽早开始重构是一项良好实践，因为重构少量代码容易得多。接下来就来进行重构。
