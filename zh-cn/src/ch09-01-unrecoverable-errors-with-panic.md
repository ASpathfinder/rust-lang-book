## 使用 `panic!` 处理不可恢复错误

有时，代码中会发生糟糕的事情，而你对此无能为力。在这些情况下，Rust 提供了 `panic!` 宏。实践中有两种方式会引发 <em>panic</em>：执行某个导致代码 panic 的操作（例如访问数组末尾之外的位置），或者显式调用 `panic!` 宏。两种情况都会让程序进入 panic。默认情况下，panic 会打印失败信息、展开并清理栈，然后退出。你也可以通过环境变量让 Rust 在 panic 发生时显示调用栈，以便更容易追踪 panic 的来源。

> ### 遇到 panic 时展开栈还是中止程序
>
> 默认情况下，panic 发生时，程序会开始<em>展开(unwinding)</em>，也就是说 Rust 会沿栈向上回溯，并清理遇到的每个函数中的数据。然而，回溯和清理需要做大量工作。因此，Rust 允许你选择另一种方式：立即<em>中止(aborting)</em>，即不做清理便结束程序。
>
> 随后，程序使用的内存需要由操作系统清理。如果项目需要让最终的二进制文件尽可能小，可以在 <em>Cargo.toml</em> 文件相应的 `[profile]` 部分添加 `panic = 'abort'`，把 panic 时的行为从展开改为中止。例如，若想在发布模式下遇到 panic 时中止程序，请添加：
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

让我们在一个简单程序中尝试调用 `panic!`：

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

运行程序时，会看到类似下面的内容：

```console
{{#include ../../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

对 `panic!` 的调用产生了最后两行中的错误信息。第一行显示了 panic 信息及其在源代码中发生的位置：<em>src/main.rs:2:5</em> 表示它位于 <em>src/main.rs</em> 文件的第 2 行第 5 个字符。

在这个例子中，所指示的行属于我们自己的代码；转到该行，可以看到对 `panic!` 宏的调用。在其他情况下，`panic!` 调用可能位于我们的代码所调用的代码中，此时错误信息报告的文件名和行号会是他人代码中调用 `panic!` 宏的位置，而不是我们代码中最终引发 `panic!` 调用的那一行。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

我们可以利用 `panic!` 调用所来自的函数<em>回溯(backtrace)</em>，找出代码中导致问题的部分。为了理解如何使用 `panic!` 的回溯，让我们再看一个例子：这一次，不是代码直接调用宏，而是由于我们代码中的 bug 导致库中的代码调用 `panic!`。示例 9-1 中的代码试图访问向量有效索引范围之外的位置。

<Listing number="9-1" file-name="src/main.rs" caption="尝试访问向量末尾之外的元素，这会导致调用 `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

这里，我们试图访问向量的第 100 个元素（由于索引从 0 开始，它位于索引 99），但向量只有三个元素。在这种情况下，Rust 会 panic。使用 `[]` 本应返回一个元素，但如果传入无效索引，Rust 无法在这里返回任何正确的元素。

在 C 语言中，试图读取数据结构末尾之外的位置属于未定义行为。你可能会得到内存中与该元素位置对应的任意内容，即使那块内存并不属于这个数据结构。这称为<em>缓冲区过度读取(buffer overread)</em>；如果攻击者能够操纵索引，读取存放在该数据结构之后、本不允许其访问的数据，就可能造成安全漏洞。

为了保护程序免受这类漏洞影响，如果试图读取不存在的索引处的元素，Rust 会停止执行并拒绝继续。让我们试试看：

```console
{{#include ../../listings/ch09-error-handling/listing-09-01/output.txt}}
```

这个错误指向 <em>main.rs</em> 的第 4 行，也就是我们试图访问向量索引 99 的位置。

`note:` 行告诉我们，可以设置 `RUST_BACKTRACE` 环境变量来获取导致错误的确切过程的回溯。回溯是为了到达当前位置而调用过的所有函数的列表。Rust 中回溯的工作方式与其他语言相同：阅读回溯的关键是从顶部开始，直到看到自己编写的文件为止，问题就源自那里。该位置上方的行是你的代码所调用的代码，下方的行则是调用了你的代码的代码。这些前后的行可能包括 Rust 核心代码、标准库代码或你正在使用的 crate。让我们把 `RUST_BACKTRACE` 环境变量设为除 `0` 之外的任意值，尝试获取回溯。示例 9-2 展示了你会看到的类似输出。

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="设置环境变量 `RUST_BACKTRACE` 后，显示由 `panic!` 调用生成的回溯">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

输出真不少！你实际看到的确切输出可能因操作系统和 Rust 版本而异。要获得包含这些信息的回溯，必须启用<em>调试符号(debug symbol)</em>。像这里这样，在不使用 `--release` 标志的情况下运行 `cargo build` 或 `cargo run` 时，默认会启用调试符号。

在示例 9-2 的输出中，回溯的第 6 项指向项目中导致问题的那一行：<em>src/main.rs</em> 的第 4 行。如果不希望程序 panic，就应该从回溯中第一处提到我们所编写文件的位置开始调查。在示例 9-1 中，我们刻意编写了会 panic 的代码；修复它的方法是不要请求超出向量索引范围的元素。今后代码发生 panic 时，你需要弄清代码使用了哪些值执行了什么操作而导致 panic，以及代码本应改做什么。

稍后在本章的[“该不该使用 `panic!`”][to-panic-or-not-to-panic]一节中，我们会回到 `panic!`，讨论何时应该以及不应该用它处理错误情况。接下来，我们来看看如何使用 `Result` 从错误中恢复。

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
