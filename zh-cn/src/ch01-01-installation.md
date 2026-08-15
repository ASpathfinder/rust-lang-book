<a id="installation"></a>

## 安装

第一步是安装 Rust。我们会通过 `rustup` 下载 Rust；`rustup` 是一个用于管理
Rust 版本及相关工具的<em>命令行工具(command-line tool)</em>。下载时需要连接互联网。

> 注意：如果你出于某种原因不想使用 `rustup`，请参阅[其他 Rust 安装方式
> 页面][otherinstall]，了解更多选择。

下面的步骤会安装最新版的稳定版 Rust <em>编译器(compiler)</em>。<em>Rust 的稳定性保证
(stability guarantees)</em>意味着，本书中能够编译的所有示例，在更新版本的 Rust
中仍然可以编译。由于 Rust 经常改进错误信息和警告，不同版本的输出可能略有
差异。换句话说，只要按照这些步骤安装较新的 Rust 稳定版，就应该能够正常学习
本书内容。

> ### 命令行表示法
>
> 在本章乃至全书中，我们会展示一些在终端中使用的命令。需要你在终端输入的行
> 都以 `$` 开头。你不必输入 `$` 字符；它只是命令行提示符，用来表示一条命令
> 的开始。不以 `$` 开头的行通常是上一条命令的输出。此外，PowerShell 专用的
> 示例会使用 `>`，而不是 `$`。

### 在 Linux 或 macOS 上安装 `rustup`

如果你使用 Linux 或 macOS，请打开终端并输入以下命令：

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

这条命令会下载一个脚本并开始安装 `rustup` 工具，后者会安装最新版的稳定版
Rust。系统可能会要求你输入密码。如果安装成功，将显示下面这行文字：

```text
Rust is installed now. Great!
```

你还需要一个<em>链接器(linker)</em>，Rust 会用这个程序把编译产生的内容合并成一个
文件。你的系统很可能已经安装了链接器。如果遇到链接器错误，应安装一个 C
编译器，它通常会包含链接器。C 编译器也很有用，因为一些常见的 <em>Rust 包
(package)</em>依赖 C 代码，需要使用 C 编译器。

在 macOS 上，可以运行以下命令获取 C 编译器：

```console
$ xcode-select --install
```

Linux 用户通常应该按照自己所用发行版的文档安装 GCC 或 Clang。例如，如果
使用 Ubuntu，可以安装 `build-essential` 包。

### 在 Windows 上安装 `rustup`

在 Windows 上，请访问
[https://www.rust-lang.org/tools/install][install]<!-- ignore -->，并按照
说明安装 Rust。在安装过程中的某个阶段，系统会提示你安装 Visual Studio。
它提供编译程序所需的链接器和原生库。如果需要关于这一步的更多帮助，请参阅
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!--
ignore -->。

本书其余部分使用的命令在 <em>cmd.exe</em> 和 PowerShell 中都可以运行。如有具体
差异，我们会说明应该使用哪一种命令。

<a id="troubleshooting"></a>

### 故障排查

要检查 Rust 是否已正确安装，请打开 shell 并输入：

```console
$ rustc --version
```

你应该会看到最新稳定版的版本号、提交哈希和提交日期，格式如下：

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

如果看到这些信息，就说明 Rust 已成功安装！如果没有看到，请按下面的方法检查
Rust 是否位于 `%PATH%` 系统变量中。

在 Windows CMD 中使用：

```console
> echo %PATH%
```

在 PowerShell 中使用：

```powershell
> echo $env:Path
```

在 Linux 和 macOS 中使用：

```console
$ echo $PATH
```

如果这些设置都正确，但 Rust 仍不能运行，你可以从许多地方获得帮助。请在
[社区页面][community]查看如何联系其他 Rustacean（这是我们对自己的一个有趣
称呼）。

### 更新和卸载

通过 `rustup` 安装 Rust 后，更新到新发布的版本非常简单。在 shell 中运行
下面的更新命令：

```console
$ rustup update
```

要卸载 Rust 和 `rustup`，请在 shell 中运行下面的卸载命令：

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### 阅读本地文档

安装 Rust 时也会安装一份本地文档，供你离线阅读。运行 `rustup doc` 即可在
浏览器中打开本地文档。

每当标准库提供了某个类型或函数，而你不确定它有什么作用或该怎样使用时，
都可以查阅<em>应用程序编程接口(API)</em>文档！

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### 使用文本编辑器和 IDE

本书不限定你使用哪种工具编写 Rust 代码。几乎任何文本编辑器都能完成这项
工作！不过，许多文本编辑器和<em>集成开发环境(IDE)</em>已经内置 Rust 支持。你随时
可以在 Rust 网站的[工具页面][tools]找到一份比较新的编辑器和 IDE 列表。

### 离线学习本书

在若干示例中，我们会使用标准库以外的 Rust 包。要完成这些示例，你需要连接
互联网，或者提前下载相应的<em>依赖(dependencies)</em>。可以运行以下命令预先下载
依赖。（我们稍后会详细解释 `cargo` 是什么，以及每条命令的作用。）

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.10.1 trpl@0.2.0
```

这些命令会缓存相关包的下载内容，之后便不必再次下载。运行命令后，无需保留
`get-dependencies` 文件夹。完成缓存后，在本书后续的所有 `cargo` 命令中都
可以使用 `--offline` 标志，让 Cargo 使用这些缓存版本，而不尝试访问网络。

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
