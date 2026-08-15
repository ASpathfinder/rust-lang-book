<a id="how-to-write-tests"></a>

## 如何编写测试

<em>测试(test)</em>是验证非测试代码是否按预期方式运行的 Rust 函数。测试函数的函数体通常执行以下三项操作：

- 设置所需的数据或状态。
- 运行要测试的代码。
- 断言结果符合预期。

让我们看看 Rust 专门为编写执行这些操作的测试提供了哪些功能，包括 `test` 属性、几个宏和 `should_panic` 属性。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### 测试函数的结构

最简单的 Rust 测试，就是带有 `test` 属性标注的函数。<em>属性(attribute)</em>是有关 Rust 代码片段的元数据；第 5 章中与结构体一起使用的 `derive` 就是一个例子。要把函数变成测试函数，请在 `fn` 之前一行添加 `#[test]`。使用 `cargo test` 命令运行测试时，Rust 会构建一个测试运行器二进制文件，用来运行带有该标注的函数，并报告每个测试函数通过还是失败。

每当使用 Cargo 创建新的库项目时，系统都会自动生成一个包含测试函数的测试模块。该模块为编写测试提供了模板，让你不必在每次开始新项目时都查找确切的结构和语法。你可以添加任意多个测试函数和测试模块！

在真正测试代码之前，我们会先试验模板测试，探索测试工作方式的几个方面。然后编写一些实际测试，调用自己编写的代码，并断言其行为正确。

让我们创建一个名为 `adder`、用于把两个数字相加的新库项目：

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

`adder` 库中 <em>src/lib.rs</em> 文件的内容应该与示例 11-1 相同。

<Listing number="11-1" file-name="src/lib.rs" caption="由 `cargo new` 自动生成的代码">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

文件开头是一个示例 `add` 函数，让我们有东西可供测试。

现在先只关注 `it_works` 函数。请注意 `#[test]` 标注：该属性表示这是一个测试函数，因此测试运行器知道要把它当作测试。`tests` 模块中也可能包含帮助设置常见场景或执行常见操作的非测试函数，所以始终需要指出哪些函数是测试。

示例函数体使用 `assert_eq!` 宏断言 `result` 等于 4；`result` 中存放的是以 2 和 2 调用 `add` 所得到的结果。这个断言展示了典型测试的格式。让我们运行它，看看测试是否通过。

`cargo test` 命令会运行项目中的所有测试，如示例 11-2 所示。

<Listing number="11-2" caption="运行自动生成测试的输出">

```console
{{#include ../../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo 编译并运行了测试。我们会看到 `running 1 test` 这一行。下一行显示生成的测试函数名 `tests::it_works`，并表明测试运行结果为 `ok`。总摘要 `test result: ok.` 表示所有测试都已通过，而 `1 passed; 0 failed` 部分汇总了通过和失败的测试数量。

可以把测试标记为忽略，使其在某次运行中不执行；本章后面的[“除非特别指定，否则忽略测试”][ignoring]一节会介绍这种做法。这里没有这样做，所以摘要显示 `0 ignored`。还可以向 `cargo test` 命令传入实参，只运行名称与某个字符串匹配的测试；这称为<em>筛选(filtering)</em>，会在[“按名称运行测试子集”][subset]一节介绍。这里没有筛选要运行的测试，所以摘要末尾显示 `0 filtered out`。

`0 measured` 统计数据用于衡量性能的基准测试。撰写本书时，基准测试只在 nightly Rust 中可用。有关更多信息，请参阅[基准测试文档][bench]。

测试输出中以 `Doc-tests adder` 开头的下一部分，是所有文档测试的结果。我们还没有文档测试，但 Rust 可以编译 API 文档中出现的所有代码示例。该功能有助于让文档与代码保持同步！第 14 章[“作为测试的文档注释”][doc-comments]一节会讨论如何编写文档测试。现在先忽略 `Doc-tests` 输出。

让我们开始按照自己的需求定制测试。首先，把 `it_works` 函数改成其他名称，例如 `exploration`：

<span class="filename">文件名：src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

然后再次运行 `cargo test`。现在输出显示 `exploration`，而不是 `it_works`：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

接下来再添加一个测试，但这次让测试失败！测试函数中的某处发生 panic 时，测试便会失败。每个测试都在新线程中运行；主线程发现测试线程已经终止时，就会把测试标记为失败。第 9 章中讨论过，引发 panic 最简单的方法是调用 `panic!` 宏。请输入名为 `another` 的新测试函数，让 <em>src/lib.rs</em> 文件与示例 11-3 相同。

<Listing number="11-3" file-name="src/lib.rs" caption="添加第二个测试；由于调用 `panic!` 宏，该测试会失败">

```rust,panics,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

再次使用 `cargo test` 运行测试。输出应与示例 11-4 类似，表明 `exploration` 测试通过，而 `another` 失败。

<Listing number="11-4" caption="一个测试通过、一个测试失败时的测试结果">

```console
{{#include ../../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

`test tests::another` 行显示的不再是 `ok`，而是 `FAILED`。单项结果和摘要之间出现了两个新部分：第一部分显示每项测试失败的详细原因。在这里，详细信息表明 `tests::another` 因为在 <em>src/lib.rs</em> 文件第 17 行以信息 `Make this test fail` 发生 panic 而失败。下一部分只列出所有失败测试的名称；当测试很多、详细失败输出也很多时，这非常有用。可以使用失败测试的名称只运行该测试，以便更轻松地调试；[“控制测试的运行方式”][controlling-how-tests-are-run]一节会更详细地讨论运行测试的方式。

摘要行显示在最后：测试的整体结果为 `FAILED`。一个测试通过，一个测试失败。

现在你已经看过不同场景下的测试结果，接下来看看测试中除了 `panic!` 之外还有哪些有用的宏。

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### 使用 `assert!` 检查结果

标准库提供的 `assert!` 宏适用于确保测试中的某个条件求值为 `true`。我们向 `assert!` 宏传入一个求值为布尔值的实参。如果值为 `true`，什么也不会发生，测试通过；如果值为 `false`，`assert!` 宏会调用 `panic!`，使测试失败。使用 `assert!` 宏有助于检查代码是否按预期方式运行。

第 5 章的示例 5-15 使用了 `Rectangle` 结构体和 `can_hold` 方法，示例 11-5 在这里再次展示它们。让我们把这段代码放入 <em>src/lib.rs</em> 文件，再使用 `assert!` 宏为其编写一些测试。

<Listing number="11-5" file-name="src/lib.rs" caption="第 5 章中的 `Rectangle` 结构体及其 `can_hold` 方法">

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

`can_hold` 方法返回布尔值，因此非常适合使用 `assert!` 宏。在示例 11-6 中，我们创建宽为 8、高为 7 的 `Rectangle` 实例，并断言它可以容纳另一个宽为 5、高为 1 的 `Rectangle` 实例，以此测试 `can_hold` 方法。

<Listing number="11-6" file-name="src/lib.rs" caption="`can_hold` 测试，检查较大矩形确实能够容纳较小矩形">

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

请注意 `tests` 模块中的 `use super::*;` 行。`tests` 模块是遵循普通可见性规则的常规模块，这些规则在第 7 章[“引用模块树中条目的路径”][paths-for-referring-to-an-item-in-the-module-tree]一节介绍过。由于 `tests` 是内部模块，需要把外部模块中被测试的代码引入内部模块的作用域。这里使用 glob，让外部模块中定义的所有内容都可供 `tests` 模块使用。

我们把测试命名为 `larger_can_hold_smaller`，并创建所需的两个 `Rectangle` 实例。然后调用 `assert!` 宏，向它传入调用 `larger.can_hold(&smaller)` 所得到的结果。这个表达式应该返回 `true`，所以测试应该通过。让我们看看！

```console
{{#include ../../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

它确实通过了！再添加一个测试，这次断言较小矩形无法容纳较大矩形：

<span class="filename">文件名：src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

因为在这种情况下，`can_hold` 函数的正确结果是 `false`，所以在把结果传给 `assert!` 宏之前需要将其取反。因此，如果 `can_hold` 返回 `false`，测试就会通过：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

两个测试都通过了！现在看看在代码中引入 bug 后，测试结果会发生什么变化。我们修改 `can_hold` 方法的实现，在比较宽度时把大于号（`>`）替换为小于号（`<`）：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

现在运行测试会产生以下结果：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

测试捕获了 bug！由于 `larger.width` 为 `8`，`smaller.width` 为 `5`，`can_hold` 中的宽度比较现在返回 `false`：8 并不小于 5。

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### 使用 `assert_eq!` 和 `assert_ne!` 测试相等性

验证功能的一种常见方式，是测试被测代码的结果与预期返回值是否相等。可以使用 `assert!` 宏，并向它传入一个使用 `==` 运算符的表达式来做到这一点。不过，这种测试非常常见，因此标准库提供了 `assert_eq!` 和 `assert_ne!` 两个宏，让相等和不等测试更加方便。如果断言失败，这些宏还会打印两个值，更容易看出测试<em>为什么</em>失败；相比之下，`assert!` 宏只会表明 `==` 表达式得到 `false`，并不打印导致 `false` 的值。

示例 11-7 中，我们编写了一个把形参加 `2` 的 `add_two` 函数，再使用 `assert_eq!` 宏测试它。

<Listing number="11-7" file-name="src/lib.rs" caption="使用 `assert_eq!` 宏测试 `add_two` 函数">

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

检查一下它是否通过！

```console
{{#include ../../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

我们创建名为 `result` 的变量，存放调用 `add_two(2)` 的结果。然后把 `result` 和 `4` 作为实参传给 `assert_eq!` 宏。该测试的输出行为 `test tests::it_adds_two ... ok`，其中 `ok` 表示测试通过！

让我们在代码中引入 bug，看看 `assert_eq!` 失败时是什么样子。修改 `add_two` 函数的实现，让它改为加 `3`：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

再次运行测试：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

测试捕获了 bug！`tests::it_adds_two` 测试失败，信息告诉我们失败的断言是 `left == right`，并显示了 `left` 和 `right` 的值。这条信息有助于开始调试：存放 `add_two(2)` 调用结果的 `left` 实参为 `5`，而 `right` 实参为 `4`。可以想象，当测试很多时，这会特别有帮助。

请注意，在一些语言和测试框架中，相等断言函数的形参称为 `expected` 和 `actual`，指定实参的顺序很重要。然而在 Rust 中，它们称为 `left` 和 `right`，预期值和代码产生值的指定顺序无关紧要。这个测试中的断言也可以写成 `assert_eq!(4, result)`，得到的失败信息同样会显示 `` assertion `left == right` failed ``。

如果提供的两个值不相等，`assert_ne!` 宏会通过；如果相等，则会失败。当我们不确定某个值<em>会</em>是什么，但知道它肯定<em>不应该</em>是什么时，这个宏最有用。例如，假设正在测试一个保证以某种方式修改输入的函数，但输入的修改方式取决于运行测试的是星期几，那么最合适的断言可能是函数输出不等于输入。

在底层，`assert_eq!` 和 `assert_ne!` 宏分别使用 `==` 和 `!=` 运算符。断言失败时，这些宏会使用调试格式打印实参，这意味着被比较的值必须实现 `PartialEq` 和 `Debug` 特征。所有基本类型和大多数标准库类型都实现了这些特征。对于自己定义的结构体和枚举，需要实现 `PartialEq` 才能断言这些类型相等，还需要实现 `Debug` 才能在断言失败时打印值。正如第 5 章示例 5-12 所述，这两个特征都是可派生特征，因此通常只需向结构体或枚举定义添加 `#[derive(PartialEq, Debug)]` 标注。有关这些及其他可派生特征的更多信息，请参阅附录 C[“可派生特征”][derivable-traits]。

### 添加自定义失败信息

还可以把可选实参传给 `assert!`、`assert_eq!` 和 `assert_ne!` 宏，添加与失败信息一起打印的自定义信息。必需实参之后指定的所有实参都会传递给 `format!` 宏（第 8 章[“使用 `+` 或 `format!` 拼接”][concatenating]中讨论过），因此可以传入含有 `{}` 占位符的格式字符串，以及填入占位符的值。自定义信息有助于记录断言的含义；测试失败时，你会更清楚代码出了什么问题。

例如，假设有一个按姓名问候用户的函数，希望测试传给函数的姓名是否出现在输出中：

<span class="filename">文件名：src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

这个程序的需求尚未确定，我们很确信问候语开头的 `Hello` 文本会发生变化。我们不想在需求变化时更新测试，因此不会检查 `greeting` 函数的返回值是否与某个值完全相等，而只断言输出包含输入形参的文本。

现在修改 `greeting`，让它排除 `name`，从而在代码中引入 bug，看看默认的测试失败是什么样子：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

运行该测试会产生以下结果：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

这个结果只表示断言失败，并指出断言所在行。更有用的失败信息应该打印 `greeting` 函数的值。让我们添加一条自定义失败信息，由格式字符串和占位符组成，占位符中填入从 `greeting` 函数实际得到的值：

```rust,ignore
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

现在运行测试时，会得到信息量更丰富的错误信息：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

测试输出显示了实际得到的值，帮助我们调试实际发生的情况与预期之间的差异。

### 使用 `should_panic` 检查 panic

除了检查返回值，检查代码是否按预期处理错误情况也很重要。例如，考虑第 9 章示例 9-13 中创建的 `Guess` 类型。使用 `Guess` 的其他代码依赖于一项保证：`Guess` 实例只包含 1 到 100 之间的值。我们可以编写测试，确保尝试使用范围外的值创建 `Guess` 实例时会 panic。

为此，我们向测试函数添加 `should_panic` 属性。如果函数内的代码发生 panic，测试通过；如果没有 panic，测试失败。

示例 11-8 展示了一个测试，用于检查 `Guess::new` 的错误情况是否在预期时发生。

<Listing number="11-8" file-name="src/lib.rs" caption="测试某个条件是否会引发 `panic!`">

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

我们把 `#[should_panic]` 属性放在 `#[test]` 属性之后、它所应用的测试函数之前。来看看该测试通过时的结果：

```console
{{#include ../../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

看起来不错！现在删除 `new` 函数在值大于 100 时 panic 的条件，在代码中引入 bug：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

运行示例 11-8 中的测试时，它会失败：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

这里没有得到非常有帮助的信息，但查看测试函数时，可以看到它带有 `#[should_panic]` 标注。得到的失败结果表示测试函数中的代码没有引发 panic。

使用 `should_panic` 的测试可能不够精确。即使测试由于预期之外的原因发生 panic，`should_panic` 测试也会通过。为了让 `should_panic` 测试更精确，可以为 `should_panic` 属性添加可选的 `expected` 形参。测试框架会确保失败信息包含所提供的文本。例如，考虑示例 11-9 中修改后的 `Guess` 代码，其中 `new` 函数会根据值太小或太大以不同信息 panic。

<Listing number="11-9" file-name="src/lib.rs" caption="测试 `panic!`，并要求 panic 信息包含指定子字符串">

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

这个测试会通过，因为 `should_panic` 属性的 `expected` 形参中的值，是 `Guess::new` 函数 panic 信息的一个子字符串。也可以指定预期的完整 panic 信息，在这里是 `Guess value must be less than or equal to 100, got 200`。具体指定多少内容，取决于 panic 信息中有多少是唯一或动态的，以及希望测试多么精确。在这个例子中，panic 信息的子字符串足以确保测试函数中的代码执行了 `else if value > 100` 分支。

为了看看带 `expected` 信息的 `should_panic` 测试失败时会发生什么，再次在代码中引入 bug，交换 `if value < 1` 与 `else if value > 100` 块的函数体：

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

这次运行 `should_panic` 测试时，它会失败：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

失败信息表明，该测试确实如预期发生了 panic，但 panic 信息不包含预期字符串 `less than or equal to 100`。实际得到的 panic 信息是 `Guess value must be greater than or equal to 1, got 200`。现在就可以开始查找 bug 所在的位置了！

### 在测试中使用 `Result<T, E>`

到目前为止，所有测试都在失败时 panic。也可以编写使用 `Result<T, E>` 的测试！下面是示例 11-1 中的测试，改写为使用 `Result<T, E>` 并在失败时返回 `Err`，而不是 panic：

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

现在，`it_works` 函数的返回类型是 `Result<(), String>`。在函数体中，我们不调用 `assert_eq!` 宏，而是在测试通过时返回 `Ok(())`，测试失败时返回包含 `String` 的 `Err`。

让测试返回 `Result<T, E>`，就可以在测试函数体中使用问号运算符。这是一种便捷写法，适用于任何内部操作返回 `Err` 变体时都应该失败的测试。

不能对使用 `Result<T, E>` 的测试使用 `#[should_panic]` 标注。要断言操作返回 `Err` 变体，请<em>不要</em>对 `Result<T, E>` 值使用问号运算符，而应使用 `assert!(value.is_err())`。

现在你已经了解编写测试的几种方式，接下来看看运行测试时会发生什么，并探索 `cargo test` 可以使用的不同选项。

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: https://doc.rust-lang.org/unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
