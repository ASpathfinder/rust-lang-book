<a id="variables-and-mutability"></a>

## 变量与可变性

正如[“使用变量存储值”][storing-values-with-variables]<!-- ignore -->一节所述，
变量默认不可变。这是 Rust 提供的诸多引导之一，促使你以能够利用 Rust 安全性
和便捷并发能力的方式编写代码。不过，你仍然可以让变量可变。下面探讨 Rust
为何鼓励优先使用不可变性，以及有时为何需要选择可变性。

变量不可变时，一旦把值绑定到某个名称，就不能再改变该值。为演示这一点，请在
<em>projects</em> 目录中运行 `cargo new variables`，生成名为 <em>variables</em> 的项目。

进入新目录，打开 <em>src/main.rs</em>，把代码替换为下面暂时无法编译的内容：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

保存后运行 `cargo run`，应该会收到有关不可变性的错误：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

这个例子展示了编译器如何帮助你发现程序错误。编译错误可能令人沮丧，但它们
实际上只表示程序暂时没有以安全方式完成你的意图，<em>并不</em>表示你不是一名优秀
程序员！经验丰富的 Rustacean 也会遇到编译错误。

你收到 `` cannot assign twice to immutable variable `x` ``，是因为尝试给
不可变变量 `x` 第二次赋值。

尝试修改声明为不可变的值时，在编译期收到错误非常重要，因为这种情况可能造成
缺陷。如果一部分代码假定某个值永远不变，另一部分却修改了它，前一部分就可能
无法按设计工作。事后追查这种缺陷可能很困难，尤其是后一部分只在<em>某些时候</em>
改变值时。Rust 编译器保证：当你声明值不会改变时，它就真的不会改变，因此
无需自己跟踪。代码也更容易推理。

不过，可变性很有用，也能让代码编写起来更方便。变量虽然默认不可变，但可以像
像[第 2 章][storing-values-with-variables]<!-- ignore -->那样，在变量名前添加 `mut` 使其可变。`mut` 还会向未来的代码读者传达
意图，表明代码的其他部分将修改这个变量的值。

把 <em>src/main.rs</em> 改成：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

运行后得到：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

使用 `mut` 后，可以把 `x` 所绑定的值从 `5` 改成 `6`。最终是否使用可变性由
你决定，应选择在具体情境中最清晰的方式。

<!-- Old headings. Do not remove or links may break. -->
<a id="constants"></a>

<a id="declaring-constants"></a>

### 声明常量

和不可变变量一样，<em>常量(constant)</em>也是绑定到名称且不允许改变的值，不过二者
存在一些差异。

首先，常量不能与 `mut` 一起使用。常量不只是默认不可变，而是始终不可变。
声明常量使用 `const` 而非 `let`，并且<em>必须</em>标注值的类型。下一节[“数据类型”][data-types]<!-- ignore -->
会介绍类型和类型标注；现在只需记住，常量总要标注类型。

常量可以在任何作用域中声明，包括全局作用域，因此适合保存代码许多部分都需要
知道的值。

最后，常量只能设置为常量表达式，不能设置为只能在运行时计算出的结果。

常量声明示例如下：

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量名是 `THREE_HOURS_IN_SECONDS`，值为 60（一分钟的秒数）乘以 60（一小时
的分钟数）再乘以 3（程序要计算的小时数）。Rust 惯例要求常量名全部大写，
单词间使用下划线。编译器能在编译期计算有限的一组运算，因此可以采用更易理解
和验证的写法，而不是直接把常量设为 10,800。声明常量时可用哪些运算，参见
[《Rust 参考手册》的常量求值部分][const-eval]。

在声明常量的作用域内，常量在程序整个运行期间都有效。这个特性适合应用领域中
程序多个部分都需要知道的值，例如游戏中玩家最多能获得的分数或光速。

把程序各处使用的硬编码值命名为常量，有助于向未来的维护者传达该值的含义；
以后更新硬编码值时，也只需修改一个位置。

<a id="shadowing"></a>

### 遮蔽

在[第 2 章][comparing-the-guess-to-the-secret-number]<!-- ignore -->的猜数字教程中，你已经声明过与旧变量同名的新变量。Rustacean 会说
第一个变量被第二个变量<em>遮蔽(shadowed)</em>：此后使用该变量名时，编译器看到的
是第二个变量。第二个变量实际上盖住了第一个变量；在它自己被遮蔽或作用域结束
前，所有变量名的使用都指向它。重复 `let` 并使用同一名称即可遮蔽变量：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

程序先把 `x` 绑定到 `5`。然后再次使用 `let x =` 创建新变量 `x`，把原值加
`1`，得到 `6`。在花括号创建的内部作用域中，第三个 `let` 再次遮蔽 `x`，
把前一个值乘以 `2`，得到 `12`。该作用域结束后，内部遮蔽也结束，`x` 恢复为
`6`。程序输出：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

遮蔽不同于把变量标记为 `mut`：如果忘记使用 `let` 而意外尝试重新赋值，会
收到编译期错误。使用 `let` 可以先对值作若干转换，再让转换后的变量保持不可变。

另一个区别是，再次使用 `let` 实际创建了新变量，因此可以改变值的类型，同时
复用同一名称。假设程序要求用户输入若干空格来指定文本间距，随后又想把输入
保存为数字：

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

第一个 `spaces` 是字符串，第二个是数字。遮蔽免去了 `spaces_str` 和
`spaces_num` 等不同名称，可以继续使用简单的 `spaces`。如果改用 `mut`，则会
收到编译期错误：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

错误指出不能改变变量的类型：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

了解变量的工作方式后，下面来看它们可以拥有的更多数据类型。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: https://doc.rust-lang.org/reference/const_eval.html
