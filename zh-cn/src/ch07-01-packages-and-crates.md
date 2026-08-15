## 包与 crate

首先介绍模块系统中的包和 crate。

<em>crate</em> 是 Rust 编译器一次处理的最小代码单位。即使运行 `rustc` 而不是
`cargo`，只传入一个源代码文件（如第 1 章[“Rust 程序基础”][basics]
<!-- ignore -->一节所做），编译器也会把该文件视为 crate。crate 可以包含模块，
模块又可以定义在其他文件中，并与 crate 一起编译，后续章节会看到。

crate 有两种形式：二进制 crate 或库 crate。<em>二进制 crate(binary crate)</em>是
可以编译为可运行可执行文件的程序，例如命令行程序或服务器。每个二进制 crate
都必须有一个名为 `main` 的函数，定义可执行文件运行时发生什么。目前创建的所有
crate 都是二进制 crate。

<em>库 crate(library crate)</em>没有 `main` 函数，也不会编译成可执行文件，而是定义
供多个项目共享的功能。例如，第 2 章使用的 [`rand` crate][rand]<!-- ignore -->
提供随机数生成功能。Rust 程序员大多数时候说“crate”时指的是库 crate，并把
“crate”与一般编程概念中的“库”交替使用。

<em>crate 根(crate root)</em>是 Rust 编译器开始处理的源文件，它构成 crate 的根模块
（[“使用模块控制作用域与私有性”][modules]<!-- ignore -->一节会深入解释模块）。

<em>包(package)</em>是提供一组功能的一个或多个 crate 的集合。包包含
<em>Cargo.toml</em>，描述如何构建这些 crate。Cargo 本身就是一个包，其中包含你一直
用于构建代码的命令行工具二进制 crate；Cargo 包也包含这个二进制 crate 所依赖的
库 crate。其他项目可以依赖 Cargo 的库 crate，使用与 Cargo 命令行工具相同的
逻辑。

包可以包含任意数量的二进制 crate，但最多只能包含一个库 crate。包至少必须包含
一个 crate，无论是库还是二进制 crate。

下面逐步看看创建包时会发生什么。首先输入 `cargo new my-project`：

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

运行命令后，用 `ls` 查看 Cargo 创建的内容。<em>my-project</em> 目录中有
<em>Cargo.toml</em>，说明我们得到一个包；还有包含 <em>main.rs</em> 的 <em>src</em> 目录。
在编辑器中打开 <em>Cargo.toml</em>，会发现其中没有提到 <em>src/main.rs</em>。Cargo
遵循一项约定：<em>src/main.rs</em> 是与包同名的二进制 crate 的 crate 根。同样，
如果包目录含有 <em>src/lib.rs</em>，Cargo 就知道包中包含一个与包同名的库 crate，
并把该文件视为 crate 根。Cargo 将 crate 根文件传给 `rustc`，构建库或二进制。

这里的包只含 <em>src/main.rs</em>，因此只包含名为 `my-project` 的二进制 crate。
如果包同时包含 <em>src/main.rs</em> 和 <em>src/lib.rs</em>，它就有两个与包同名的 crate：
一个二进制 crate 和一个库 crate。把文件放入 <em>src/bin</em> 目录，可让包拥有多个
二进制 crate；每个文件都是独立的二进制 crate。

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
