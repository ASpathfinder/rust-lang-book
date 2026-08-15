<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## 把错误重定向到标准错误

目前，我们使用 `println!` 宏把所有输出写入终端。在大多数终端中，输出分为两类：用于一般信息的<em>标准输出(standard output)</em>（`stdout`），以及用于错误信息的<em>标准错误(standard error)</em>（`stderr`）。这种区分让用户可以选择把程序成功运行的输出定向到文件，同时仍把错误信息打印到屏幕上。

`println!` 宏只能打印到标准输出，所以必须使用其他方式打印到标准错误。

### 检查错误写到了哪里

首先，观察 `minigrep` 打印的内容目前如何写入标准输出，其中包括我们希望改写到标准错误的所有错误信息。为此，我们会把标准输出流重定向到文件，同时故意引发错误。不会重定向标准错误流，因此发送到标准错误的所有内容仍会显示在屏幕上。

命令行程序应该把错误信息发送到标准错误流，这样即使把标准输出流重定向到文件，仍能在屏幕上看到错误信息。程序目前的行为并不规范：马上会看到，它竟然把错误信息输出保存到了文件中！

为了展示这种行为，我们会使用 `>` 和文件路径 <em>output.txt</em> 运行程序，把标准输出流重定向到该文件。我们不传任何实参，这应该会导致错误：

```console
$ cargo run > output.txt
```

`>` 语法告诉 shell，把标准输出的内容写入 <em>output.txt</em> 而不是屏幕。屏幕上没有打印预期的错误信息，因此它一定进入了文件。<em>output.txt</em> 的内容如下：

```text
Problem parsing arguments: not enough arguments
```

没错，错误信息打印到了标准输出。把这类错误信息打印到标准错误会更有用，这样只有成功运行的数据才会进入文件。我们来修改它。

### 把错误打印到标准错误

我们会使用示例 12-24 中的代码修改错误信息的打印方式。得益于本章前面完成的重构，打印错误信息的所有代码都集中在 `main` 这一个函数中。标准库提供了打印到标准错误流的 `eprintln!` 宏，因此把两个调用 `println!` 打印错误的位置改为使用 `eprintln!`。

<Listing number="12-24" file-name="src/main.rs" caption="使用 `eprintln!` 把错误信息写入标准错误而不是标准输出">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

现在再次以相同方式运行程序，不传任何实参并使用 `>` 重定向标准输出：

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

现在屏幕上会显示错误，而 <em>output.txt</em> 不含任何内容，这正是命令行程序应有的行为。

再使用不会导致错误的实参运行程序，但仍把标准输出重定向到文件：

```console
$ cargo run -- to poem.txt > output.txt
```

终端上不会显示任何输出，而 <em>output.txt</em> 包含搜索结果：

<span class="filename">文件名：output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

这表明我们现在已经恰当地使用标准输出输出成功结果、使用标准错误输出错误。

## 总结

本章回顾了目前学过的一些主要概念，并介绍了如何在 Rust 中执行常见 I/O 操作。通过使用命令行实参、文件、环境变量和打印错误的 `eprintln!` 宏，你现在已经能够编写命令行应用程序。结合前面各章的概念，代码会组织良好、在合适的数据结构中有效存储数据、妥善处理错误，并得到充分测试。

接下来，我们会探索一些受函数式语言影响的 Rust 功能：闭包与迭代器。
