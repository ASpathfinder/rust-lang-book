
<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### 把控制权让给运行时

回想[“第一个异步程序”][async-program]<!-- ignore -->一节：在每个等待点，如果正在等待的 future 尚未就绪，Rust 就会给运行时一个机会，让它暂停当前任务并切换到另一个任务。反过来也同样成立：Rust <em>只</em>会在等待点暂停 async 块，并把控制权交还给运行时。等待点之间的一切都是同步的。

这意味着，如果在 async 块中执行大量工作却没有等待点，该 future 就会阻止其他 future 取得进展。有时你会听到人们把这种情况称为一个 future 让其他 future <em>饥饿(starving)</em>。在某些情况下，这可能没什么大不了。不过，如果正在进行某种昂贵的设置、耗时工作，或某个 future 会无限期地持续执行特定任务，就需要考虑何时、何处把控制权交还给运行时。

下面模拟一项耗时操作来说明饥饿问题，再探索如何解决它。示例 17-14 引入一个 `slow` 函数。

<Listing number="17-14" caption="使用 `thread::sleep` 模拟缓慢操作" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

这段代码使用 `std::thread::sleep` 而不是 `trpl::sleep`，所以调用 `slow` 会阻塞当前线程若干毫秒。可以用 `slow` 代表现实世界中既耗时又阻塞的操作。

在示例 17-15 中，我们使用 `slow` 模拟在一对 future 中执行这种 CPU 密集型工作。

<Listing number="17-15" caption="调用 `slow` 函数模拟缓慢操作" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

每个 future 都要执行完一批缓慢操作，<em>之后</em>才把控制权交还给运行时。运行这段代码，会看到以下输出：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

与示例 17-5 使用 `trpl::select` 让获取两个 URL 的 future 相互竞速一样，`select` 仍会在 `a` 完成后立即结束。不过，两个 future 中对 `slow` 的调用并没有交错。`a` future 执行自己的全部工作，直到等待 `trpl::sleep` 调用；接着 `b` future 执行自己的全部工作，直到等待它自己的 `trpl::sleep` 调用；最后 `a` future 完成。为了让两个 future 都能在各自的缓慢任务之间取得进展，需要设置等待点，以便把控制权交还给运行时。这意味着我们需要某种可以等待的东西！

示例 17-15 中已经能看到这种交接：如果移除 `a` future 末尾的 `trpl::sleep`，它将直接完成，而 `b` future <em>完全不会</em>运行。我们先尝试用 `trpl::sleep` 函数让操作轮流取得进展，如示例 17-16 所示。

<Listing number="17-16" caption="使用 `trpl::sleep` 让操作轮流取得进展" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

我们在每次调用 `slow` 之间添加了带等待点的 `trpl::sleep` 调用。现在两个 future 的工作会交错进行：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

`a` future 仍会运行一小段时间后才把控制权交给 `b`，因为它先调用 `slow`，之后才第一次调用 `trpl::sleep`；但从那以后，每当其中一个 future 遇到等待点，两者就会来回切换。本例在每次调用 `slow` 后都这样做，但也可以采用任何最合理的方式拆分工作。

不过，我们并不是真的想在这里<em>休眠</em>：我们希望尽快取得进展，只是需要把控制权交还给运行时。可以直接使用 `trpl::yield_now` 函数实现这一点。在示例 17-17 中，我们把所有 `trpl::sleep` 调用替换为 `trpl::yield_now`。

<Listing number="17-17" caption="使用 `yield_now` 让操作轮流取得进展" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

这段代码更清楚地表达了真实意图，而且可能明显快于使用 `sleep`，因为 `sleep` 所使用的计时器通常对时间粒度有所限制。例如，我们使用的 `sleep` 版本即使接收一纳秒的 `Duration`，也总会至少休眠一毫秒。再次强调，现代计算机速度<em>很快</em>：一毫秒内就能完成许多工作！

这意味着，根据程序还在做什么，async 即使对计算密集型任务也可能有用，因为它提供了一个实用工具来组织程序不同部分之间的关系（代价是异步状态机的开销）。这是一种<em>协作式多任务(cooperative multitasking)</em>，每个 future 都有权通过等待点决定何时交出控制权。因此，每个 future 也有责任避免阻塞太久。在一些基于 Rust 的嵌入式操作系统中，这是<em>唯一</em>的多任务形式！

当然，在现实代码中，通常不会让每行函数调用都与等待点交替出现。以这种方式让出控制权的成本相对较低，但并非免费。在许多情况下，尝试拆分计算密集型任务可能使它明显变慢，所以有时让操作短暂阻塞反而对<em>整体</em>性能更好。务必通过测量确定代码真正的性能瓶颈。不过，如果你发现许多原本预期并发发生的工作却在串行执行，就必须牢记这种底层机制！

### 构建自己的异步抽象

还可以把 future 组合起来创建新的模式。例如，可以使用已有的异步构建模块构建 `timeout` 函数。完成后，得到的结果会成为另一个构建模块，可用于创建更多异步抽象。

示例 17-18 展示了我们期望这个 `timeout` 如何配合缓慢的 future 工作。

<Listing number="17-18" caption="使用设想的 `timeout` 在时间限制内运行缓慢操作" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

来实现它！首先考虑 `timeout` 的 API：

- 它本身必须是异步函数，这样才能等待它。
- 第一个形参应当是要运行的 future。可以把它设为泛型，使其适用于任何 future。
- 第二个形参是最长等待时间。如果使用 `Duration`，就能方便地把它传给 `trpl::sleep`。
- 它应当返回 `Result`。如果 future 成功完成，`Result` 就是包含该 future 所产生值的 `Ok`。如果超时先发生，`Result` 就是包含超时所等待时长的 `Err`。

示例 17-19 展示了这个声明。

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="定义 `timeout` 的签名" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

这满足了类型方面的目标。现在考虑所需的<em>行为</em>：我们希望让传入的 future 与指定时长相互竞速。可以使用 `trpl::sleep` 从时长创建计时器 future，再使用 `trpl::select` 同时运行这个计时器与调用方传入的 future。

在示例 17-20 中，我们通过匹配等待 `trpl::select` 的结果来实现 `timeout`。

<Listing number="17-20" caption="使用 `select` 和 `sleep` 定义 `timeout`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

`trpl::select` 的实现并不公平：它总是按实参传入的顺序进行轮询（其他 `select` 实现可能随机选择先轮询哪个实参）。因此，我们先把 `future_to_try` 传给 `select`，这样即使 `max_time` 非常短，它也有机会完成。如果 `future_to_try` 先完成，`select` 会返回包含 `future_to_try` 输出的 `Left`。如果 `timer` 先完成，`select` 则返回包含计时器输出 `()` 的 `Right`。

如果 `future_to_try` 成功并得到 `Left(output)`，就返回 `Ok(output)`。如果休眠计时器先到期并得到 `Right(())`，就用 `_` 忽略 `()`，改为返回 `Err(max_time)`。

至此，我们用另外两个异步辅助工具构建出了可以工作的 `timeout`。运行代码时，它会在超时后打印失败模式：

```text
Failed after 2 seconds
```

因为 future 可以与其他 future 组合，所以能使用较小的异步构建模块创建非常强大的工具。例如，可以采用相同方法把超时与重试相结合，再把它们用于网络调用等操作（例如示例 17-5 中的操作）。

实践中，通常会直接使用 `async` 和 `await`，其次使用 `select` 等函数和 `join!` 等宏来控制最外层 future 的执行方式。

我们已经了解了多种同时处理多个 future 的方式。接下来看看如何使用 <em>stream</em>，随时间推移按顺序处理多个 future。

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
