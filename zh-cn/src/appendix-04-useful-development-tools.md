## 附录 D：实用开发工具

本附录介绍 Rust 项目提供的一些实用开发工具。我们会了解自动格式化、快速应用警告修复、代码检查工具，以及 IDE 集成。

### 使用 `rustfmt` 自动格式化

`rustfmt` 工具会根据社区代码风格重新格式化代码。许多协作项目使用 `rustfmt`，避免在编写 Rust 时争论应采用哪种风格：每个人都用该工具格式化代码。

Rust 安装默认包含 `rustfmt`，所以系统中应该已经有 `rustfmt` 和 `cargo-fmt` 程序。这两个命令之间的关系类似于 `rustc` 与 `cargo`：`rustfmt` 可以进行更细粒度的控制，而 `cargo-fmt` 理解使用 Cargo 的项目惯例。要格式化任意 Cargo 项目，请输入：

```console
$ cargo fmt
```

运行这条命令会重新格式化当前 crate 中的所有 Rust 代码。它应该只改变代码风格，不改变代码语义。有关 `rustfmt` 的更多信息，请参阅[其文档][rustfmt]。

### 使用 `rustfix` 修复代码

Rust 安装中包含 `rustfix` 工具，它可以自动修复那些有明确解决办法、而该办法很可能正是你所需的编译器警告。你很可能已经见过编译器警告。例如，来看下面的代码：

<span class="filename">文件名：src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

这里把变量 `x` 定义为可变，但实际上从未改变它。Rust 会对此发出警告：

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

警告建议删除 `mut` 关键字。运行 `cargo fix` 命令，可以使用 `rustfix` 工具自动应用这条建议：

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

再次查看 <em>src/main.rs</em>，会看到 `cargo fix` 已经修改了代码：

<span class="filename">文件名：src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

变量 `x` 现在不可变，警告也不再出现。

还可以使用 `cargo fix` 命令在不同 Rust 版本之间迁移代码。版本相关内容见[附录 E][editions]<!-- ignore -->。

### 使用 Clippy 获取更多 Lint

Clippy 工具是一组用于分析代码的 <em>lint</em>，可以帮助你发现常见错误并改进 Rust 代码。标准 Rust 安装包含 Clippy。

要对任意 Cargo 项目运行 Clippy 的 lint，请输入：

```console
$ cargo clippy
```

例如，假设你编写了一个使用数学常量近似值（比如圆周率）的程序，如下所示：

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

在这个项目上运行 `cargo clippy` 会得到以下错误：

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

这个错误告诉你，Rust 已经定义了更精确的 `PI` 常量，改用该常量会让程序更加正确。随后，可以修改代码以使用 `PI` 常量。

下面的代码不会产生任何 Clippy 错误或警告：

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

有关 Clippy 的更多信息，请参阅[其文档][clippy]。

### 使用 `rust-analyzer` 集成 IDE

为了帮助集成 IDE，Rust 社区建议使用 [`rust-analyzer`][rust-analyzer]<!-- ignore -->。这个工具是一组以编译器为中心、支持[<em>语言服务器协议(Language Server Protocol, LSP)</em>][lsp]<!-- ignore -->的实用工具；该协议是一项用于 IDE 与编程语言相互通信的规范。不同客户端都可以使用 `rust-analyzer`，例如 [Visual Studio Code 的 Rust Analyzer 插件][vscode]。

请访问 `rust-analyzer` 项目的[主页][rust-analyzer]<!-- ignore -->获取安装说明，然后在你所用的 IDE 中安装语言服务器支持。IDE 将获得自动补全、跳转到定义和内联错误等功能。

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.html
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
