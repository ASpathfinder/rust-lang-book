## Hello, Cargo!

Cargo 是 Rust 的构建系统和包管理器。大多数 Rustacean 都使用它管理 Rust
项目，因为 Cargo 可以代你完成许多任务，例如构建代码、下载代码依赖的库，
以及构建这些库。（代码所需的库称为<em>依赖</em>。）

最简单的 Rust 程序（比如我们目前编写的程序）没有任何依赖。如果用 Cargo
构建“Hello, world!”项目，只会用到 Cargo 负责构建代码的部分。随着编写的
Rust 程序越来越复杂，你会添加各种依赖；如果从一开始就使用 Cargo 创建项目，
添加依赖会容易得多。

绝大多数 Rust 项目都使用 Cargo，因此本书其余部分也假定你会使用 Cargo。
如果通过[“安装”一节][installation]<!-- ignore -->介绍的官方安装程序安装 Rust，
Cargo 会随 Rust 一同安装。如果通过其他方式安装 Rust，请在终端中输入以下
命令，检查 Cargo 是否已安装：

```console
$ cargo --version
```

如果看到版本号，就说明 Cargo 已经安装！如果看到 `command not found` 等错误，
请查阅相应安装方式的文档，了解如何单独安装 Cargo。

### 使用 Cargo 创建项目

下面使用 Cargo 创建一个新项目，看看它与原来的“Hello, world!”项目有何不同。
返回 <em>projects</em> 目录（或你选择用来存放代码的目录），然后在任意操作系统上
运行：

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

第一条命令创建了一个名为 <em>hello_cargo</em> 的新目录和新项目。项目名是
<em>hello_cargo</em>，Cargo 会把项目文件放在同名目录中。

进入 <em>hello_cargo</em> 目录并列出其中的文件。可以看到，Cargo 为我们生成了两个
文件和一个目录：一个 <em>Cargo.toml</em> 文件，以及一个内含 <em>main.rs</em> 文件的
<em>src</em> 目录。

Cargo 还初始化了一个新的 Git 仓库和一个 <em>.gitignore</em> 文件。如果在现有 Git
仓库中运行 `cargo new`，则不会生成 Git 文件；可以使用 `cargo new --vcs=git`
覆盖这一行为。

> 注意：Git 是一种常用的版本控制系统。通过 `--vcs` 标志，可以让 `cargo new`
> 使用其他版本控制系统，或者不使用版本控制。运行 `cargo new --help` 可查看
> 可用选项。

使用你喜欢的文本编辑器打开 <em>Cargo.toml</em>。其内容应该与示例 1-2 类似。

<Listing number="1-2" file-name="Cargo.toml" caption="`cargo new` 生成的 <em>Cargo.toml</em> 内容">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

该文件采用 [<em>TOML</em>][toml]<!-- ignore -->（<em>Tom's Obvious, Minimal Language</em>）
格式，这是 Cargo 使用的配置格式。

第一行 `[package]` 是一个章节标题，表示后面的语句用于配置一个包。随着我们
向文件中添加更多信息，还会增加其他章节。

接下来的三行设置 Cargo 编译程序所需的配置信息：名称、版本以及所用的 Rust
Edition。[附录 E][appendix-e]<!-- ignore -->会进一步介绍 `edition` 键。

最后一行 `[dependencies]` 是一个章节的开头，用于列出项目的各项依赖。在
Rust 中，代码包称为 <em>crate</em>。这个项目不需要任何其他 crate，但第 2 章的第一
个项目会用到，因此届时会使用这个依赖章节。

现在打开 <em>src/main.rs</em> 查看内容：

<span class="filename">文件名：src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 生成了一个“Hello, world!”程序，与我们在示例 1-1 中编写的程序完全
相同！到目前为止，原项目和 Cargo 所生成项目的区别是：Cargo 把代码放进了
<em>src</em> 目录，而且项目顶层目录中多了一个 <em>Cargo.toml</em> 配置文件。

Cargo 期望源文件位于 <em>src</em> 目录中。项目顶层目录只用于存放 README 文件、
许可证信息、配置文件以及其他与代码无关的内容。使用 Cargo 有助于组织项目：
每类内容都有合适的位置，所有内容也都各归其位。

如果项目最初没有使用 Cargo，就像我们的“Hello, world!”项目一样，也可以把
它转换为使用 Cargo 的项目。把项目代码移到 <em>src</em> 目录，并创建合适的
<em>Cargo.toml</em> 文件。获取这个文件的一种简单方法是运行 `cargo init`，它会自动
创建文件。

### 构建并运行 Cargo 项目

现在来看看使用 Cargo 构建和运行“Hello, world!”程序时有哪些不同！在
<em>hello_cargo</em> 目录中输入以下命令构建项目：

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

这条命令会在 <em>target/debug/hello_cargo</em>（Windows 上是
<em>target\debug\hello_cargo.exe</em>）中创建可执行文件，而不是放在当前目录。默认
构建是<em>调试构建(debug build)</em>，因此 Cargo 会把二进制文件放进名为 <em>debug</em> 的
目录。可以使用以下命令运行可执行文件：

```console
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

如果一切正常，终端中应该会打印 `Hello, world!`。第一次运行 `cargo build`
还会让 Cargo 在项目顶层创建一个新文件 <em>Cargo.lock</em>，它记录了项目中各项依赖
的确切版本。这个项目没有依赖，因此文件内容比较少。你完全不需要手动修改这个
文件；Cargo 会替你管理其中的内容。

刚才我们使用 `cargo build` 构建项目，再通过
`./target/debug/hello_cargo` 运行它。也可以使用 `cargo run`，用一条命令编译
代码并运行生成的可执行文件：

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

使用 `cargo run` 比先运行 `cargo build`、再输入二进制文件的完整路径更方便，
所以大多数开发者都会使用 `cargo run`。

注意，这一次没有出现表示 Cargo 正在编译 `hello_cargo` 的输出。Cargo 发现
文件没有变化，所以没有重新构建，而是直接运行二进制文件。如果修改了源代码，
Cargo 会在运行前重新构建项目，你将看到以下输出：

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo 还提供了一个名为 `cargo check` 的命令。它可以快速检查代码能否编译，但
不会生成可执行文件：

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

为什么会不想生成可执行文件呢？`cargo check` 往往比 `cargo build` 快得多，
因为它跳过了生成可执行文件的步骤。如果在编写代码时不断检查工作成果，使用
`cargo check` 能更快确认项目是否仍可编译！因此，许多 Rustacean 会在编写
程序的过程中定期运行 `cargo check`；准备使用可执行文件时，再运行
`cargo build`。

下面回顾目前学到的 Cargo 知识：

- 可以使用 `cargo new` 创建项目。
- 可以使用 `cargo build` 构建项目。
- 可以使用 `cargo run` 一步完成项目的构建和运行。
- 可以使用 `cargo check` 构建项目并检查错误，但不生成二进制文件。
- Cargo 不会把构建结果保存在代码所在目录，而是存放在 <em>target/debug</em> 目录。

使用 Cargo 还有一个好处：无论使用哪种操作系统，命令都完全相同。因此，从
这里开始，我们不再分别提供适用于 Linux、macOS 和 Windows 的具体说明。

### 发布构建

项目最终准备好发布时，可以使用 `cargo build --release` 启用优化并进行编译。
这条命令会在 <em>target/release</em> 而不是 <em>target/debug</em> 中创建可执行文件。优化
可以让 Rust 代码运行得更快，但启用优化会延长程序编译所需的时间。因此需要
两种不同的<em>配置档(profile)</em>：一种用于开发，此时希望经常快速地重新构建；另一种
用于构建最终交付给用户的程序，它不会反复重新构建，但需要尽可能快地运行。
如果要对代码的运行时间进行基准测试，请务必运行 `cargo build --release`，并
使用 <em>target/release</em> 中的可执行文件进行测试。

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### 利用 Cargo 的惯例

对于简单项目，与只使用 `rustc` 相比，Cargo 的价值并不明显；但程序变得更加
复杂时，它便会证明自己的价值。程序增长到包含多个文件或需要依赖以后，让
Cargo 协调构建会容易得多。

尽管 `hello_cargo` 项目很简单，它现在已经使用了你在此后 Rust 开发生涯中会
用到的许多实际工具。事实上，要处理任何现有项目，都可以使用以下命令通过 Git
检出代码、切换到项目目录并进行构建：

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

有关 Cargo 的更多信息，请查阅[其文档][cargo]。

## 小结

你的 Rust 之旅已经有了一个很棒的开端！本章介绍了如何：

- 使用 `rustup` 安装最新版的稳定版 Rust。
- 更新到较新的 Rust 版本。
- 打开本地安装的文档。
- 直接使用 `rustc` 编写并运行“Hello, world!”程序。
- 按照 Cargo 的惯例创建并运行新项目。

现在正适合构建一个更具规模的程序，以便习惯阅读和编写 Rust 代码。因此，
我们会在第 2 章中编写一个猜数字游戏。如果更愿意先学习常见编程概念在 Rust
中的工作方式，可以先阅读第 3 章，再回到第 2 章。

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
