# 编写猜数字游戏

让我们通过亲手完成一个项目来进入 Rust 的世界！本章会在一个实际程序中使用
若干常见的 Rust 概念，带你初步认识 `let`、`match`、方法、关联函数、外部
crate 等内容。后续章节将更详细地探讨这些概念；本章只需练习基础知识。

我们将实现一个经典的编程入门题目：猜数字游戏。游戏规则如下：程序生成一个
1 到 100 之间的随机整数，然后提示玩家输入猜测。玩家输入后，程序会指出数字
太小还是太大。如果猜对了，游戏会打印祝贺信息并退出。

## 新建项目

进入第 1 章创建的 <em>projects</em> 目录，使用 Cargo 新建项目：

```console
$ cargo new guessing_game
$ cd guessing_game
```

第一条命令 `cargo new` 接收项目名称 `guessing_game` 作为第一个实参；第二条
命令进入新项目的目录。

查看生成的 <em>Cargo.toml</em> 文件：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">文件名：Cargo.toml</span>

```toml
{{#include ../../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

正如第 1 章所见，`cargo new` 会生成一个“Hello, world!”程序。查看
<em>src/main.rs</em>：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

现在使用 `cargo run`，一步完成这个程序的编译和运行：

```console
{{#include ../../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

当项目需要快速<em>迭代(iteration)</em>时，`run` 命令非常方便；在这个游戏的开发中，
我们会在进入下一次迭代前迅速测试每次修改。

重新打开 <em>src/main.rs</em>。本章所有代码都会写在这个文件中。

## 处理一次猜测

猜数字程序的第一部分将请求用户输入、处理输入，并检查输入是否符合预期格式。
首先允许玩家输入一个猜测。把示例 2-1 的代码写入 <em>src/main.rs</em>。

<Listing number="2-1" file-name="src/main.rs" caption="获取并打印用户猜测的代码">

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

这段代码包含许多信息，下面逐行分析。为了取得用户输入并打印结果，需要把输入
输出库 `io` 引入<em>作用域(scope)</em>。`io` 来自名为 `std` 的标准库：

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Rust 默认会把标准库中定义的一组条目引入每个程序的作用域。这组条目称为
<em>预导入模块(prelude)</em>，可以在[标准库文档][prelude]中查看其全部内容。

如果要使用的类型不在预导入模块中，就必须使用 `use` 语句显式地将其引入
作用域。`std::io` 库提供了许多实用功能，包括接收用户输入的能力。

第 1 章介绍过，`main` 函数是程序的入口点：

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` 语法声明一个新函数；圆括号 `()` 表示没有形参；左花括号 `{` 开始函数体。

第 1 章还介绍过，`println!` 是一个在屏幕上打印字符串的宏：

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

这行代码打印一条提示，说明游戏内容并请求用户输入。

<a id="storing-values-with-variables"></a>

### 使用变量存储值

接下来创建一个<em>变量(variable)</em>来存储用户输入：

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

程序开始变得有趣了！这一小行代码做了不少事情。我们用 `let` 语句创建变量。
再看一个例子：

```rust,ignore
let apples = 5;
```

这行代码创建名为 `apples` 的变量，并把它<em>绑定(binding)</em>到值 `5`。Rust 变量
默认<em>不可变(immutable)</em>，即变量一旦获得一个值，该值就不会改变。第 3 章的
[“变量与可变性”][variables-and-mutability]<!-- ignore -->一节会详细讨论这个
概念。要使变量<em>可变(mutable)</em>，需在变量名前加上 `mut`：

```rust,ignore
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> 注意：`//` 语法开始一条延续到行末的注释，Rust 会忽略注释中的所有内容。
> [第 3 章][comments]<!-- ignore -->会更详细地讨论注释。

回到猜数字程序，现在可以看出，`let mut guess` 引入了一个名为 `guess` 的可变
变量。等号 `=` 告诉 Rust，我们现在要把某个值绑定到这个变量。等号右边是
`guess` 所绑定的值：调用 `String::new` 的结果。这个函数会返回 `String` 的
一个新的<em>实例(instance)</em>。[`String`][string]<!-- ignore --> 是标准库提供的字符串
类型，可增长并采用 UTF-8 编码。

`::new` 中的 `::` 语法表示 `new` 是 `String` 类型的<em>关联函数
(associated function)</em>。关联函数是在某个类型（这里是 `String`）上实现的
函数。这个 `new` 函数创建一个新的空字符串。许多类型都有名为 `new` 的函数，
因为这是创建某类新值的函数的常用名称。

完整来看，`let mut guess = String::new();` 创建了一个可变变量，并把它绑定到
一个新的空 `String` 实例。呼！

### 接收用户输入

程序第一行使用 `use std::io;` 引入了标准库的输入输出功能。现在调用 `io`
模块的 `stdin` 函数来处理用户输入：

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

即使没有在程序开头用 `use std::io;` 导入 `io` 模块，也可以把函数调用写成
`std::io::stdin`。`stdin` 返回 [`std::io::Stdin`][iostdin]<!-- ignore --> 的
一个实例；该类型代表终端标准输入的<em>句柄(handle)</em>。

接着，`.read_line(&mut guess)` 在标准输入句柄上调用
[`read_line`][read_line]<!-- ignore --> 方法，以取得用户输入。我们还把
`&mut guess` 作为实参传给 `read_line`，告诉它用哪个字符串存储用户输入。
`read_line` 的完整工作，是把用户在标准输入中键入的内容追加到一个字符串中
（而不是覆盖原有内容），因此要把该字符串作为实参传入。字符串实参必须可变，
方法才能修改其内容。

`&` 表明这个实参是<em>引用(reference)</em>。引用让代码的多个部分能够访问同一份数据，
而不必在内存中反复复制。引用是一项复杂的功能；能够安全、轻松地使用引用，是
Rust 的主要优势之一。完成这个程序不需要掌握太多细节。目前只需知道，引用和
变量一样默认不可变，因此要写 `&mut guess` 而不是 `&guess`，使其可变。第 4
章会更深入地解释引用。

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

<a id="handling-potential-failure-with-result"></a>

### 使用 `Result` 处理潜在失败

我们仍在分析同一行代码。虽然现在讨论的是第三行文本，但从逻辑上看，它依然是
同一行代码的一部分。下一部分是这个方法：

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

这段代码也可以写成：

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

但长行难以阅读，最好将其拆开。用 `.method_name()` 语法调用方法时，通常适合
插入换行和其他空白来分隔长行。下面讨论这段代码的作用。

前面提到，`read_line` 会把用户输入放入传给它的字符串，同时还返回一个
`Result` 值。[`Result`][result]<!-- ignore --> 是一种[<em>枚举(enumeration)</em>][enums]<!-- ignore -->，通常
简称为 <em>enum</em>，即可以处于多种可能状态之一的类型。每一种可能状态称为变体
<em>变体(variant)</em>。[第 6 章][enums]<!-- ignore -->会更详细地介绍枚举。`Result` 类型用于编码错误处理信息。

`Result` 的变体是 `Ok` 和 `Err`。`Ok` 表示操作成功，其中包含成功生成的值；
`Err` 表示操作失败，其中包含失败原因等信息。

与其他类型的值一样，`Result` 类型的值也定义了方法。`Result` 实例提供可调用
的 [`expect` 方法][expect]<!-- ignore -->。如果这个实例是 `Err`，`expect` 会
让程序崩溃，并显示作为实参传入的消息。`read_line` 返回 `Err` 时，很可能是
底层操作系统发生了错误。如果实例是 `Ok`，`expect` 会取出 `Ok` 保存的返回值
并交给你使用。在这里，该值是用户输入的字节数。

如果不调用 `expect`，程序仍能编译，但会收到警告：

```console
{{#include ../../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust 警告我们没有使用 `read_line` 返回的 `Result`，这说明程序没有处理一个
潜在错误。

消除警告的正确方式，是实际编写错误处理代码。但这里遇到问题时只需让程序崩溃，
所以可以使用 `expect`。[第 9 章][recover]<!-- ignore -->会介绍如何从错误中恢复。

### 使用 `println!` 占位符打印值

除右花括号外，目前的代码只剩一行需要讨论：

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

这行代码打印现在保存着用户输入的字符串。花括号 `{}` 是<em>占位符
(placeholder)</em>：可以把它想成一对小螃蟹钳，把值固定在相应位置。打印变量值时，
可以把变量名放在花括号中。打印表达式求值结果时，要在格式字符串中放置空花
括号，然后在格式字符串后按相同顺序列出由逗号分隔、分别对应各占位符的表达式。
在一次 `println!` 调用中同时打印变量和表达式结果，可以这样写：

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

这段代码会打印 `x = 5 and y + 2 = 12`。

### 测试第一部分

使用 `cargo run` 测试猜数字游戏的第一部分：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

至此，游戏的第一部分已经完成：程序从键盘取得输入，然后将其打印出来。

## 生成秘密数字

接下来，需要生成一个供用户猜测的秘密数字。秘密数字每次都应该不同，游戏才
适合反复游玩。我们使用 1 到 100 之间的随机数，以免游戏太难。Rust 标准库
尚未包含随机数功能，不过 Rust 团队提供了带有这一功能的 [`rand`
crate][randcrate]。

<!-- Old headings. Do not remove or links may break. -->
<a id="using-a-crate-to-get-more-functionality"></a>

### 使用 crate 扩展功能

crate 是 Rust 源代码文件的集合。我们正在构建的项目是<em>二进制 crate
(binary crate)</em>，也就是一个可执行程序。`rand` 是<em>库 crate(library crate)</em>，
其中包含供其他程序使用、不能单独执行的代码。

Cargo 在协调外部 crate 方面尤其出色。编写使用 `rand` 的代码之前，需要修改
<em>Cargo.toml</em>，把 `rand` 加入依赖。打开该文件，在 Cargo 创建的
`[dependencies]` 标题下面添加以下一行。务必完全按照这里的版本号指定 `rand`，
否则本教程的代码示例可能无法工作：

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">文件名：Cargo.toml</span>

```toml
{{#include ../../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

在 <em>Cargo.toml</em> 中，一个标题后的所有内容都属于该章节，直到下一个章节开始。
在 `[dependencies]` 中，要告诉 Cargo 项目依赖哪些外部 crate，以及需要哪些
版本。这里用语义化版本说明符 `0.10.1` 指定 `rand`。Cargo 理解[<em>语义化版本
(Semantic Versioning)</em>][semver]<!-- ignore -->（有时简称 <em>SemVer</em>），这是一套
书写版本号的标准。说明符 `0.10.1` 实际是 `^0.10.1` 的简写，表示版本至少为
0.10.1，但低于 0.11.0。

Cargo 认为这些版本提供与 0.10.1 兼容的公开 API；这项约束保证你会取得仍可
编译本章代码的最新补丁版本。0.11.0 或更高版本不保证拥有后续示例所用的同一
API。

现在不修改任何代码，按照示例 2-2 构建项目。

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="将 `rand` crate 添加为依赖后运行 `cargo build` 的输出">

```console
$ cargo build
    Updating crates.io index
     Locking 8 packages to latest Rust 1.96.0 compatible versions
  Downloaded rand_core v0.10.1
  Downloaded chacha20 v0.10.1
  Downloaded rand v0.10.1
  Downloaded 3 crates (162.9KiB) in 0.59s
   Compiling libc v0.2.186
   Compiling rand_core v0.10.1
   Compiling getrandom v0.4.3
   Compiling cfg-if v1.0.4
   Compiling chacha20 v0.10.1
   Compiling rand v0.10.1
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.03s
```

</Listing>

你看到的版本号可能不同（但由于 SemVer，它们都会与代码兼容），不同操作系统
显示的行也可能不同，行的顺序也可能变化。

加入外部依赖时，Cargo 会从<em>注册表(registry)</em>获取该依赖所需各项内容的最新
版本。注册表是 [Crates.io][cratesio] 数据的副本；Rust 生态系统的参与者会在
Crates.io 发布开源 Rust 项目，供他人使用。

更新注册表后，Cargo 检查 `[dependencies]`，下载其中尚未下载的 crate。虽然
这里只列出 `rand`，Cargo 也会取得 `rand` 工作所需的其他依赖。下载完成后，
Rust 先编译这些 crate，再在依赖可用的情况下编译项目。

如果不作任何修改便立即再次运行 `cargo build`，除了 `Finished` 行外不会看到
其他输出。Cargo 知道依赖已经下载和编译，<em>Cargo.toml</em> 没有改变，代码也没有
改变，所以无需重新编译任何内容，直接退出。

如果打开 <em>src/main.rs</em>，作一处微小修改，保存后再次构建，只会看到两行输出：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

这说明 Cargo 只使用 <em>src/main.rs</em> 中的小改动更新构建。依赖没有变化，所以可
复用此前下载和编译的内容。

<!-- Old headings. Do not remove or links may break. -->
<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### 确保构建可复现

Cargo 提供一种机制，确保你或其他人每次都能重新构建出相同的产物：除非明确
要求，否则 Cargo 只会使用已经确定的依赖版本。例如，假设下周发布了 `rand`
0.10.2，其中包含一个重要错误修复，但也引入了会破坏代码的回归。为处理这种
情况，第一次运行 `cargo build` 时，Rust 会创建 <em>Cargo.lock</em>；现在
<em>guessing_game</em> 目录中已经有了这个文件。

首次构建项目时，Cargo 找出所有符合条件的依赖版本，并将其写入 <em>Cargo.lock</em>。
以后构建时，Cargo 发现该文件存在，就会使用其中指定的版本，而不重新解析版本。
这样会自动得到<em>可复现构建(reproducible build)</em>。换句话说，在你明确升级之前，
项目会因 <em>Cargo.lock</em> 而停留在 0.10.1。由于该文件对构建复现很重要，通常会
和项目的其他代码一起提交到版本控制系统。

#### 将 crate 更新到新版本

需要更新 crate 时，Cargo 提供 `update` 命令。它会忽略 <em>Cargo.lock</em>，找出
符合 <em>Cargo.toml</em> 约束的所有最新版本，再把这些版本写入 <em>Cargo.lock</em>。默认
情况下，Cargo 只会查找高于 0.10.1 且低于 0.11.0 的版本。假设 `rand` 发布
了 0.10.2 和 0.999.0，运行 `cargo update` 会看到：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.96.0 compatible version
    Updating rand v0.10.1 -> v0.10.2 (available: v0.999.0)
```

Cargo 会忽略 0.999.0。此时 <em>Cargo.lock</em> 也会变化，记录项目现在使用 `rand`
0.10.2。要使用 0.999.0 或 0.999.<em>x</em> 系列中的版本，就必须把 <em>Cargo.toml</em>
改成下面这样（不要真的修改，因为后续示例假定使用 `rand` 0.10）：

```toml
[dependencies]
rand = "0.999.0"
```

下次运行 `cargo build` 时，Cargo 会更新可用 crate 的注册表，并按照指定的新
版本重新评估 `rand` 要求。

[Cargo][doccargo]<!-- ignore -->及其[生态系统][doccratesio]<!-- ignore -->还有
许多内容可讲，第 14 章会继续讨论；目前掌握这些就够了。Cargo 让复用库变得
十分容易，Rustacean 因而可以把多个包组合成更小的项目。

<a id="generating-a-random-number"></a>

### 生成随机数

现在开始使用 `rand` 生成待猜数字。下一步按照示例 2-3 更新 <em>src/main.rs</em>。

<Listing number="2-3" file-name="src/main.rs" caption="添加生成随机数的代码">

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

首先添加 `use rand::prelude::*;`。`prelude` 模块包含 `rand` crate 中最常用的
部分，`use` 让这些条目在程序作用域中可用。

接着在中间添加两行。第一行调用 `rand::rng`，取得要使用的随机数生成器：它
属于当前执行线程，并由操作系统提供种子。然后在随机数生成器上调用
`random_range`。该方法由 `rand::prelude` 中的 `RngExt` trait 定义，而我们
已经用 `use rand::prelude::*;` 将其引入作用域。`random_range` 接收一个范围
表达式并生成范围内的随机数。这里的范围表达式形式为 `start..=end`，同时包含
上下界，因此用 `1..=100` 请求 1 到 100 之间的数字。

> 注意：你不可能凭空知道应该从 crate 引入哪些内容、调用哪些方法和函数，
> 所以每个 crate 都有使用文档。Cargo 的另一个便利功能是：运行
> `cargo doc --open` 会在本地构建所有依赖提供的文档，并在浏览器中打开。
> 例如，要了解 `rand` 的其他功能，可以运行该命令，再点击左侧边栏中的
> `rand`。

第二个新增行会打印秘密数字。这在开发期间有助于测试，但最终版本会删除它。
程序一启动就给出答案，可算不上什么游戏！

尝试多运行几次程序：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

每次应该得到不同的随机数，而且都位于 1 到 100 之间。警告可以安全地忽略。
如果出现错误，请检查 <em>Cargo.toml</em> 中是否有 `rand = "0.10.1"`；未来版本的
`rand` 可能使用不同 API，但 0.10 系列中的任何版本都应适用于本章代码。

<a id="comparing-the-guess-to-the-secret-number"></a>

## 比较猜测与秘密数字

现在已经有了用户输入和随机数，可以比较二者。示例 2-4 展示了这一步。请注意，
这段代码暂时还不能编译，原因稍后解释。

<Listing number="2-4" file-name="src/main.rs" caption="处理两个数字比较后可能得到的返回值">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

首先添加另一个 `use` 语句，把标准库中的 `std::cmp::Ordering` 类型引入作用域。
`Ordering` 也是枚举，拥有 `Less`、`Greater` 和 `Equal` 三个变体，分别对应
比较两个值时的三种可能结果。

然后在底部添加五行使用 `Ordering` 的代码。`cmp` 方法比较两个值，可以在
任何可比较的值上调用；它接收要比较对象的引用，这里比较的是 `guess` 和
`secret_number`。它返回此前用 `use` 引入的 `Ordering` 枚举变体。我们使用
[`match`][match]<!-- ignore --> 表达式，根据 `cmp` 返回的变体决定下一步操作。

`match` 表达式由<em>分支(arm)</em>组成。每个分支包含一个用于匹配的<em>模式(pattern)</em>，
以及传给 `match` 的值符合该模式时要运行的代码。Rust 取得交给 `match` 的值，
依次检查各分支的模式。模式和 `match` 是强大的 Rust 功能：它们能够表达代码
可能遇到的各种情况，并确保你处理所有情况。第 19 章和第 6 章会分别详细介绍
模式和 `match`。

以这里的 `match` 为例，假设用户猜 50，而这次生成的秘密数字是 38。

比较 50 与 38 时，`cmp` 会返回 `Ordering::Greater`。`match` 取得这个值，
开始检查各分支模式。第一个模式 `Ordering::Less` 不匹配，所以忽略该分支并
继续。下一个模式 `Ordering::Greater` 与值匹配，于是执行相应代码，在屏幕上
打印 `Too big!`。`match` 在第一次成功匹配后结束，因此不会再检查最后一个
分支。

不过，示例 2-4 目前还不能编译。试试看：

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

错误的核心是<em>类型不匹配(mismatched types)</em>。Rust 拥有强静态类型系统，也
支持<em>类型推断(type inference)</em>。写下 `let mut guess = String::new()` 时，Rust
能够推断 `guess` 应为 `String`，无需显式标注类型。而 `secret_number` 是数值
类型。Rust 的多种数值类型都能保存 1 到 100，例如 32 位的 `i32`、无符号
32 位的 `u32`、64 位的 `i64` 等。除非另有说明，Rust 默认采用 `i32`，因此
如果其他地方没有类型信息使 Rust 作出不同推断，`secret_number` 就是 `i32`。
错误的原因是 Rust 无法比较字符串和数值类型。

最终要把程序读取的 `String` 转换为数值类型，才能按数值和秘密数字比较。在
`main` 函数体中添加：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

这一行是：

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

我们创建了名为 `guess` 的变量。可是程序不是已经有同名变量了吗？确实有，但
Rust 允许新变量<em>遮蔽(shadow)</em> `guess` 的旧值。<em>遮蔽(shadowing)</em>让我们复用
`guess` 这个名称，而不必创建 `guess_str` 和 `guess` 等两个不同变量。[第 3 章][shadowing]<!-- ignore -->
会详细介绍；目前只需知道，需要把值从一种类型转换为另一种类型时经常使用它。

新变量绑定到表达式 `guess.trim().parse()`。表达式中的 `guess` 指向保存字符串
输入的原变量。`String` 实例的 `trim` 会去掉首尾空白；只有这样才能把字符串
转换成只能包含数值数据的 `u32`。用户必须按 <kbd>enter</kbd> 才能让 `read_line`
读取猜测，这会向字符串添加换行符。例如输入 <kbd>5</kbd> 并按
<kbd>enter</kbd> 后，`guess` 是 `5\n`，其中 `\n` 表示换行（Windows 上会得到
回车加换行 `\r\n`）。`trim` 去掉 `\n` 或 `\r\n`，只留下 `5`。

字符串的 [`parse` 方法][parse]<!-- ignore -->把字符串转换为另一种类型。这里
把字符串转换为数字。必须用 `let guess: u32` 告诉 Rust 所需的确切数值类型。
`guess` 后的冒号 `:` 表示要标注变量类型。Rust 内置多种数值类型；这里的
`u32` 是 32 位无符号整数，是较小正数的良好默认选择。[第 3 章][integers]<!-- ignore -->会介绍其他数值
类型。

示例中的 `u32` 标注和与 `secret_number` 的比较，也让 Rust 推断后者应为
`u32`。现在比较的是两个相同类型的值。

`parse` 只适用于逻辑上可转换为数字的字符，因此很容易失败。例如字符串包含
`A👍%` 时，就无法转换。由于可能失败，`parse` 和 `read_line` 一样返回
`Result`（前面的[“使用 `Result` 处理潜在失败”](#handling-potential-failure-with-result)<!-- ignore -->一节讨论过）。我们再次使用 `expect` 处理它。如果 `parse` 无法从字符串创建数字，
就返回 `Err`，`expect` 会让游戏崩溃并打印给定消息。转换成功时，`parse` 返回
`Ok`，`expect` 再从中取出所需的数字。

现在运行程序：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

不错！即使猜测前带有空格，程序仍能判断用户猜的是 76。多运行几次，验证不同
输入下的行为：猜对数字、猜得太大，以及猜得太小。

游戏大部分功能已经完成，但用户只能猜一次。下面添加循环来改变这一点。

## 使用循环允许多次猜测

`loop` 关键字创建<em>无限循环(infinite loop)</em>。添加循环，让用户有更多猜测机会：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

可以看到，从输入猜测的提示开始，所有内容都移入了循环。务必让循环内部每一行
再缩进四个空格，然后重新运行程序。现在程序会永远请求下一次猜测，这又带来
一个问题：用户似乎无法退出！

用户始终可以按 <kbd>ctrl</kbd>-<kbd>C</kbd> 中断程序。但还有另一种逃离这个贪得
无厌怪物的方法：在[“比较猜测与秘密数字”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->一节对 `parse` 的讨论中提到，输入非数字会让程序崩溃。
可以利用这一点退出：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' (6694925) panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

输入 `quit` 会退出游戏，但其他任何非数字输入也会如此。至少可以说，这并不
理想；我们还希望在用户猜对时停止游戏。

<a id="quitting-after-a-correct-guess"></a>

### 猜对后退出

添加 `break` 语句，让游戏在用户获胜时退出：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

在 `You win!` 后添加 `break`，会在用户猜对秘密数字时退出循环。循环是 `main`
的最后一部分，因此退出循环也意味着退出程序。

### 处理无效输入

进一步完善游戏行为：用户输入非数字时，不让程序崩溃，而是忽略该输入，让用户
继续猜测。可以修改把 `guess` 从 `String` 转换为 `u32` 的代码，如示例 2-5。

<Listing number="2-5" file-name="src/main.rs" caption="忽略非数字猜测并再次请求输入，而不是让程序崩溃">

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

这里从调用 `expect` 改为使用 `match`，不再因错误崩溃，而是处理错误。`parse`
返回 `Result`，它是拥有 `Ok` 和 `Err` 变体的枚举。这里使用 `match` 的方式和
处理 `cmp` 方法返回的 `Ordering` 相同。

如果 `parse` 成功把字符串转成数字，就返回包含结果数字的 `Ok`。该值匹配第一
个分支的 `Ok(num)` 模式，`match` 返回 `parse` 生成并放进 `Ok` 的 `num`，
这个数字最终正好成为新 `guess` 变量的值。

如果转换失败，`parse` 返回包含更多错误信息的 `Err`。它不匹配第一个分支的
`Ok(num)`，却匹配第二个分支的 `Err(_)`。下划线 `_` 是<em>通配值(catch-all
value)</em>，表示无论 `Err` 内含什么信息都匹配。程序于是执行第二个分支的
`continue`，进入 `loop` 的下一次迭代并再次请求猜测。实际上，程序忽略了
`parse` 可能遇到的所有错误。

现在程序中的一切都应按预期工作。试试看：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

太棒了！只需最后一个小改动，游戏就完成了。程序仍在打印秘密数字，这在测试时
很好用，却会毁掉游戏。删除输出秘密数字的 `println!`。示例 2-6 是最终代码。

<Listing number="2-6" file-name="src/main.rs" caption="完整的猜数字游戏代码">

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

至此，你已经成功完成了猜数字游戏。恭喜！

## 小结

这个动手项目介绍了许多新的 Rust 概念：`let`、`match`、函数、外部 crate 的
使用等。接下来的几章会更详细地讲解这些概念。第 3 章介绍大多数编程语言都
具备的概念，如变量、数据类型和函数，并展示如何在 Rust 中使用它们。第 4 章
探讨所有权，这是 Rust 区别于其他语言的一项特性。第 5 章讨论结构体和方法
语法，第 6 章解释枚举的工作方式。

[prelude]: https://doc.rust-lang.org/std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: https://doc.rust-lang.org/std/string/struct.String.html
[iostdin]: https://doc.rust-lang.org/std/io/struct.Stdin.html
[read_line]: https://doc.rust-lang.org/std/io/struct.Stdin.html#method.read_line
[result]: https://doc.rust-lang.org/std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: https://doc.rust-lang.org/std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: https://doc.rust-lang.org/std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
