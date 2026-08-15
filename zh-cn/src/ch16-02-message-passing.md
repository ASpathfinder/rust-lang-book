<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## 使用消息传递在线程间传输数据

确保安全并发的一种日益流行的方法是消息传递，线程或 actor 通过相互发送包含数据的消息进行通信。[Go 语言文档](https://golang.org/doc/effective_go.html#concurrency)用一句口号概括了这个思想：“不要通过共享内存来通信；而要通过通信来共享内存。”

为了实现发送消息式的并发，Rust 标准库提供了通道的实现。<em>通道(channel)</em>是一个通用编程概念，数据通过它从一个线程发送到另一个线程。

可以把编程中的通道想象成水流的单向通道，例如小溪或河流。如果把橡皮鸭之类的东西放进河里，它会顺流而下，到达水道的终点。

通道有两半：发送器和接收器。发送器这一半是上游，即把橡皮鸭放入河中的位置；接收器这一半则是橡皮鸭最终到达的下游位置。代码的一部分对发送器调用方法，传入希望发送的数据；另一部分则在接收端检查到达的消息。如果发送器或接收器中的任何一半被丢弃，就称通道已经<em>关闭(closed)</em>。

这里将逐步构建一个程序：一个线程生成值并沿通道发送，另一个线程接收这些值并打印。为了演示这一功能，我们会通过通道在线程间发送简单的值。熟悉这种技术后，就可以把通道用于任何需要彼此通信的线程，例如聊天系统，或由多个线程分别执行计算的一部分、再把各部分发送给一个线程汇总结果的系统。

首先，在示例 16-6 中创建一个通道，但不对它执行任何操作。请注意，这段代码暂时无法编译，因为 Rust 无法判断我们希望通过通道发送什么类型的值。

<Listing number="16-6" file-name="src/main.rs" caption="创建一个通道，并把两端分别赋给 `tx` 和 `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

我们使用 `mpsc::channel` 函数创建新通道；`mpsc` 代表<em>多生产者、单消费者(multiple producer, single consumer)</em>。简而言之，Rust 标准库对通道的实现意味着，一个通道可以有多个产生值的<em>发送</em>端，但只能有一个消费这些值的<em>接收</em>端。想象多条小溪汇入一条大河：沿任何一条小溪送出的所有东西，最终都会汇入同一条河。现在先从单个生产者开始，等示例可以工作后，再添加多个生产者。

`mpsc::channel` 函数返回一个元组，第一个元素是发送端（发送器），第二个元素是接收端（接收器）。在许多领域，缩写 `tx` 和 `rx` 传统上分别表示<em>发送器(transmitter)</em>和<em>接收器(receiver)</em>，所以我们这样命名变量，以指明通道的两端。这里使用带有模式的 `let` 语句解构元组；第 19 章将讨论在 `let` 语句中使用模式和解构。现在只需知道，以这种方式使用 `let` 语句，是提取 `mpsc::channel` 所返回元组中各部分的一种便捷方法。

接下来把发送端移入生成线程，让它发送一个字符串，使生成线程与主线程通信，如示例 16-7 所示。这就像把橡皮鸭放入上游的河中，或从一个线程向另一个线程发送聊天消息。

<Listing number="16-7" file-name="src/main.rs" caption='把 `tx` 移入生成线程并发送 `"hi"`'>

```rust
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

我们再次使用 `thread::spawn` 创建新线程，然后使用 `move` 把 `tx` 移入闭包，使生成线程拥有 `tx`。生成线程必须拥有发送器，才能通过通道发送消息。

发送器有一个 `send` 方法，接收希望发送的值。`send` 方法返回 `Result<T, E>` 类型；因此，如果接收器已经被丢弃，没有地方可供发送值，发送操作就会返回错误。本例调用 `unwrap`，在出错时 panic。但在真实应用程序中，应当正确处理错误：可回到第 9 章复习恰当的错误处理策略。

示例 16-8 将在主线程中从接收器取得值。这就像从河流末端的水中捞起橡皮鸭，或接收一条聊天消息。

<Listing number="16-8" file-name="src/main.rs" caption='在主线程中接收值 `"hi"` 并打印它'>

```rust
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

接收器有两个实用方法：`recv` 和 `try_recv`。这里使用 `recv`，它是 <em>receive</em> 的缩写；该方法会阻塞主线程的执行，等待有值沿通道发送过来。值一旦发送，`recv` 就在 `Result<T, E>` 中返回它。发送器关闭时，`recv` 会返回错误，表示不会再有值到来。

`try_recv` 方法不会阻塞，而是立即返回 `Result<T, E>`：如果有消息可用，返回保存消息的 `Ok` 值；如果这次没有任何消息，则返回 `Err` 值。当这个线程在等待消息期间还有其他工作要做时，使用 `try_recv` 很有帮助：可以编写一个循环，每隔一段时间调用 `try_recv`，有消息可用就处理，否则先执行一会儿其他工作，然后再次检查。

为简单起见，本例使用 `recv`；主线程除了等待消息之外没有其他工作，因此阻塞主线程是合适的。

运行示例 16-8 中的代码，会看到主线程打印出这个值：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

完美！

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### 通过通道转移所有权

所有权规则在消息发送中起着至关重要的作用，因为它们能帮助你编写安全的并发代码。在整个 Rust 程序中思考所有权，其优势之一就是能防止并发编程中的错误。下面做一个实验，展示通道和所有权如何协同防止问题：把 `val` 值沿通道发送出去<em>之后</em>，尝试在生成线程中使用它。试着编译示例 16-9 中的代码，看看为何不允许这样做。

<Listing number="16-9" file-name="src/main.rs" caption="尝试在沿通道发送 `val` 后继续使用它">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

这里通过 `tx.send` 沿通道发送 `val` 后，又尝试打印它。允许这样做会很糟糕：值被发送给另一个线程后，那个线程可能在我们尝试再次使用它之前修改或丢弃它。另一个线程所作的修改可能因数据不一致或不存在而导致错误或意外结果。不过，如果尝试编译示例 16-9 中的代码，Rust 会给出错误：

```console
{{#include ../../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

我们的并发错误导致了编译期错误。`send` 函数取得其形参的所有权；值被移动后，接收器取得它的所有权。这阻止我们在发送值后意外地再次使用它，所有权系统会检查一切是否正确。

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### 发送多个值

示例 16-8 中的代码能够编译和运行，但没有清楚表明两个独立线程正通过通道相互通信。

在示例 16-10 中，我们进行了一些修改，以证明示例 16-8 中的代码正在并发运行：生成线程现在会发送多条消息，并在每条消息之间暂停一秒。

<Listing number="16-10" file-name="src/main.rs" caption="发送多条消息，并在每条消息之间暂停">

```rust,noplayground
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

这一次，生成线程拥有一个由字符串构成的向量，希望将它们发送给主线程。我们迭代这些字符串，逐个发送，并在每次发送之间调用 `thread::sleep` 函数，以一秒的 `Duration` 值暂停。

在主线程中，不再显式调用 `recv` 函数，而是把 `rx` 当作迭代器。每收到一个值，就将它打印出来。通道关闭时，迭代也会结束。

运行示例 16-10 中的代码时，应当看到以下输出，每行之间暂停一秒：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

由于主线程的 `for` 循环中没有任何暂停或延迟代码，可以看出主线程正在等待接收来自生成线程的值。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### 创建多个生产者

前面提到，`mpsc` 是<em>多生产者、单消费者</em>的缩写。现在实际运用 `mpsc`，扩展示例 16-10 中的代码，创建多个线程，让它们都向同一个接收器发送值。如示例 16-11 所示，可以通过克隆发送器来实现。

<Listing number="16-11" file-name="src/main.rs" caption="从多个生产者发送多条消息">

```rust,noplayground
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

这一次，在创建第一个生成线程前，我们对发送器调用 `clone`。这样会得到一个可传给第一个生成线程的新发送器。原始发送器则传给第二个生成线程。于是得到两个线程，每个线程都向同一个接收器发送不同的消息。

运行代码时，输出应当类似以下内容：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

根据系统不同，你可能会看到顺序不同的值。这正是并发既有趣又困难的原因。如果尝试在不同线程中为 `thread::sleep` 提供各种不同的值，每次运行都会更加不确定，并产生不同的输出。

现在已经了解通道的工作方式，下面看看另一种并发方法。
