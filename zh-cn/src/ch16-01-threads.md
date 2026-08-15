## 使用线程同时运行代码

在目前的大多数操作系统中，执行中的程序代码运行在<em>进程(process)</em>里，操作系统会同时管理多个进程。在程序内部，也可以让相互独立的部分同时运行。运行这些独立部分的功能称为<em>线程(thread)</em>。例如，Web 服务器可以拥有多个线程，从而同时响应多个请求。

把程序中的计算拆分到多个线程中，同时执行多项任务，可以提高性能，但也会增加复杂性。由于线程能够同时运行，不同线程上的各部分代码以何种顺序运行并没有内在保证。这可能导致以下问题：

- <em>竞态条件(race condition)</em>：线程以不一致的顺序访问数据或资源
- <em>死锁(deadlock)</em>：两个线程相互等待，导致双方都无法继续
- 只在特定情况下发生、难以重现和可靠修复的 bug

Rust 试图减轻使用线程所带来的负面影响，但在多线程环境中编程仍然需要仔细思考，并要求采用与单线程程序不同的代码结构。

编程语言以多种不同方式实现线程，许多操作系统都提供了可供编程语言调用、用来创建新线程的 API。Rust 标准库采用 <em>1:1</em> 线程实现模型，即程序中的每一个语言线程对应一个操作系统线程。有些 crate 实现了其他线程模型，它们相较于 1:1 模型有着不同的权衡。（下一章将介绍的 Rust 异步系统，也提供了另一种并发方式。）

<a id="creating-a-new-thread-with-spawn"></a>

### 使用 `spawn` 创建新线程

要创建新线程，可以调用 `thread::spawn` 函数，并向它传入一个闭包（第 13 章讨论过闭包），其中包含希望在新线程中运行的代码。示例 16-1 从主线程打印一些文本，同时从新线程打印另一些文本。

<Listing number="16-1" file-name="src/main.rs" caption="创建新线程打印一项内容，同时主线程打印另一项内容">

```rust
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

请注意，当 Rust 程序的主线程结束时，所有生成的线程都会被关闭，无论它们是否已完成运行。这个程序每次运行的输出可能略有不同，但看起来会与以下内容相似：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

调用 `thread::sleep` 会强制线程暂停执行一小段时间，让其他线程得以运行。线程可能会轮流运行，但这并无保证，具体取决于操作系统如何调度线程。在这次运行中，尽管生成线程的打印语句在代码中出现得更早，主线程还是率先打印。即使我们要求生成线程打印到 `i` 为 `9`，它也只打印到 `5`，主线程就关闭了。

如果运行这段代码时只看到主线程的输出，或完全看不到交错输出，可以尝试增大范围中的数字，为操作系统在线程间切换创造更多机会。

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

<a id="waiting-for-all-threads-to-finish"></a>

### 等待所有线程结束

示例 16-1 中的代码不仅会因主线程结束而在大多数时候提前停止生成线程；由于线程运行顺序没有保证，我们甚至无法保证生成线程一定会获得运行机会！

可以把 `thread::spawn` 的返回值保存在变量中，解决生成线程不运行或提前结束的问题。`thread::spawn` 的返回类型是 `JoinHandle<T>`。`JoinHandle<T>` 是一个拥有所有权的值；对它调用 `join` 方法时，会等待其所代表的线程结束。示例 16-2 展示了如何使用示例 16-1 中所创建线程的 `JoinHandle<T>`，以及如何调用 `join` 来确保生成线程在 `main` 退出前结束。

<Listing number="16-2" file-name="src/main.rs" caption="保存 `thread::spawn` 返回的 `JoinHandle<T>`，保证线程运行到结束">

```rust
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

对句柄调用 `join` 会阻塞当前正在运行的线程，直到该句柄所代表的线程终止。<em>阻塞(blocking)</em>一个线程，意味着阻止该线程工作或退出。由于我们把 `join` 调用放在主线程的 `for` 循环之后，运行示例 16-2 应当会产生类似以下内容的输出：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

两个线程会继续交替运行，但由于调用了 `handle.join()`，主线程会等待，直到生成线程结束才终止。

不过，下面看看如果把 `handle.join()` 移到 `main` 中的 `for` 循环之前会发生什么：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

主线程会等待生成线程结束，然后再运行自己的 `for` 循环，因此输出不再交错，如下所示：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

诸如在何处调用 `join` 这样的细节，都会影响线程是否同时运行。

<a id="using-move-closures-with-threads"></a>

### 在线程中使用 `move` 闭包

我们经常会对传给 `thread::spawn` 的闭包使用 `move` 关键字，因为闭包随后会取得它从环境中使用的值的所有权，从而把这些值的所有权从一个线程转移到另一个线程。第 13 章[“捕获引用或移动所有权”][capture]<!-- ignore -->一节曾在闭包的语境下讨论 `move`。现在，我们将更专注于 `move` 与 `thread::spawn` 的交互。

请注意，示例 16-1 中传给 `thread::spawn` 的闭包不接收任何实参：生成线程的代码没有使用主线程中的任何数据。要在生成线程中使用主线程的数据，生成线程的闭包必须捕获所需的值。示例 16-3 尝试在主线程中创建一个向量，并在生成线程中使用它。不过，正如马上会看到的，这暂时还行不通。

<Listing number="16-3" file-name="src/main.rs" caption="尝试在另一个线程中使用主线程创建的向量">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

闭包使用了 `v`，所以会捕获 `v`，使它成为闭包环境的一部分。由于 `thread::spawn` 会在新线程中运行这个闭包，我们理应能在新线程中访问 `v`。但编译这个示例时，会得到以下错误：

```console
{{#include ../../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust 会<em>推断</em>如何捕获 `v`。由于 `println!` 只需要对 `v` 的引用，闭包会尝试借用 `v`。但这里存在一个问题：Rust 无法判断生成线程将运行多久，因此不知道对 `v` 的引用是否会始终有效。

示例 16-4 给出了一种更可能导致对 `v` 的引用失效的场景。

<Listing number="16-4" file-name="src/main.rs" caption="一个线程，其闭包尝试捕获主线程中对 `v` 的引用，而主线程会丢弃 `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

如果 Rust 允许运行这段代码，生成线程可能会立刻被放到后台，完全没有开始运行。生成线程内部有一个对 `v` 的引用，但主线程会立即使用第 15 章讨论过的 `drop` 函数丢弃 `v`。之后，当生成线程开始执行时，`v` 已不再有效，对它的引用也同样无效。糟糕！

为了修复示例 16-3 中的编译错误，可以采纳错误消息中的建议：

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

在闭包前添加 `move` 关键字，可以强制闭包取得其所使用值的所有权，而不是让 Rust 推断它应当借用这些值。示例 16-5 展示了对示例 16-3 的修改，它会按我们的预期编译和运行。

<Listing number="16-5" file-name="src/main.rs" caption="使用 `move` 关键字强制闭包取得其所使用值的所有权">

```rust
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

我们可能想用同样的方法修复示例 16-4 中主线程调用 `drop` 的代码，即使用 `move` 闭包。然而，这种修复行不通，因为示例 16-4 所尝试的操作由于另一原因而不被允许。如果给闭包添加 `move`，就会把 `v` 移入闭包环境，从而无法再在主线程中对它调用 `drop`。我们会改为得到以下编译错误：

```console
{{#include ../../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Rust 的所有权规则再次保护了我们！示例 16-3 中的代码之所以报错，是因为 Rust 采取保守做法，只让线程借用 `v`，这意味着理论上主线程可以使生成线程中的引用失效。通过告诉 Rust 把 `v` 的所有权移交给生成线程，我们向 Rust 保证主线程将不再使用 `v`。如果以同样的方式修改示例 16-4，尝试在主线程中使用 `v` 时就违反了所有权规则。`move` 关键字会覆盖 Rust 默认的保守借用行为，但不会让我们违反所有权规则。

现在已经介绍了线程及线程 API 提供的方法，下面看看一些可以使用线程的场景。

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
