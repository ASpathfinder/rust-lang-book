<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## 使用 Async 实现并发

本节将用 async 处理第 16 章中使用线程解决过的一些并发挑战。由于那里已经讨论了许多关键概念，本节将重点关注线程与 future 之间的区别。

在许多情况下，使用 async 处理并发的 API 与使用线程的 API 非常相似；另一些情况下，它们却有很大不同。即使线程与 async 的 API <em>看起来</em>相似，其行为通常也不同，而且几乎总是具有不同的性能特征。

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### 使用 `spawn_task` 创建新任务

第 16 章[“使用 `spawn` 创建新线程”][thread-spawn]<!-- ignore -->一节处理的第一个操作，是在两个独立线程中递增计数。现在用 async 完成相同的事情。`trpl` crate 提供的 `spawn_task` 函数看起来与 `thread::spawn` API 非常相似，还提供了 `thread::sleep` API 的异步版本 `sleep` 函数。可以把两者结合起来实现计数示例，如示例 17-6 所示。

<Listing number="17-6" caption="创建一个新任务来打印一项内容，同时主任务打印另一项内容" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

作为起点，我们在 `main` 函数中设置 `trpl::block_on`，让顶层函数能够异步执行。

> 注意：从本章此处开始，每个示例都会在 `main` 中包含完全相同的 `trpl::block_on` 包装代码，因此我们会像经常省略 `main` 那样省略它。请记得在自己的代码中包含它！

然后在该代码块中编写两个循环，每个循环都包含一次 `trpl::sleep` 调用，在发送下一条消息前等待半秒（500 毫秒）。其中一个循环放在 `trpl::spawn_task` 的函数体中，另一个放在顶层 `for` 循环中。我们还在 `sleep` 调用后添加 `await`。

这段代码的行为与基于线程的实现相似，其中也包括一点：在自己的终端中运行时，可能会看到消息以不同顺序出现。

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

这个版本会在主 async 块函数体中的 `for` 循环结束后立即停止，因为 `main` 函数结束时，由 `spawn_task` 生成的任务会被关闭。如果希望任务一直运行到完成，就需要使用 join 句柄等待第一个任务完成。使用线程时，我们调用 `join` 方法“阻塞”，直到线程运行完毕。在示例 17-7 中，可以使用 `await` 完成同样的事情，因为任务句柄本身就是一个 future。它的 `Output` 类型是 `Result`，因此等待后还要对它调用 `unwrap`。

<Listing number="17-7" caption="对 join 句柄使用 `await`，让任务运行到完成" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

更新后的版本会运行到<em>两个</em>循环都结束：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

到目前为止，async 和线程似乎只是使用不同语法给出了相似结果：对 join 句柄使用 `await`，而不是调用 `join`；并等待 `sleep` 调用。

更大的区别在于，实现这一点不需要再生成一个操作系统线程。事实上，这里甚至不需要生成任务。因为 async 块会编译为匿名 future，所以可以把每个循环分别放入 async 块，再让运行时使用 `trpl::join` 函数把两者都运行到完成。

第 16 章[“等待所有线程结束”][join-handles]<!-- ignore -->一节展示了如何对调用 `std::thread::spawn` 时返回的 `JoinHandle` 类型使用 `join` 方法。`trpl::join` 函数与它相似，但用于 future。向它提供两个 future 时，它会产生一个新的单一 future；当传入的两个 future <em>都</em>完成后，新 future 的输出是一个元组，包含每个 future 的输出。因此，在示例 17-8 中，我们使用 `trpl::join` 等待 `fut1` 和 `fut2` 都结束。我们<em>不</em>等待 `fut1` 和 `fut2` 本身，而是等待 `trpl::join` 产生的新 future。我们忽略其输出，因为它只是一个包含两个单元值的元组。

<Listing number="17-8" caption="使用 `trpl::join` 等待两个匿名 future" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

运行时，可以看到两个 future 都运行到完成：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

现在，每次都会看到完全相同的顺序，这与线程以及示例 17-7 中的 `trpl::spawn_task` 非常不同。这是因为 `trpl::join` 函数是<em>公平(fair)</em>的：它同等频率地检查每个 future，在两者之间交替；如果另一个已经就绪，就绝不会让其中一个抢先太多。使用线程时，操作系统决定检查哪个线程以及让它运行多久。使用异步 Rust 时，运行时决定检查哪个任务。（实践中的细节会更加复杂，因为异步运行时在底层管理并发时可能使用操作系统线程，因此保证公平可能会给运行时带来更多工作，但仍然能够做到！）运行时不必为任何给定操作保证公平，通常会提供不同的 API，让你选择是否需要公平。

尝试以下几种等待 future 的变化，看看会发生什么：

- 移除包围其中一个或两个循环的 async 块。
- 定义每个 async 块后立即等待它。
- 只把第一个循环包裹在 async 块中，并在第二个循环的函数体之后等待得到的 future。

作为额外挑战，看看能否在运行代码<em>之前</em>判断每种情况下的输出！

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

<a id="sending-data-between-two-tasks-using-message-passing"></a>

### 使用消息传递在两个任务间发送数据

在 future 之间共享数据也会让人感到熟悉：我们会再次使用消息传递，但这次使用类型和函数的异步版本。与第 16 章[“使用消息传递在线程间传输数据”][message-passing-threads]<!-- ignore -->一节略有不同，我们会换一条路径，以展示基于线程和基于 future 的并发之间的一些关键差异。在示例 17-9 中，我们从单个 async 块开始，<em>不</em>像生成独立线程那样生成单独的任务。

<Listing number="17-9" caption="创建异步通道，并把两端分别赋给 `tx` 和 `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

这里使用 `trpl::channel`，它是第 16 章在线程中使用过的多生产者、单消费者通道 API 的异步版本。异步 API 与基于线程的版本只有少许不同：它使用可变接收器 `rx`，而不是不可变接收器；它的 `recv` 方法产生一个需要等待的 future，而不是直接产生值。现在可以从发送器向接收器发送消息。请注意，不必生成单独的线程，甚至也不必生成任务，只需等待 `rx.recv` 调用。

`std::mpsc::channel` 中的同步 `Receiver::recv` 方法会阻塞，直到收到消息。`trpl::Receiver::recv` 方法不会，因为它是异步的。它不会阻塞，而是把控制权交还给运行时，直到收到消息或通道的发送端关闭。相比之下，我们不等待 `send` 调用，因为它不会阻塞。之所以不需要阻塞，是因为发送消息所进入的通道是无界的。

> 注意：由于这些异步代码全都运行在 `trpl::block_on` 调用中的 async 块内，其中的一切都能避免阻塞。不过，它<em>外部</em>的代码会阻塞，等待 `block_on` 函数返回。这正是 `trpl::block_on` 函数的全部意义：它让你能够<em>选择</em>在何处阻塞以等待一组异步代码，从而选择在何处于同步代码与异步代码之间转换。

请注意这个示例中的两点。第一，消息会立即到达。第二，虽然这里使用了 future，但还没有并发。示例中的一切都按顺序发生，与完全不涉及 future 时一样。

先解决第一点：发送一系列消息，并在消息之间休眠，如示例 17-10 所示。

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="通过异步通道发送和接收多条消息，并在每条消息之间使用 `await` 休眠" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

除了发送消息，还需要接收消息。在这个例子中，因为知道会收到多少条消息，可以手动调用四次 `rx.recv().await`。不过在现实世界中，我们通常会等待某个<em>未知</em>数量的消息，因此需要一直等待，直到确定不会再有消息。

示例 16-10 使用 `for` 循环处理从同步通道接收的所有项。然而，Rust 尚且无法用 `for` 循环处理<em>异步产生的</em>一系列项，因此需要使用一种之前未见过的循环：`while let` 条件循环。它是第 6 章[“使用 `if let` 和 `let...else` 实现简洁控制流”][if-let]<!-- ignore -->一节中 `if let` 结构的循环版本。只要指定的模式继续与值匹配，循环就会继续执行。

`rx.recv` 调用产生一个 future，我们等待它。运行时会暂停该 future，直到它就绪。消息每到达一次，future 就会解析为一次 `Some(message)`。通道关闭时，无论是否曾有<em>任何</em>消息到达，future 都会改为解析成 `None`，表示不会再有值，因此应当停止轮询，也就是停止等待。

`while let` 循环把这些内容组合起来。如果调用 `rx.recv().await` 的结果是 `Some(message)`，就能访问消息并在循环体中使用它，与 `if let` 中的做法一样。如果结果是 `None`，循环就会结束。循环每完成一次，都会再次遇到等待点，因此运行时再次暂停它，直到另一条消息到达。

现在，代码成功发送并接收了所有消息。遗憾的是，仍然存在几个问题。首先，消息并不是每隔半秒到达一次，而是在程序启动 2 秒（2,000 毫秒）后同时到达。其次，这个程序永远不会退出！它会一直等待新消息。你需要使用 <kbd>ctrl</kbd>-<kbd>C</kbd> 将其关闭。

#### 单个 Async 块内的代码线性执行

先看看为什么消息会在完整延迟过后同时到达，而不是每条消息之间都有延迟。在给定的 async 块内，`await` 关键字在代码中的出现顺序，也就是程序运行时执行它们的顺序。

示例 17-10 中只有一个 async 块，因此其中一切都线性运行，仍然没有并发。所有 `tx.send` 调用依次发生，中间穿插所有 `trpl::sleep` 调用及其关联的等待点。只有在这之后，`while let` 循环才能开始经过 `recv` 调用上的任何等待点。

为了获得想要的行为，即每条消息之间出现休眠延迟，需要把 `tx` 和 `rx` 操作分别放入自己的 async 块中，如示例 17-11 所示。然后，运行时可以像示例 17-8 那样，使用 `trpl::join` 分别执行它们。我们再次等待 `trpl::join` 调用的结果，而不是单独等待各个 future。如果依次等待各个 future，最终只会回到顺序执行流，而这正是我们<em>不</em>想要的。

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="把 `send` 和 `recv` 分别放入自己的 `async` 块，并等待这些代码块的 future" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

使用示例 17-11 中更新后的代码，消息会以 500 毫秒为间隔打印，而不是在 2 秒后一下子全部出现。

#### 把所有权移入 Async 块

不过，由于 `while let` 循环与 `trpl::join` 交互的方式，程序仍然永远不会退出：

- `trpl::join` 返回的 future 只有在传给它的<em>两个</em> future 都完成后才会完成。
- `tx_fut` future 会在发送完 `vals` 中最后一条消息并结束休眠后完成。
- `rx_fut` future 要等到 `while let` 循环结束才会完成。
- `while let` 循环要等到等待 `rx.recv` 产生 `None` 才会结束。
- 只有通道的另一端关闭后，等待 `rx.recv` 才会返回 `None`。
- 只有调用 `rx.close`，或发送端 `tx` 被丢弃时，通道才会关闭。
- 我们没有在任何地方调用 `rx.close`，而 `tx` 要等到传给 `trpl::block_on` 的最外层 async 块结束后才会被丢弃。
- 这个代码块无法结束，因为它阻塞在等待 `trpl::join` 完成；这样又回到了列表顶部。

目前，发送消息的 async 块只会<em>借用</em> `tx`，因为发送消息并不要求所有权；但如果能够把 `tx` <em>移入</em>该 async 块，它就会在代码块结束时被丢弃。第 13 章[“捕获引用或移动所有权”][capture-or-move]<!-- ignore -->一节介绍了如何对闭包使用 `move` 关键字；第 16 章[“在线程中使用 `move` 闭包”][move-threads]<!-- ignore -->一节还讨论了使用线程时常常需要把数据移入闭包。同样的基本机制也适用于 async 块，所以 `move` 关键字用于 async 块的方式与用于闭包一样。

在示例 17-12 中，我们把用于发送消息的代码块从 `async` 改为 `async move`。

<Listing number="17-12" caption="修订示例 17-11 中的代码，使其在完成后正确关闭" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

运行<em>这个</em>版本时，它会在最后一条消息发送并接收后正常关闭。接下来看看要从多个 future 发送数据，需要进行哪些修改。

#### 使用 `join!` 宏合并多个 Future

这个异步通道同样是多生产者通道，因此如果希望从多个 future 发送消息，可以对 `tx` 调用 `clone`，如示例 17-13 所示。

<Listing number="17-13" caption="在 async 块中使用多个生产者" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

首先在第一个 async 块外部克隆 `tx`，创建 `tx1`。与之前对 `tx` 所做的一样，把 `tx1` 移入该代码块。随后，把原始 `tx` 移入一个<em>新的</em> async 块，在其中以略慢的延迟发送更多消息。我们恰好把这个新 async 块放在用于接收消息的 async 块之后，但也完全可以放在它之前。关键在于等待 future 的顺序，而不是创建它们的顺序。

两个用于发送消息的 async 块都必须是 `async move` 块，以便 `tx` 和 `tx1` 在相应代码块结束时被丢弃。否则，又会回到最初那个无限循环。

最后，我们从 `trpl::join` 改用 `trpl::join!` 来处理额外的 future：如果在编译期知道 future 的数量，`join!` 宏可以等待任意数量的 future。本章稍后会讨论如何等待一个数量未知的 future 集合。

现在可以看到两个发送 future 发出的全部消息；由于它们在发送后采用略有不同的延迟，这些消息也会以相应的不同间隔被接收：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

我们已经探讨了如何使用消息传递在 future 间发送数据、async 块内的代码如何顺序运行、如何把所有权移入 async 块，以及如何合并多个 future。接下来讨论如何以及为何告诉运行时，它可以切换到另一个任务。

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
