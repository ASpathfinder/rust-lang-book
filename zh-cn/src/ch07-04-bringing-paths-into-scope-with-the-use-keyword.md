## 使用 `use` 关键字把路径引入作用域

每次调用函数都写出完整路径可能既不方便又重复。在示例 7-7 中，无论选择绝对路径
还是相对路径，每次调用 `add_to_waitlist` 都必须同时写出 `front_of_house` 和
`hosting`。幸好可以简化：先用 `use` 关键字为路径创建一次快捷方式，之后在该
作用域的其他位置使用较短名称。

示例 7-11 把 `crate::front_of_house::hosting` 模块引入 `eat_at_restaurant` 的
作用域，这样调用函数时只需写 `hosting::add_to_waitlist`。

<Listing number="7-11" file-name="src/lib.rs" caption="使用 `use` 把模块引入作用域">

```rust,noplayground,test_harness
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

在作用域中添加 `use` 和路径，类似于在文件系统中创建符号链接。在 crate 根添加
`use crate::front_of_house::hosting` 后，`hosting` 就成为该作用域中的有效名称，
仿佛模块直接定义在 crate 根中。与其他路径一样，通过 `use` 引入作用域的路径也会
接受私有性检查。

`use` 只为它所在的特定作用域创建快捷方式。示例 7-12 把 `eat_at_restaurant`
移入名为 `customer` 的新子模块；它与 `use` 语句处于不同作用域，所以函数体无法
编译。

<Listing number="7-12" file-name="src/lib.rs" caption="`use` 语句只在其所在作用域中生效">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

编译器错误表明快捷方式在 `customer` 模块中不再生效：

```console
{{#include ../../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

同时还有警告指出 `use` 在其作用域中不再使用！要修复问题，可以把 `use` 也移入
`customer`，或者在这个子模块中用 `super::hosting` 引用父模块的快捷方式。

<a id="creating-idiomatic-use-paths"></a>

### 创建惯用的 `use` 路径

在示例 7-11 中，你可能疑惑为何只写 `use crate::front_of_house::hosting`，然后
在函数中调用 `hosting::add_to_waitlist`；为什么不像示例 7-13 那样，把 `use`
路径一直写到 `add_to_waitlist`，达到相同结果？

<Listing number="7-13" file-name="src/lib.rs" caption="使用 `use` 把 `add_to_waitlist` 函数引入作用域，这种写法不符合惯例">

```rust,noplayground,test_harness
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

两个示例都能完成任务，但把函数引入作用域时，示例 7-11 才是惯用写法。用 `use`
引入函数的父模块，意味着调用时仍需指定父模块。这样既减少完整路径的重复，又明确
函数并非在本地定义。示例 7-13 则看不出 `add_to_waitlist` 定义在哪里。

另一方面，用 `use` 引入结构体、枚举和其他条目时，惯例是指定完整路径。示例
7-14 展示了把标准库的 `HashMap` 引入二进制 crate 作用域的惯用方式。

<Listing number="7-14" file-name="src/main.rs" caption="以惯用方式把 `HashMap` 引入作用域">

```rust
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

这种惯例没有强制理由，只是逐渐形成的约定，大家已经习惯这样读写 Rust 代码。

例外情况是用 `use` 把两个同名条目引入作用域，因为 Rust 不允许这样做。示例
7-15 展示如何引入名称相同、父模块不同的两个 `Result` 类型，并引用它们。

<Listing number="7-15" file-name="src/lib.rs" caption="把两个同名类型引入同一作用域时需要使用其父模块">

```rust,noplayground
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

父模块能区分两个 `Result`。如果改为指定 `use std::fmt::Result` 和
`use std::io::Result`，同一作用域会有两个 `Result`，Rust 使用时无法判断指哪一个。

### 使用 `as` 关键字提供新名称

还可以用另一种方式解决同名类型的冲突：在路径后写 `as` 和新的本地名称，也就是
<em>别名(alias)</em>。示例 7-16 用 `as` 重命名两个 `Result` 中的一个，以另一种方式
改写示例 7-15。

<Listing number="7-16" file-name="src/lib.rs" caption="用 `as` 关键字重命名引入作用域的类型">

```rust,noplayground
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

第二条 `use` 为 `std::io::Result` 选择新名称 `IoResult`，不会与同时引入的
`std::fmt::Result` 冲突。示例 7-15 和 7-16 都符合惯例，可自行选择！

### 使用 `pub use` 重新导出名称

用 `use` 把名称引入作用域后，该名称对导入它的作用域是私有的。如果希望作用域
外的代码能像名称定义在该作用域中一样引用它，可以组合 `pub` 与 `use`。这种技巧
称为<em>重新导出(re-exporting)</em>：既把条目引入作用域，也让其他代码能够再把它
引入自己的作用域。

示例 7-17 把示例 7-11 根模块中的 `use` 改为 `pub use`。

<Listing number="7-17" file-name="src/lib.rs" caption="使用 `pub use`，让任何代码都能从新作用域使用某个名称">

```rust,noplayground,test_harness
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

修改前，外部代码必须通过路径
`restaurant::front_of_house::hosting::add_to_waitlist()` 调用函数，而且还要把
`front_of_house` 标记为 `pub`。现在 `pub use` 从根模块重新导出 `hosting`，外部
代码可以改用 `restaurant::hosting::add_to_waitlist()`。

当代码内部结构与调用者理解问题领域的方式不同，重新导出很有用。例如，餐厅经营
者会考虑“前台”和“后台”，顾客却很可能不会这样划分。借助 `pub use`，代码内部
采用一种结构，对外暴露另一种结构，让库对开发者和调用者都组织良好。第 14 章的
[“导出方便的公开 API”][ch14-pub-use]<!-- ignore -->一节会展示另一个 `pub use`
示例及其对 crate 文档的影响。

### 使用外部包

第 2 章的猜数字项目使用外部包 `rand` 生成随机数。为了使用它，我们在
<em>Cargo.toml</em> 中加入：

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

把 `rand` 作为依赖加入 <em>Cargo.toml</em>，会让 Cargo 从
[crates.io](https://crates.io/) 下载该包及其所有依赖，并使项目可以使用 `rand`。

随后，为了把 `rand` 定义引入包的作用域，我们添加从 crate 名称 `rand` 开始的
`use` 行，并列出要引入的条目。回顾第 2 章的[“生成随机数”][rand]<!-- ignore -->
一节：代码把 `rand::prelude` 中的条目引入作用域，并调用 `rand::rng`：

```rust,ignore
{{#rustdoc_include ../../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Rust 社区在 [crates.io](https://crates.io/) 提供了许多包。把其中任意包引入项目
都遵循相同步骤：在包的 <em>Cargo.toml</em> 中列出，再用 `use` 把其 crate 中的条目
引入作用域。

标准库 `std` 同样是包之外的 crate。它随 Rust 一起提供，所以无须修改
<em>Cargo.toml</em>，但仍需用 `use` 引入其中条目。例如 `HashMap` 使用：

```rust
use std::collections::HashMap;
```

这是以标准库 crate 名称 `std` 开始的绝对路径。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### 使用嵌套路径整理较长的 `use` 列表

使用同一 crate 或模块中定义的多个条目时，每项单独一行会占用许多垂直空间。
例如，猜数字游戏示例 2-4 中有两条从 `std` 引入条目的 `use`：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

可以改用嵌套路径，一行引入相同条目：先指定路径的共同部分和双冒号，再用花括号
包住各路径的不同部分，如示例 7-18 所示。

<Listing number="7-18" file-name="src/main.rs" caption="用嵌套路径把具有相同前缀的多个条目引入作用域">

```rust,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

在大型程序中，用嵌套路径从同一 crate 或模块引入许多条目，可以大幅减少所需的
独立 `use` 语句！

路径的任何层级都可以使用嵌套路径，这有助于合并共享子路径的 `use`。示例 7-19
有两条语句，分别引入 `std::io` 和 `std::io::Write`。

<Listing number="7-19" file-name="src/lib.rs" caption="一条路径是另一条路径子路径的两个 `use` 语句">

```rust,noplayground
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

共同部分 `std::io` 恰好也是第一条完整路径。把二者合为一条 `use` 时，可以像
示例 7-20 那样在嵌套路径中使用 `self`。

<Listing number="7-20" file-name="src/lib.rs" caption="把示例 7-19 的路径合并为一条 `use` 语句">

```rust,noplayground
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

这一行把 `std::io` 和 `std::io::Write` 都引入作用域。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-glob-operator"></a>

### 使用 glob 运算符导入条目

要把路径中定义的<em>所有</em>公开条目引入作用域，可以在路径后加 `*` <em>glob 运算符
(glob operator)</em>：

```rust
use std::collections::*;
```

这条语句把 `std::collections` 的所有公开条目引入当前作用域。使用 glob 时要
小心！它会让人难以判断作用域中有哪些名称，以及程序中的名称定义在哪里。此外，
依赖更改定义时，导入的内容也会改变。例如依赖新增一个与你在同一作用域中定义
同名的条目，升级后可能出现编译错误。

测试时经常使用 glob，把所有被测内容引入 `tests` 模块；第 11 章的[“如何编写
测试”][writing-tests]<!-- ignore -->一节会介绍。glob 有时也用于预导入模块模式；
更多信息参阅[标准库文档](https://doc.rust-lang.org/std/prelude/index.html#other-preludes)<!-- ignore -->。

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
