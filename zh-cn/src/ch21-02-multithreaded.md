<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## 从单线程服务器到多线程服务器

目前，服务器会依次处理每个请求，意味着第一个连接处理完之前，它不会处理第二个连接。如果服务器收到越来越多的请求，这种串行执行会变得越来越低效。如果服务器收到一个需要长时间处理的请求，即使新请求可以迅速处理，后续请求也必须等待这个耗时请求结束。我们需要修复这一点，但首先来实际观察问题。

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### 模拟缓慢请求

我们将观察一个处理缓慢的请求如何影响对当前服务器实现发出的其他请求。示例 21-10 实现了对 <em>/sleep</em> 请求的处理，以模拟缓慢响应：服务器会在响应前休眠五秒。

<Listing number="21-10" file-name="src/main.rs" caption="通过休眠五秒模拟缓慢请求">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

现在有三种情况，所以我们从 `if` 改用了 `match`。需要显式地匹配 `request_line` 的切片，才能针对字符串字面量值执行模式匹配；与相等性方法不同，`match` 不会自动进行引用与解引用。

第一个分支与示例 21-9 中的 `if` 块相同。第二个分支匹配对 <em>/sleep</em> 的请求。收到该请求时，服务器会休眠五秒，然后再渲染成功的 HTML 页面。第三个分支与示例 21-9 中的 `else` 块相同。

可以看出我们的服务器有多原始：真正的库会用远没这么冗长的方式识别多个请求！

使用 `cargo run` 启动服务器。然后打开两个浏览器窗口：一个访问 <em>http://127.0.0.1:7878</em>，另一个访问 <em>http://127.0.0.1:7878/sleep</em>。如果像之前那样多次输入 <em>/</em> URI，会看到它迅速响应。但是，如果先输入 <em>/sleep</em>，再加载 <em>/</em>，会发现 <em>/</em> 必须等 `sleep` 完整休眠五秒后才会加载。

可以采用多种技术，避免请求在缓慢请求之后积压，包括使用第 17 章中用过的 async；我们要实现的是线程池。

### 使用线程池提升吞吐量

线程池是一组已生成、随时等待处理任务的线程。程序收到新任务时，会把任务分配给池中的一个线程，由该线程处理。第一个线程处理任务期间，池中的其余线程仍可处理传入的其他任务。第一个线程处理完任务后，会返回空闲线程池，准备处理新任务。线程池让你能够并发处理连接，从而提高服务器的吞吐量。

我们会把池中的线程数限制为一个较小值，以防范 DoS 攻击；如果让程序为每个传入请求创建新线程，有人向服务器发出一千万个请求就可能耗尽服务器的全部资源，令请求处理陷入停顿，造成严重破坏。

因此，我们不生成无限数量的线程，而是让固定数量的线程在池中等待。传入的请求会发送给池来处理。池维护一个传入请求队列，池中的每个线程从队列取出一个请求，处理它，然后再向队列索取另一个请求。采用这种设计，我们最多可以并发处理 <em>`N`</em> 个请求，其中 <em>`N`</em> 是线程数。如果每个线程都在响应长时间运行的请求，后续请求仍可能在队列中积压；但到达这种情况之前，我们能处理的长时间请求数量已经增加。

这项技术只是提升 Web 服务器吞吐量的众多方式之一。还可以探索 fork/join 模型、单线程异步 I/O 模型和多线程异步 I/O 模型。如果对这个主题感兴趣，可以进一步阅读其他方案，并尝试实现它们；使用 Rust 这样的低级语言，所有这些选择都能实现。

开始实现线程池之前，先讨论使用这个池时应该是什么样子。尝试设计代码时，先编写客户端接口有助于指导设计。让代码 API 具有你希望调用的结构，再在该结构内部实现功能；而不是先实现功能，再设计公共 API。

与第 12 章项目使用测试驱动开发类似，这里将使用<em>编译器驱动开发(compiler-driven development)</em>。我们先编写调用所需函数的代码，再查看编译器错误，确定下一步应做哪些更改才能让代码工作。不过在此之前，先探索一种不会作为起点采用的技术。

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### 为每个请求生成一个线程

首先探索一下，如果代码真的为每条连接创建新线程，它可能是什么样子。正如之前所说，由于可能生成无限数量线程所带来的问题，这不是最终方案；但它可以作为让多线程服务器先运行起来的起点。然后再加入线程池进行改进，对比两种方案也会更加容易。

示例 21-11 展示了要对 `main` 做出的更改：在 `for` 循环内为每个 stream 生成一个新线程来处理它。

<Listing number="21-11" file-name="src/main.rs" caption="为每个 stream 生成一个新线程">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

第 16 章介绍过，`thread::spawn` 会创建新线程，然后在新线程中运行闭包里的代码。如果运行这段代码，在浏览器中加载 <em>/sleep</em>，再在另两个浏览器标签页中加载 <em>/</em>，确实会看到对 <em>/</em> 的请求无需等待 <em>/sleep</em> 完成。然而正如我们提到的，这最终会压垮系统，因为不断创建新线程却没有任何限制。

你可能还记得第 17 章的内容：这正是 async 和 await 真正大显身手的场景！构建线程池时，请记住这一点，并思考使用 async 时哪些地方会不同、哪些会相同。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

<a id="creating-a-finite-number-of-threads"></a>

#### 创建有限数量的线程

我们希望线程池以相似、熟悉的方式工作，这样从线程切换到线程池时，使用 API 的代码无需发生大幅改变。示例 21-12 展示了理想中的 `ThreadPool` 结构体接口，我们想用它取代 `thread::spawn`。

<Listing number="21-12" file-name="src/main.rs" caption="理想的 `ThreadPool` 接口">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

我们使用 `ThreadPool::new` 创建具有可配置线程数的新线程池，这里是四个线程。然后在 `for` 循环中，`pool.execute` 采用与 `thread::spawn` 相似的接口：它接收一个闭包，池应当为每个 stream 运行该闭包。需要实现 `pool.execute`，让它接收闭包并交给池中的一个线程运行。这段代码尚无法编译，但我们会尝试编译，让编译器指导如何修复它。

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### 使用编译器驱动开发构建 `ThreadPool`

对 <em>src/main.rs</em> 做出示例 21-12 中的更改，然后使用 `cargo check` 给出的编译器错误来推动开发。得到的第一个错误如下：

```console
{{#include ../../listings/ch21-web-server/listing-21-12/output.txt}}
```

很好！这个错误告诉我们需要 `ThreadPool` 类型或模块，所以现在就来构建它。`ThreadPool` 实现与 Web 服务器所做的工作种类无关。因此，让我们把 `hello` crate 从二进制 crate 改为库 crate，用来容纳 `ThreadPool` 实现。改为库 crate 后，这个独立线程池库也可以用于任何想通过线程池完成的工作，不仅限于服务 Web 请求。

创建 <em>src/lib.rs</em> 文件并加入以下内容；这是目前最简单的 `ThreadPool` 结构体定义：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

然后编辑 <em>main.rs</em>，在 <em>src/main.rs</em> 顶部添加以下代码，把库 crate 中的 `ThreadPool` 引入作用域：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

这段代码仍然无法工作，但我们再次检查，以取得下一个需要解决的错误：

```console
{{#include ../../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

这个错误指出，接下来需要为 `ThreadPool` 创建名为 `new` 的关联函数。我们还知道，`new` 需要有一个可以接收 `4` 作为实参的形参，并应返回 `ThreadPool` 实例。让我们实现具有这些特征的最简单 `new` 函数：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

我们为 `size` 形参选择 `usize` 类型，因为负数个线程没有意义。我们还知道，会把这个 `4` 用作线程集合中的元素数量；正如第 3 章[“整数类型”][integer-types]<!-- ignore -->一节讨论的那样，这正是 `usize` 类型的用途。

再次检查代码：

```console
{{#include ../../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

现在出现错误是因为 `ThreadPool` 上没有 `execute` 方法。回想[“创建有限数量的线程”](#creating-a-finite-number-of-threads)<!-- ignore -->一节，我们决定线程池应拥有与 `thread::spawn` 相似的接口。此外，我们会实现 `execute` 函数，让它接收给定闭包并交给池中的空闲线程运行。

我们把 `ThreadPool` 上的 `execute` 方法定义为接收闭包形参。回想第 13 章[“从闭包中移出捕获的值”][moving-out-of-closures]<!-- ignore -->一节，我们可以通过三种不同 trait 接收闭包作为形参：`Fn`、`FnMut` 和 `FnOnce`。这里需要决定使用哪种闭包。我们知道最终会做一些类似标准库 `thread::spawn` 实现的事情，所以可以查看 `thread::spawn` 的签名对形参施加了哪些 bound。文档显示如下：

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

这里我们关心的是类型参数 `F`；类型参数 `T` 与返回值有关，不在本节关注范围内。可以看到，`spawn` 把 `FnOnce` 用作 `F` 的 trait bound。这很可能也是我们需要的，因为最终会把 `execute` 收到的实参传给 `spawn`。我们还能进一步确认 `FnOnce` 是所需 trait，因为运行请求的线程只会执行该请求的闭包一次，与 `FnOnce` 中的 `Once` 相符。

类型参数 `F` 还有 trait bound `Send` 和生命周期 bound `'static`，它们在这里也很有用：需要 `Send` 把闭包从一个线程传到另一个线程；需要 `'static`，因为不知道线程要执行多久。让我们在 `ThreadPool` 上创建 `execute` 方法，接收具有这些 bound 的 `F` 类型泛型形参：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

我们仍在 `FnOnce` 后使用 `()`，因为这里的 `FnOnce` 表示一个不接收形参、返回单元类型 `()` 的闭包。与函数定义一样，返回类型可以从签名中省略；但即使没有形参，仍然需要圆括号。

同样，这是 `execute` 方法最简单的实现：它什么也不做，但我们现在只是在尝试让代码通过编译。再次检查：

```console
{{#include ../../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

编译成功了！不过请注意，如果尝试执行 `cargo run` 并在浏览器中发出请求，会看到本章开头见过的浏览器错误。我们的库实际上还没有调用传给 `execute` 的闭包！

> 注意：对于 Haskell 和 Rust 等拥有严格编译器的语言，你可能听过一句话：“代码只要能编译，就能工作。”但这句话并非普遍成立。我们的项目能够编译，却完全什么也不做！如果正在构建真正而完整的项目，现在正适合开始编写单元测试，检查代码既能编译，<em>又</em>具备所需行为。

思考一下：如果这里要执行的是 future 而不是闭包，会有什么不同？

#### 验证 `new` 中的线程数

我们还没有对传给 `new` 和 `execute` 的形参做任何事情。现在来实现这些函数体，使其具备所需行为。首先考虑 `new`。之前为 `size` 形参选择了无符号类型，因为负数个线程组成的池没有意义。然而，包含零个线程的池同样没有意义，而零是完全有效的 `usize`。我们会添加代码，在返回 `ThreadPool` 实例前检查 `size` 是否大于零；如果收到零，则使用 `assert!` 宏让程序 panic，如示例 21-13 所示。

<Listing number="21-13" file-name="src/lib.rs" caption="实现 `ThreadPool::new`，使其在 `size` 为零时 panic">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

我们还通过文档注释为 `ThreadPool` 添加了一些文档。请注意，按照第 14 章讨论的良好文档实践，我们添加了一个指出函数可能在哪些情况下 panic 的小节。试着运行 `cargo doc --open` 并点击 `ThreadPool` 结构体，看看生成的 `new` 文档是什么样子！

也可以不添加这里的 `assert!` 宏，而是像 I/O 项目示例 12-9 中的 `Config::build` 一样，把 `new` 改成 `build` 并返回 `Result`。但这里我们决定，尝试创建没有任何线程的线程池应当属于不可恢复错误。如果有兴趣挑战自己，请尝试编写一个具有以下签名、名为 `build` 的函数，与 `new` 函数进行比较：

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### 创建存储线程的空间

现在有办法确认池中要存储的线程数有效，便可以在返回 `ThreadPool` 结构体之前创建这些线程，并将它们存入结构体。但要如何“存储”线程呢？再看看 `thread::spawn` 的签名：

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` 函数返回 `JoinHandle<T>`，其中 `T` 是闭包的返回类型。让我们也尝试使用 `JoinHandle`，看看会发生什么。在这里，传给线程池的闭包将处理连接而不返回任何内容，所以 `T` 是单元类型 `()`。

示例 21-14 中的代码可以编译，但尚未创建任何线程。我们修改了 `ThreadPool` 的定义，让它保存 `thread::JoinHandle<()>` 实例的向量；以 `size` 作为容量初始化该向量；设置一个将运行某些代码以创建线程的 `for` 循环；然后返回包含这些线程的 `ThreadPool` 实例。

<Listing number="21-14" file-name="src/lib.rs" caption="为 `ThreadPool` 创建用于保存线程的向量">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

我们把 `std::thread` 引入库 crate 的作用域，因为 `ThreadPool` 中向量的项类型使用了 `thread::JoinHandle`。

收到有效的 size 后，`ThreadPool` 创建一个能够保存 `size` 项的新向量。`with_capacity` 函数与 `Vec::new` 执行相同任务，但有一个重要区别：它会预先在向量中分配空间。因为知道需要在向量中存储 `size` 个元素，所以预先执行这次分配会比使用 `Vec::new` 稍微高效一些；后者会随着插入元素而调整自身大小。

再次运行 `cargo check` 时，它应该会成功。

<!-- Old headings. Do not remove or links may break. -->
<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### 从 `ThreadPool` 向线程发送代码

我们在示例 21-14 的 `for` 循环中留下了一条有关创建线程的注释。这里将了解实际如何创建线程。标准库提供 `thread::spawn` 来创建线程；`thread::spawn` 期望获得一些线程一经创建就应运行的代码。但在当前情况下，我们想创建线程，让它们<em>等待</em>稍后发送的代码。标准库的线程实现没有提供任何办法直接做到这一点，必须自行实现。

为了实现这种行为，我们会在 `ThreadPool` 与线程之间引入一个新的数据结构来管理它。这个数据结构称为 <em>Worker</em>，是池实现中的常用术语。`Worker` 取得需要运行的代码，并在自己的线程中运行它。

可以把它想象成餐厅厨房中工作的人：工作人员等待顾客的订单到来，然后负责接单并完成订单。

在线程池中，我们不再存储 `JoinHandle<()>` 实例的向量，而是存储 `Worker` 结构体的实例。每个 `Worker` 保存一个 `JoinHandle<()>` 实例。然后，我们为 `Worker` 实现一个方法，它接收要运行的代码闭包，并把闭包发送给已经运行的线程执行。还会给每个 `Worker` 一个 `id`，以便记录日志或调试时区分池中的不同 `Worker` 实例。

下面是创建 `ThreadPool` 时将发生的新过程。按照这种方式设置好 `Worker` 后，再实现把闭包发送给线程的代码：

1. 定义一个保存 `id` 和 `JoinHandle<()>` 的 `Worker` 结构体。
2. 修改 `ThreadPool`，使其保存 `Worker` 实例的向量。
3. 定义 `Worker::new` 函数，它接收一个 `id` 数字，返回保存该 `id` 以及通过空闭包生成之线程的 `Worker` 实例。
4. 在 `ThreadPool::new` 中，使用 `for` 循环计数器生成 `id`，以该 `id` 创建新 `Worker`，再把 `Worker` 存入向量。

如果愿意接受挑战，请在查看示例 21-15 的代码之前，自行尝试实现这些更改。

准备好了吗？示例 21-15 给出了进行上述修改的一种方式。

<Listing number="21-15" file-name="src/lib.rs" caption="修改 `ThreadPool`，让它保存 `Worker` 实例，而不是直接保存线程">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

我们把 `ThreadPool` 上字段的名称从 `threads` 改为 `workers`，因为它现在保存的是 `Worker` 实例，而不是 `JoinHandle<()>` 实例。以 `for` 循环中的计数器作为 `Worker::new` 的实参，并把每个新 `Worker` 存入名为 `workers` 的向量。

外部代码（例如 <em>src/main.rs</em> 中的服务器）不需要知道 `ThreadPool` 内部使用 `Worker` 结构体的实现细节，所以把 `Worker` 结构体及其 `new` 函数设为私有。`Worker::new` 函数使用传入的 `id`，并存储一个通过空闭包生成新线程所创建的 `JoinHandle<()>` 实例。

> 注意：如果系统资源不足，导致操作系统无法创建线程，`thread::spawn` 就会 panic。即使已经成功创建了一些线程，这也会导致整个服务器 panic。为简单起见，这种行为没有问题；但在生产环境的线程池实现中，你很可能希望使用 [`std::thread::Builder`][builder]<!-- ignore --> 及其会返回 `Result` 的 [`spawn`][builder-spawn]<!-- ignore --> 方法。

这段代码能够编译，并会存储我们作为 `ThreadPool::new` 实参指定数量的 `Worker` 实例。但我们<em>仍然</em>没有处理 `execute` 收到的闭包。接下来看看怎样做到这一点。

#### 通过通道向线程发送请求

接下来要解决的问题是，传给 `thread::spawn` 的闭包完全什么也不做。目前，我们在 `execute` 方法中取得了想执行的闭包。但是，在创建 `ThreadPool` 的过程中创建每个 `Worker` 时，需要向 `thread::spawn` 提供一个要运行的闭包。

我们希望刚刚创建的 `Worker` 结构体从 `ThreadPool` 中保存的队列获取要运行的代码，再把代码发送给自己的线程运行。

第 16 章学过的通道——一种让两个线程通信的简单方式——非常适合这个用例。我们用通道作为作业队列；`execute` 从 `ThreadPool` 向 `Worker` 实例发送作业，`Worker` 再把作业交给自己的线程。计划如下：

1. `ThreadPool` 创建通道并保留发送端。
2. 每个 `Worker` 保留接收端。
3. 创建一个新的 `Job` 结构体，用来保存想通过通道发送的闭包。
4. `execute` 方法通过发送端发送它想执行的作业。
5. `Worker` 在自己的线程中循环读取接收端，并执行收到的任何作业闭包。

首先在 `ThreadPool::new` 中创建通道，并让 `ThreadPool` 实例保存发送端，如示例 21-16 所示。`Job` 结构体目前不保存任何内容，但它将成为我们通过通道发送的项类型。

<Listing number="21-16" file-name="src/lib.rs" caption="修改 `ThreadPool`，让它存储传输 `Job` 实例的通道发送端">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

在 `ThreadPool::new` 中，我们创建新通道，让池保存发送端。这段代码可以成功编译。

让我们尝试在线程池创建每个 `Worker` 时，向其传入通道的接收端。我们知道想在 `Worker` 实例生成的线程中使用接收端，所以会在闭包中引用 `receiver` 形参。示例 21-17 中的代码还不能完全通过编译。

<Listing number="21-17" file-name="src/lib.rs" caption="把接收端传给每个 `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

我们进行了一些简单的小改动：把接收端传入 `Worker::new`，然后在闭包中使用它。

尝试检查这段代码时，会得到以下错误：

```console
{{#include ../../listings/ch21-web-server/listing-21-17/output.txt}}
```

代码正尝试把 `receiver` 传给多个 `Worker` 实例。正如第 16 章所说，这无法工作：Rust 提供的通道实现是多<em>生产者(producer)</em>、单<em>消费者(consumer)</em>。这意味着不能简单克隆通道的消费端来修复代码。我们也不想把一条消息多次发送给多个消费者；我们想要一份消息列表和多个 `Worker` 实例，让每条消息只处理一次。

此外，从通道队列中取出作业需要改变 `receiver`，因此各线程需要一种安全地共享和修改 `receiver` 的方式；否则可能出现竞态条件（第 16 章已介绍）。

回想第 16 章讨论的线程安全智能指针：要在多个线程之间共享所有权，并允许线程改变值，需要使用 `Arc<Mutex<T>>`。`Arc` 类型让多个 `Worker` 实例能够拥有接收端，`Mutex` 则确保一次只有一个 `Worker` 从接收端取得作业。示例 21-18 展示了所需更改。

<Listing number="21-18" file-name="src/lib.rs" caption="使用 `Arc` 和 `Mutex` 在各 `Worker` 实例之间共享接收端">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

在 `ThreadPool::new` 中，我们把接收端放入 `Arc` 和 `Mutex`。对于每个新 `Worker`，克隆 `Arc` 以增加引用计数，让 `Worker` 实例能够共享接收端的所有权。

完成这些更改后，代码可以编译！我们离目标越来越近了！

#### 实现 `execute` 方法

最后来实现 `ThreadPool` 上的 `execute` 方法。我们还会把 `Job` 从结构体改为 trait 对象的类型别名，用来保存 `execute` 所接收的闭包类型。正如第 20 章[“类型同义词与类型别名”][type-aliases]<!-- ignore -->一节所说，类型别名让我们能够缩短长类型，以便使用。请看示例 21-19。

<Listing number="21-19" file-name="src/lib.rs" caption="为保存各闭包的 `Box` 创建 `Job` 类型别名，然后通过通道发送作业">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

使用 `execute` 收到的闭包创建新 `Job` 实例后，我们通过通道的发送端发送该作业。这里对 `send` 调用 `unwrap`，以处理发送失败的情况。例如，如果停止所有线程的执行，接收端便会停止接收新消息，此时就可能发送失败。目前还无法停止线程执行：只要池还存在，线程就会继续执行。我们使用 `unwrap` 的原因是，自己知道失败情况不会发生，但编译器不知道。

不过还没完全结束！在 `Worker` 中，传给 `thread::spawn` 的闭包仍然只是<em>引用</em>通道的接收端。我们需要让闭包永远循环，不断向通道接收端请求作业，并在取得作业时运行它。按照示例 21-20 修改 `Worker::new`。

<Listing number="21-20" file-name="src/lib.rs" caption="在 `Worker` 实例的线程中接收并执行作业">

```rust,noplayground
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

这里先对 `receiver` 调用 `lock` 取得互斥锁，然后调用 `unwrap`，在出现任何错误时 panic。如果其他线程持有锁时发生 panic，而没有释放锁，互斥锁可能进入<em>中毒(poisoned)</em>状态，此时获取锁就可能失败。在这种情况下，调用 `unwrap` 让当前线程 panic 是正确的做法。你也可以把这个 `unwrap` 改为包含对自己有意义的错误消息的 `expect`。

如果成功取得互斥锁，就调用 `recv` 从通道接收 `Job`。最后一个 `unwrap` 同样用于越过这里的任何错误；如果持有发送端的线程已经关闭，就可能发生这种错误，类似于接收端关闭时 `send` 方法返回 `Err`。

对 `recv` 的调用会阻塞，所以如果还没有作业，当前线程会等待作业出现。`Mutex<T>` 确保一次只有一个 `Worker` 线程尝试请求作业。

现在，线程池已经进入可工作状态！执行 `cargo run`，并发出一些请求：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

成功了！现在有了一个异步执行连接的线程池。创建的线程永远不会超过四个，所以即使服务器收到大量请求，系统也不会过载。如果向 <em>/sleep</em> 发出请求，服务器能够让另一个线程运行其他请求，从而继续为它们提供服务。

> 注意：如果同时在多个浏览器窗口中打开 <em>/sleep</em>，它们可能会每隔五秒依次加载。出于缓存原因，有些 Web 浏览器会按顺序执行同一个请求的多个实例。这项限制并非由我们的 Web 服务器造成。

现在正适合停下来思考一下：如果使用 future 而不是闭包来表示要完成的工作，示例 21-18、21-19 和 21-20 中的代码会有什么不同？哪些类型会改变？方法签名会有何不同——如果有的话？代码的哪些部分会保持不变？

在第 17 章和第 19 章学习过 `while let` 循环后，你可能想知道为什么不把 `Worker` 线程代码写成示例 21-21 所示的样子。

<Listing number="21-21" file-name="src/lib.rs" caption="使用 `while let` 的另一种 `Worker::new` 实现">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

这段代码能够编译和运行，却不会产生所需的线程行为：一个缓慢请求仍会导致其他请求等待处理。原因有些微妙：`Mutex` 结构体没有公开的 `unlock` 方法，因为锁的所有权取决于 `lock` 方法所返回的 `LockResult<MutexGuard<T>>` 中 `MutexGuard<T>` 的生命周期。这样一来，借用检查器就能在编译时强制执行一项规则：除非持有锁，否则不能访问受 `Mutex` 保护的资源。不过，如果没有留意 `MutexGuard<T>` 的生命周期，这种实现也可能导致锁的持有时间超出预期。

示例 21-20 中使用 `let job = receiver.lock().unwrap().recv().unwrap();` 的代码能够工作，是因为对于 `let`，等号右侧表达式中使用的任何临时值都会在 `let` 语句结束时立即丢弃。然而，`while let`（以及 `if let` 和 `match`）直到关联代码块结束才会丢弃临时值。在示例 21-21 中，调用 `job()` 的整个期间都会持有锁，这意味着其他 `Worker` 实例无法接收作业。

[type-aliases]: ch20-03-advanced-types.html#type-synonyms-and-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[moving-out-of-closures]: ch13-01-closures.html#moving-captured-values-out-of-closures
[builder]: https://doc.rust-lang.org/std/thread/struct.Builder.html
[builder-spawn]: https://doc.rust-lang.org/std/thread/struct.Builder.html#method.spawn
