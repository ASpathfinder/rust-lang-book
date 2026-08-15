# 一个 I/O 项目：构建命令行程序

本章会回顾你目前学到的许多技能，并探索更多标准库功能。我们将构建一个与文件和命令行输入/输出交互的命令行工具，以练习你已经掌握的一些 Rust 概念。

Rust 速度快、安全、能生成单一二进制文件并支持跨平台，因此是创建命令行工具的理想语言。这个项目会制作经典命令行搜索工具 `grep`（<strong>g</strong>lobally search a <strong>r</strong>egular <strong>e</strong>xpression and <strong>p</strong>rint，全局搜索正则表达式并打印）的自有版本。在最简单的用法中，`grep` 会在指定文件中搜索指定字符串。为此，`grep` 接收文件路径和字符串作为实参，然后读取文件，在其中查找包含该字符串实参的行，并打印这些行。

在此过程中，我们会展示如何让命令行工具使用许多其他命令行工具都采用的终端功能。我们会读取环境变量的值，让用户可以配置工具的行为。还会把错误信息打印到<em>标准错误(standard error)</em>控制台流（`stderr`），而不是<em>标准输出(standard output)</em>（`stdout`）。这样，例如用户就可以把成功输出重定向到文件，同时仍在屏幕上看到错误信息。

Rust 社区成员 Andrew Gallant 已经创建了功能完备、速度极快的 `grep` 版本，名为 `ripgrep`。相比之下，我们的版本会很简单，但本章会提供理解 `ripgrep` 这类现实项目所需的一些背景知识。

我们的 `grep` 项目会结合目前学到的多个概念：

- 组织代码（[第 7 章][ch7]）
- 使用向量和字符串（[第 8 章][ch8]）
- 处理错误（[第 9 章][ch9]）
- 在适当时使用特征和生命周期（[第 10 章][ch10]）
- 编写测试（[第 11 章][ch11]）

我们还会简要介绍闭包、迭代器和特征对象，[第 13 章][ch13]和[第 18 章][ch18]会详细讲解它们。

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
