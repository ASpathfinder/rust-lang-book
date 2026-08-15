## 优雅停机与清理

示例 21-20 中的代码按照预期，通过线程池异步响应请求。我们收到了一些警告，指出没有直接使用 `workers`、`id` 和 `thread` 字段，这也提醒我们还没有清理任何内容。使用不那么优雅的 <kbd>ctrl</kbd>-<kbd>C</kbd> 方法停止主线程时，所有其他线程也会立刻停止，即使它们正在服务某个请求。

接下来，我们将实现 `Drop` trait，对池中的每个线程调用 `join`，让它们在关闭前完成正在处理的请求。然后实现一种方式，通知线程停止接收新请求并关闭。为了观察代码的实际效果，我们会修改服务器，让它只接收两个请求，然后优雅地关闭线程池。

在这一过程中需要注意一点：这些内容都不会影响处理闭包执行的代码部分，因此即使将线程池用于异步运行时，这里的一切也都相同。

### 为 `ThreadPool` 实现 `Drop` Trait

先为线程池实现 `Drop`。池被丢弃时，所有线程都应执行 join，确保完成自己的工作。示例 21-22 展示了对 `Drop` 实现的首次尝试；这段代码还不能完全工作。

<Listing number="21-22" file-name="src/lib.rs" caption="线程池离开作用域时 join 每个线程">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

首先遍历线程池中的每个 `worker`。这里使用 `&mut`，因为 `self` 是可变引用，而且我们还需要能够改变 `worker`。对于每个 `worker`，打印消息说明这个特定 `Worker` 实例正在关闭，然后对该 `Worker` 实例的线程调用 `join`。如果 `join` 调用失败，则使用 `unwrap` 让 Rust panic，进入不优雅的停机过程。

编译这段代码时会得到以下错误：

```console
{{#include ../../listings/ch21-web-server/listing-21-22/output.txt}}
```

错误告诉我们不能调用 `join`，因为我们只有每个 `worker` 的可变借用，而 `join` 会取得其实参的所有权。要解决这个问题，需要把线程从拥有 `thread` 的 `Worker` 实例中移出，让 `join` 能够消费线程。一种做法是采用示例 18-15 中使用的方法。如果 `Worker` 保存 `Option<thread::JoinHandle<()>>`，就可以对 `Option` 调用 `take` 方法，从 `Some` 变体移出值，并在原处留下 `None` 变体。换句话说，正在运行的 `Worker` 的 `thread` 中会有一个 `Some` 变体；当想清理 `Worker` 时，用 `None` 替换 `Some`，这样 `Worker` 就不再有可运行的线程。

然而，这种情况<em>只会</em>在丢弃 `Worker` 时出现。作为交换，每次访问 `worker.thread` 时都必须处理 `Option<thread::JoinHandle<()>>`。惯用 Rust 经常使用 `Option`，但如果发现自己为了变通而用 `Option` 包装明知始终存在的内容，最好寻找其他办法，让代码更简洁、更不易出错。

在这里，确实存在一个更好的替代方案：`Vec::drain` 方法。它接收一个范围形参，指定要从向量中移除哪些项，并返回这些项的迭代器。传入 `..` 范围语法，会从向量中移除<em>所有</em>值。

因此，需要像下面这样更新 `ThreadPool` 的 `drop` 实现：

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

这样就解决了编译器错误，而且无需对代码做任何其他更改。请注意，因为 panic 期间也可能调用 drop，所以 `unwrap` 也可能 panic 并造成二次 panic，这会立即令程序崩溃，并终止正在进行的任何清理。这对示例程序来说没有问题，但不建议用于生产代码。

### 向线程发出停止监听作业的信号

经过目前所有更改后，代码可以编译，而且没有任何警告。不过坏消息是，这段代码仍未按预期工作。关键在于 `Worker` 实例的线程所运行的闭包逻辑：目前我们调用 `join`，但这不会关闭线程，因为线程会永远 `loop`，寻找作业。如果尝试使用当前的 `drop` 实现丢弃 `ThreadPool`，主线程将永远阻塞，等待第一个线程结束。

要修复这个问题，需要先修改 `ThreadPool` 的 `drop` 实现，再修改 `Worker` 循环。

首先更改 `ThreadPool` 的 `drop` 实现，在等待线程结束前显式丢弃 `sender`。示例 21-23 展示了为显式丢弃 `sender` 而对 `ThreadPool` 所做的更改。与线程的情况不同，这里<em>确实</em>需要使用 `Option`，才能通过 `Option::take` 把 `sender` 从 `ThreadPool` 中移出。

<Listing number="21-23" file-name="src/lib.rs" caption="join 各 `Worker` 线程之前显式丢弃 `sender`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

丢弃 `sender` 会关闭通道，表示不会再发送更多消息。发生这种情况时，`Worker` 实例在无限循环中进行的所有 `recv` 调用都会返回错误。在示例 21-24 中，我们修改 `Worker` 循环，使其在这种情况下优雅地退出循环；这意味着 `ThreadPool` 的 `drop` 实现对线程调用 `join` 时，它们会结束。

<Listing number="21-24" file-name="src/lib.rs" caption="`recv` 返回错误时显式跳出循环">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

为了观察这段代码的实际效果，让我们像示例 21-25 那样修改 `main`，使其只接收两个请求，然后优雅地关闭服务器。

<Listing number="21-25" file-name="src/main.rs" caption="通过退出循环，在服务两个请求后关闭服务器">

```rust,ignore
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

真实世界的 Web 服务器当然不会只服务两个请求就关闭。这段代码只是用于证明优雅停机和清理能够正常工作。

`take` 方法定义在 `Iterator` trait 中，最多只让迭代进行前两项。`ThreadPool` 会在 `main` 末尾离开作用域，届时将运行 `drop` 实现。

使用 `cargo run` 启动服务器并发出三个请求。第三个请求应该会出错，而终端中应看到类似以下内容的输出：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

你看到的 `Worker` ID 和消息打印顺序可能不同。可以通过消息看出代码如何工作：`Worker` 实例 0 和 3 收到了前两个请求。服务器在第二条连接后停止接收连接，`ThreadPool` 上的 `Drop` 实现甚至在 `Worker 3` 开始作业之前就开始执行。丢弃 `sender` 会断开所有 `Worker` 实例，并通知它们关闭。每个 `Worker` 实例都会在断开时打印消息，然后线程池调用 `join`，等待每个 `Worker` 线程结束。

请注意这次特定执行中一个有趣之处：`ThreadPool` 丢弃了 `sender`，而在任何 `Worker` 收到错误之前，我们就尝试 join `Worker 0`。`Worker 0` 尚未从 `recv` 得到错误，所以主线程阻塞，等待 `Worker 0` 结束。与此同时，`Worker 3` 收到一项作业，随后所有线程都收到错误。`Worker 0` 结束后，主线程等待其余 `Worker` 实例结束。此时，它们都已退出循环并停止。

恭喜！现在项目已经完成：我们拥有一个使用线程池异步响应的基本 Web 服务器。服务器可以优雅停机，清理池中的所有线程。

下面列出完整代码，供参考：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

这里还可以继续完成更多工作！如果想进一步增强这个项目，可以考虑以下想法：

- 为 `ThreadPool` 及其公共方法添加更多文档。
- 添加对库功能的测试。
- 把 `unwrap` 调用改为更健壮的错误处理。
- 使用 `ThreadPool` 完成服务 Web 请求以外的任务。
- 在 [crates.io](https://crates.io/) 上寻找一个线程池 crate，并使用它实现类似的 Web 服务器。然后，把它的 API 和健壮性与我们实现的线程池进行比较。

## 小结

做得好！你已经读到本书末尾！感谢你加入这场 Rust 之旅。现在，你已经准备好实现自己的 Rust 项目，并帮助他人完成项目。请记住，还有一个热情友好的 Rustacean 社区，他们很乐意帮助你应对 Rust 旅程中遇到的任何挑战。
