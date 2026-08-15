## 接收命令行实参

像往常一样，使用 `cargo new` 创建新项目。我们把项目命名为 `minigrep`，以便与系统上可能已经存在的 `grep` 工具区分开：

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

第一项任务是让 `minigrep` 接收两个命令行实参：文件路径和要搜索的字符串。也就是说，我们希望能够用 `cargo run` 运行程序，后跟两个连字符，表明之后的实参传给程序而不是 `cargo`，再后跟要搜索的字符串和待搜索文件的路径，如下所示：

```console
$ cargo run -- searchstring example-filename.txt
```

目前，`cargo new` 生成的程序无法处理我们提供的实参。[crates.io](https://crates.io/) 上有一些现成的库可以帮助编写接收命令行实参的程序，但因为你才刚开始学习这个概念，让我们自行实现这项功能。

### 读取实参值

为了让 `minigrep` 读取传给它的命令行实参值，需要使用 Rust 标准库提供的 `std::env::args` 函数。该函数返回一个迭代器，遍历传给 `minigrep` 的命令行实参。[第 13 章][ch13]会完整介绍迭代器。现在只需要知道关于迭代器的两个细节：迭代器会产生一系列值；可以对迭代器调用 `collect` 方法，把它转换为向量等集合，其中包含迭代器产生的所有元素。

示例 12-1 中的代码让 `minigrep` 程序能够读取传给它的任意命令行实参，再把这些值收集到向量中。

<Listing number="12-1" file-name="src/main.rs" caption="把命令行实参收集到向量中并打印">

```rust
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

首先，使用 `use` 语句把 `std::env` 模块引入作用域，以便使用其 `args` 函数。请注意，`std::env::args` 函数嵌套在两层模块中。正如[第 7 章][ch7-idiomatic-use]所讨论的，当所需函数嵌套在不止一层模块中时，我们选择把父模块而不是函数引入作用域。这样就能轻松使用 `std::env` 中的其他函数。这也比添加 `use std::env::args` 再仅以 `args` 调用函数更不容易产生歧义，因为 `args` 很容易被误认为当前模块中定义的函数。

> ### `args` 函数与无效 Unicode
>
> 请注意，如果任何实参包含无效 Unicode，`std::env::args` 会 panic。如果程序需要接收包含无效 Unicode 的实参，请改用 `std::env::args_os`。该函数返回一个产生 `OsString` 值而不是 `String` 值的迭代器。这里为了简单而选择 `std::env::args`，因为 `OsString` 值因平台而异，处理起来也比 `String` 值更复杂。

在 `main` 的第一行，我们调用 `env::args`，并立即使用 `collect` 把迭代器转换为包含其所产生全部值的向量。`collect` 函数可以创建许多种集合，因此我们显式标注 `args` 的类型，指定想要一个字符串向量。虽然在 Rust 中很少需要标注类型，但 `collect` 是经常需要标注的函数之一，因为 Rust 无法推断你想要哪种集合。

最后，我们使用调试宏打印向量。先不带实参运行代码，再带两个实参运行：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

请注意，向量中的第一个值是 `"target/debug/minigrep"`，也就是二进制文件的名称。这与 C 语言中实参列表的行为一致，使程序能够在执行期间使用调用自身时所用的名称。访问程序名称通常很方便，例如可以把它打印在信息中，或者根据调用程序时使用的命令行别名改变程序行为。不过在本章中，我们会忽略它，只保存所需的两个实参。

### 把实参值保存到变量中

程序现在可以访问指定为命令行实参的值。接下来，需要把两个实参的值保存到变量中，以便在程序其余部分使用。示例 12-2 完成了这项工作。

<Listing number="12-2" file-name="src/main.rs" caption="创建变量，存放查询实参和文件路径实参">

```rust,should_panic,noplayground
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

正如打印向量时看到的，程序名占据向量中 `args[0]` 的第一个值，所以从索引 1 开始处理实参。`minigrep` 接收的第一个实参是要搜索的字符串，因此把对第一个实参的引用放入变量 `query`。第二个实参是文件路径，因此把对第二个实参的引用放入变量 `file_path`。

我们暂时打印这些变量的值，证明代码按预期工作。再次使用实参 `test` 和 `sample.txt` 运行程序：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

很好，程序正常工作！所需的实参值已经保存到正确的变量中。稍后会添加错误处理，以应对一些潜在错误情况，例如用户没有提供实参。现在先忽略这种情况，继续添加文件读取功能。

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
