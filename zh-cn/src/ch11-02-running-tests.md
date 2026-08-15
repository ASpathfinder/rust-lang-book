<a id="controlling-how-tests-are-run"></a>

## 控制测试的运行方式

正如 `cargo run` 会编译代码再运行生成的二进制文件，`cargo test` 会以测试模式编译代码，并运行生成的测试二进制文件。`cargo test` 生成的二进制文件默认会并行运行所有测试，并捕获测试运行期间生成的输出，不让它们显示出来，使与测试结果有关的输出更易阅读。不过，可以指定命令行选项来改变这种默认行为。

有些命令行选项传给 `cargo test`，有些传给生成的测试二进制文件。为了分隔这两类实参，请先列出传给 `cargo test` 的实参，接着写分隔符 `--`，然后列出传给测试二进制文件的实参。运行 `cargo test --help` 会显示可与 `cargo test` 一起使用的选项；运行 `cargo test -- --help` 则显示分隔符之后可以使用的选项。[《`rustc` 手册》的“测试”一节][tests]也记录了这些选项。

[tests]: https://doc.rust-lang.org/rustc/tests/index.html

### 并行或连续运行测试

运行多个测试时，它们默认使用线程并行运行，因此可以更快结束，更早获得反馈。由于测试同时运行，必须确保测试不相互依赖，也不依赖任何共享状态，包括当前工作目录或环境变量等共享环境。

例如，假设每个测试都运行某段代码，在磁盘上创建名为 <em>test-output.txt</em> 的文件，并向文件写入一些数据。接着，每个测试读取该文件中的数据，并断言文件包含某个特定值，而每个测试所需的值都不同。由于测试同时运行，一个测试可能会在另一个测试写入和读取文件之间覆盖该文件。第二个测试随后会失败，并非因为代码不正确，而是测试并行运行时相互干扰。一种解决方案是确保每个测试写入不同文件；另一种是每次只运行一个测试。

如果不想并行运行测试，或者希望更精细地控制所使用的线程数量，可以把 `--test-threads` 标志和希望使用的线程数传给测试二进制文件。请看下面的例子：

```console
$ cargo test -- --test-threads=1
```

我们把测试线程数设为 `1`，告诉程序不使用任何并行执行。使用一个线程运行测试比并行运行花费更长时间，但如果测试共享状态，它们不会相互干扰。

### 显示函数输出

默认情况下，如果测试通过，Rust 测试库会捕获打印到标准输出的所有内容。例如，如果在测试中调用 `println!` 且测试通过，就不会在终端看到 `println!` 的输出，只会看到表明测试通过的那一行。如果测试失败，则会看到打印到标准输出的内容以及其余失败信息。

例如，示例 11-10 中有一个无实际意义的函数，它会打印形参值并返回 10；其中还有一个通过的测试和一个失败的测试。

<Listing number="11-10" file-name="src/lib.rs" caption="调用 `println!` 的函数的测试">

```rust,panics,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

使用 `cargo test` 运行这些测试时，会看到以下输出：

```console
{{#include ../../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

请注意，输出中没有出现通过的测试运行时打印的 `I got the value 4`，因为这段输出被捕获了。失败测试的输出 `I got the value 8` 则出现在测试摘要输出的相应部分，其中还显示了测试失败的原因。

如果还想查看通过测试所打印的值，可以使用 `--show-output` 告诉 Rust 同时显示成功测试的输出：

```console
$ cargo test -- --show-output
```

使用 `--show-output` 标志再次运行示例 11-10 中的测试，会看到以下输出：

```console
{{#include ../../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

<a id="running-a-subset-of-tests-by-name"></a>

### 按名称运行测试子集

运行完整测试套件有时会花费很长时间。如果正在处理某个特定区域的代码，可能只想运行与该代码有关的测试。可以把要运行的测试名称作为实参传给 `cargo test`，选择运行哪些测试。

为了展示如何运行测试子集，我们会先为 `add_two` 函数创建三个测试，如示例 11-11 所示，然后选择要运行的测试。

<Listing number="11-11" file-name="src/lib.rs" caption="具有三个不同名称的三个测试">

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

如果像之前一样不传任何实参地运行测试，所有测试都会并行运行：

```console
{{#include ../../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### 运行单个测试

可以把任意测试函数的名称传给 `cargo test`，只运行该测试：

```console
{{#include ../../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

只有名为 `one_hundred` 的测试运行了，其他两个测试不匹配该名称。测试输出末尾显示 `2 filtered out`，告诉我们还有更多测试没有运行。

不能用这种方式指定多个测试名称；`cargo test` 只会使用给出的第一个值。不过，仍然有办法运行多个测试。

#### 通过筛选运行多个测试

可以指定测试名称的一部分，名称与该值匹配的所有测试都会运行。例如，由于两个测试的名称包含 `add`，可以运行 `cargo test add` 来执行这两个测试：

```console
{{#include ../../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

该命令运行了名称中含 `add` 的所有测试，并筛掉名为 `one_hundred` 的测试。另请注意，测试所在的模块会成为测试名称的一部分，因此可以通过筛选模块名称，运行某个模块中的所有测试。

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-some-tests-unless-specifically-requested"></a>

<a id="ignoring-tests-unless-specifically-requested"></a>

### 除非特别指定，否则忽略测试

有时，少数特定测试的执行可能非常耗时，因此在大多数 `cargo test` 运行中都希望排除它们。与其把所有希望运行的测试列为实参，不如使用 `ignore` 属性标注耗时的测试，将其排除，如下所示：

<span class="filename">文件名：src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

在 `#[test]` 之后，我们为要排除的测试添加 `#[ignore]` 行。现在运行测试时，`it_works` 会运行，而 `expensive_test` 不会：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

`expensive_test` 函数被列为 `ignored`。如果只想运行被忽略的测试，可以使用 `cargo test -- --ignored`：

```console
{{#include ../../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

通过控制运行哪些测试，可以确保 `cargo test` 快速返回结果。当适合检查 `ignored` 测试的结果、而且有时间等待时，可以改为运行 `cargo test -- --ignored`。如果无论是否忽略都想运行所有测试，可以运行 `cargo test -- --include-ignored`。
