## Hello, World!

安装好 Rust 之后，就可以编写第一个 Rust 程序了。学习一门新语言时，通常会先
写一个在屏幕上打印 `Hello, world!` 的小程序，所以这里也这样做！

> 注意：本书假定你对命令行有基本了解。Rust 对编辑方式、所用工具或代码存放
> 位置没有特殊要求，因此如果你更愿意使用 IDE 而不是命令行，完全可以选择
> 自己喜欢的 IDE。如今许多 IDE 都在一定程度上支持 Rust；详情请查阅相应 IDE
> 的文档。Rust 团队一直致力于通过 `rust-analyzer` 提供出色的 IDE 支持。
> 更多信息请参阅[附录 D][devtools]<!-- ignore -->。

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-project-directory"></a>

### 设置项目目录

首先创建一个用来存放 Rust 代码的目录。Rust 并不在意代码放在哪里，但为了
完成本书中的练习和项目，建议在主目录下创建一个 <em>projects</em> 目录，把所有项目
都放在这里。

打开终端并输入以下命令，创建 <em>projects</em> 目录，然后在其中为“Hello, world!”
项目创建一个目录。

在 Linux、macOS 和 Windows PowerShell 中输入：

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

在 Windows CMD 中输入：

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-and-running-a-rust-program"></a>

<a id="rust-program-basics"></a>

### Rust 程序基础

接下来，新建一个名为 <em>main.rs</em> 的<em>源文件(source file)</em>。Rust 文件的扩展名
总是 <em>.rs</em>。如果文件名由多个单词组成，惯例是使用下划线分隔。例如，应使用
<em>hello_world.rs</em>，而不是 <em>helloworld.rs</em>。

打开刚刚创建的 <em>main.rs</em> 文件，输入示例 1-1 中的代码。

<Listing number="1-1" file-name="main.rs" caption="打印 `Hello, world!` 的程序">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

保存文件，然后回到终端中的 <em>~/projects/hello_world</em> 目录。在 Linux 或
macOS 上，输入以下命令编译并运行这个文件：

```console
$ rustc main.rs
$ ./main
Hello, world!
```

在 Windows 上，请输入 `.\main`，而不是 `./main`：

```powershell
> rustc main.rs
> .\main
Hello, world!
```

无论使用哪种操作系统，终端都应该打印字符串 `Hello, world!`。如果没有看到
这一输出，请返回“安装”一节的[“故障排查”部分][troubleshooting]<!-- ignore -->
寻找解决方法。

如果成功打印出 `Hello, world!`，恭喜！你已经正式编写了一个 Rust 程序，
从此就是一名 Rust 程序员了——欢迎你！

<!-- Old headings. Do not remove or links may break. -->

<a id="anatomy-of-a-rust-program"></a>

### Rust 程序剖析

下面详细回顾这个“Hello, world!”程序。先来看第一部分：

```rust
fn main() {

}
```

这几行定义了一个名为 `main` 的<em>函数(function)</em>。`main` 函数非常特殊：它总是
每个 Rust 可执行程序最先运行的代码。第一行声明了一个名为 `main` 的函数；
它没有<em>形参(parameter)</em>，也不返回任何内容。如果有形参，就应写在圆括号 `()`
内。

<em>函数体(function body)</em>由 `{}` 包围。Rust 要求所有函数体都使用花括号。良好的
代码风格是把左花括号放在函数声明所在行，并在两者之间留一个空格。

> 注意：如果想让不同 Rust 项目都遵循统一风格，可以使用名为 `rustfmt` 的
> 自动格式化工具，按照特定风格格式化代码（关于 `rustfmt` 的更多内容，请参阅
> [附录 D][devtools]<!-- ignore -->）。与 `rustc` 一样，Rust 团队已经把这个
> 工具包含在标准 Rust 发行版中，因此你的计算机上应该已经安装了它！

`main` 函数体包含以下代码：

```rust
println!("Hello, world!");
```

这个小程序的所有工作都由这一行完成：它在屏幕上打印文字。这里有三个重要细节。

第一，`println!` 调用的是一个 Rust <em>宏(macro)</em>。如果它调用的是函数，则应写成
`println`（不带 `!`）。Rust 宏是一种编写代码来生成其他代码、进而扩展 Rust
语法的方式；[第 20 章][ch20-macros]<!-- ignore -->会更详细地讨论宏。目前只需
知道，`!` 表明这里调用的是宏而不是普通函数，而且宏并不总是遵循与函数相同的
规则。

第二，可以看到<em>字符串(string)</em> `"Hello, world!"`。我们把这个字符串作为<em>实参
(argument)</em>传给 `println!`，它便会被打印到屏幕上。

第三，该行以分号 `;` 结尾，表示这个<em>表达式(expression)</em>已经结束，可以开始
下一个表达式。大多数 Rust 代码行都以分号结尾。

<!-- Old headings. Do not remove or links may break. -->
<a id="compiling-and-running-are-separate-steps"></a>

### 编译和执行

你刚刚运行了一个新创建的程序，下面来看看这个过程中的每一步。

运行 Rust 程序之前，必须先使用 Rust 编译器完成编译：输入 `rustc` 命令，并
把源文件名传给它，如下所示：

```console
$ rustc main.rs
```

如果你有 C 或 C++ 背景，会发现这和使用 `gcc` 或 `clang` 很相似。成功编译
后，Rust 会输出一个<em>二进制可执行文件(binary executable)</em>。

在 Linux、macOS 和 Windows PowerShell 中，可以在 shell 里输入 `ls` 命令
查看可执行文件：

```console
$ ls
main  main.rs
```

在 Linux 和 macOS 上，你会看到两个文件。使用 Windows PowerShell 时，看到
的三个文件与使用 CMD 时相同。在 Windows CMD 中，应输入：

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

这里显示了扩展名为 <em>.rs</em> 的源代码文件、可执行文件（在 Windows 上是
<em>main.exe</em>，其他平台上是 <em>main</em>），以及 Windows 上扩展名为 <em>.pdb</em>、包含
调试信息的文件。接下来可以像这样运行 <em>main</em> 或 <em>main.exe</em>：

```console
$ ./main # or .\main on Windows
```

如果 <em>main.rs</em> 是刚才的“Hello, world!”程序，这条命令会在终端中打印
`Hello, world!`。

如果你更熟悉 Ruby、Python 或 JavaScript 等<em>动态语言(dynamic language)</em>，
可能不习惯把编译和运行程序分成两个步骤。Rust 是一门<em>预先编译
(ahead-of-time compiled)</em>的语言，这意味着你可以编译程序并把可执行文件交给
别人；即使对方没有安装 Rust，也能运行它。如果交给别人的是 <em>.rb</em>、<em>.py</em>
或 <em>.js</em> 文件，对方就必须分别安装 Ruby、Python 或 JavaScript 的实现。不过，
在这些语言中，只需一条命令就能编译并运行程序。编程语言设计中的一切都是取舍。

对于简单程序，只使用 `rustc` 进行编译完全没有问题。但随着项目增长，你会
希望管理各种选项，并方便地共享代码。下一节将介绍 Cargo 工具，它会帮助你
编写实际的 Rust 程序。

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
