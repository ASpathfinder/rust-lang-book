<a id="control-flow"></a>

## 控制流

根据条件是否为 `true` 来运行某些代码，以及在条件为 `true` 时反复运行代码，
是大多数编程语言的基础构件。Rust 中控制执行流程最常见的结构是 `if` 表达式
和循环。

### `if` 表达式

`if` 表达式允许根据条件为代码分支：提供一个条件，并规定“条件满足时运行这个
代码块；不满足时不运行”。

在 <em>projects</em> 中创建项目 <em>branches</em>，把以下内容写入 <em>src/main.rs</em>：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

所有 `if` 都以关键字 `if` 开头，后跟条件。这里检查 `number` 是否小于 5。
条件为 `true` 时执行的代码块紧跟其后，放在花括号内。与第 2 章[“比较猜测与秘密数字”][comparing-the-guess-to-the-secret-number]<!-- ignore -->一节的 `match` 一样，
`if` 条件对应的代码块有时也称为<em>分支(arm)</em>。

还可以像这里一样添加 `else`，提供条件为 `false` 时执行的备选代码块。没有
`else` 且条件为 `false` 时，程序会跳过 `if` 块，继续执行后续代码。

运行会看到：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

把 `number` 改成会让条件为 `false` 的值：

```rust,ignore
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

再次运行：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

条件<em>必须</em>是 `bool`，否则会报错。例如：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

这次条件求值得到 `3`，Rust 会报错：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

错误指出 Rust 期望 `bool`，却得到整数。不同于 Ruby、JavaScript 等语言，Rust
不会自动把非布尔类型转换成布尔值，必须显式给 `if` 提供布尔条件。例如，只在
数字不等于 `0` 时运行代码块，可以写：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

运行会打印 `number was something other than zero`。

#### 使用 `else if` 处理多个条件

可以组合 `if` 和 `else`，用 `else if` 处理多个条件：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

程序有四条可能路径，运行输出：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

程序依次检查各 `if`，执行第一个求值为 `true` 的代码块。虽然 6 也能被 2 整除，
却不会打印对应文本，也不会执行 `else`，因为 Rust 找到第一个真条件后便不再
检查其余条件。

过多 `else if` 会让代码杂乱；如果超过一个，可以考虑重构。第 6 章会介绍适合
这类场景的强大分支结构 `match`。

#### 在 `let` 语句中使用 `if`

`if` 是表达式，所以能放在 `let` 右侧，把结果赋给变量，如示例 3-2。

<Listing number="3-2" file-name="src/main.rs" caption="把 `if` 表达式的结果赋给变量">

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

`number` 根据 `if` 的结果绑定值。运行得到：

```console
{{#include ../../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

代码块求值得到最后一个表达式，单独的数字也是表达式，所以整个 `if` 的值取决于
执行哪个块。这意味着每个分支可能产生的结果必须类型相同；示例中都是 `i32`。
类型不匹配则报错：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

编译错误会准确指出问题：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

`if` 块求值得到整数，`else` 块得到字符串，无法工作。变量只能拥有一种类型，
Rust 必须在编译期明确 `number` 的类型，才能验证它在各处的使用。若类型只能在
运行时决定，编译器会更复杂，对代码的保证也更少。

### 使用循环重复执行

经常需要多次执行同一代码块。Rust 提供多种<em>循环(loop)</em>：运行完循环体后，
立即回到开头。新建项目 <em>loops</em> 进行实验。

Rust 有三种循环：`loop`、`while` 和 `for`。

#### 使用 `loop` 重复代码

`loop` 让 Rust 一再执行代码块，永远持续或直到显式停止。把 <em>loops</em> 中的
<em>src/main.rs</em> 改成：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

程序会不停打印 `again!`，直到手动停止。多数终端可用
<kbd>ctrl</kbd>-<kbd>C</kbd> 中断持续循环：

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

`^C` 表示按下组合键的位置。中断信号到达时循环执行到哪里，决定了 `^C` 后是否
还能看到一个 `again!`。

Rust 也能用代码退出循环。在循环中放置 `break`，可告诉程序何时停止。第 2 章
猜数字游戏的[“猜对后退出”][quitting-after-a-correct-guess]<!-- ignore -->一节就这样在用户猜对时退出。`continue` 则跳过本次迭代的剩余代码，进入
下一次迭代。

#### 从循环返回值

`loop` 可用于重试可能失败的操作，例如检查线程是否完成；有时还需要把操作结果
传出循环。可在停止循环的 `break` 后添加要返回的值：

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

循环前声明 `counter` 并初始化为 `0`，再声明 `result` 保存循环返回值。每次
迭代把 `counter` 加 `1`，等于 `10` 时执行带有 `counter * 2` 的 `break`。
循环后的分号结束向 `result` 赋值的语句，最后打印 `20`。

也可在循环中 `return`。`break` 只退出当前循环，`return` 总是退出当前函数。

<!-- Old headings. Do not remove or links may break. -->
<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### 使用循环标签消除歧义

循环嵌套时，`break` 和 `continue` 默认作用于最内层循环。可以给循环添加以单
引号开头的<em>循环标签(loop label)</em>，再让关键字作用于带标签的循环：

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

外层循环标签是 `'counting_up`，从 0 数到 2；无标签的内层循环从 10 数到 9。
第一个无标签 `break` 只退出内层循环，`break 'counting_up;` 退出外层。输出：

```console
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->
<a id="conditional-loops-with-while"></a>

#### 使用 `while` 简化条件循环

程序经常需要在循环中求值条件：条件为 `true` 时运行，变为 `false` 时停止。
可以组合 `loop`、`if`、`else` 和 `break` 实现，但这一模式十分常见，所以
Rust 内置了 `while`。示例 3-3 用它循环三次倒计时，随后打印消息并退出。

<Listing number="3-3" file-name="src/main.rs" caption="条件为 `true` 时使用 `while` 循环运行代码">

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

它消除了组合其他结构所需的大量嵌套，也更清晰：条件为真时运行，否则退出。

<a id="looping-through-a-collection-with-for"></a>

#### 使用 `for` 遍历集合

可以用 `while` 遍历数组等集合。示例 3-4 打印数组 `a` 的每个元素。

<Listing number="3-4" file-name="src/main.rs" caption="使用 `while` 循环遍历集合中的每个元素">

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

代码从索引 `0` 开始递增，直到 `index < 5` 不再为真。输出：

```console
{{#include ../../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

五个值都出现了；虽然 `index` 最终到达 5，循环会在尝试取得第六个值前停止。

不过这种方式容易出错：索引或测试条件不正确会导致 panic。例如把数组改成四个
元素却忘记把条件改成 `index < 4`。它也较慢，因为编译器会加入运行时代码，
在每次迭代中检查索引是否位于数组范围内。

更简洁的选择是用 `for` 对集合中的每个条目执行代码，如示例 3-5。

<Listing number="3-5" file-name="src/main.rs" caption="使用 `for` 循环遍历集合中的每个元素">

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

输出与示例 3-4 相同。更重要的是，代码更安全，不会越过数组末尾，也不会过早
停止而遗漏条目；生成的机器码还可能更高效，因为不必每次比较索引和数组长度。

改变数组值的数量时，`for` 也无需同步修改其他代码。它既安全又简洁，是 Rust
最常用的循环结构。即使只想运行固定次数，多数 Rustacean 也会用 `for`。标准库
提供的 `Range` 会生成从某个数字开始，到另一个数字之前结束的连续数字。

使用 `for` 和尚未介绍的 `rev` 方法反转范围，可以这样倒计时：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

这样更漂亮，不是吗？

## 小结

你完成了！这是内容丰富的一章：变量、标量与复合数据类型、函数、注释、`if`
表达式和循环。可以尝试编写以下程序来练习：

- 在华氏温度和摄氏温度之间转换。
- 生成第 <em>n</em> 个斐波那契数。
- 利用重复内容，打印圣诞颂歌“The Twelve Days of Christmas”的歌词。

准备好后，我们将讨论一种其他编程语言中并不常见的 Rust 概念：所有权。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
