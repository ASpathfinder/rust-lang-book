## 共享状态并发

消息传递是处理并发的好方法，但不是唯一方法。另一种方法是让多个线程访问同一份共享数据。再次想想 Go 语言文档那句口号中的一部分：“不要通过共享内存来通信。”

通过共享内存来通信会是什么样子？另外，为什么消息传递的拥护者会告诫人们不要共享内存？

从某种意义上说，任何编程语言中的通道都类似于单一所有权，因为一旦沿通道转移了一个值，就不应再使用该值。<em>共享内存并发(shared-memory concurrency)</em>则类似于多重所有权：多个线程可以同时访问同一内存位置。正如第 15 章所见，智能指针使多重所有权成为可能，而多重所有权会增加复杂性，因为需要管理不同的所有者。Rust 的类型系统和所有权规则极大地帮助我们正确完成这种管理。下面以互斥锁为例；它是共享内存中较常见的并发原语之一。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time"></a>

### 使用互斥锁控制访问

<em>互斥锁(mutex)</em>是<em>互斥(mutual exclusion)</em>的缩写，它在任意给定时刻只允许一个线程访问某些数据。要访问互斥锁中的数据，线程必须先请求取得互斥锁的锁定，以表明自己希望访问数据。<em>锁(lock)</em>是互斥锁中的一种数据结构，用于记录当前是谁在独占访问数据。因此，我们说互斥锁通过锁定系统<em>守护(guarding)</em>它所保存的数据。

互斥锁以难用著称，因为必须记住两条规则：

1. 使用数据前，必须尝试取得锁。
2. 使用完互斥锁所守护的数据后，必须解锁数据，让其他线程能够取得锁。

可以用会议上的小组讨论来类比互斥锁：设想现场只有一支麦克风。小组成员发言前，必须请求或示意自己想使用麦克风。拿到麦克风后，可以想讲多久就讲多久，讲完再把麦克风交给下一位请求发言的成员。如果有人用完后忘了交出麦克风，其他人就都无法发言。如果共享麦克风管理不当，小组讨论就无法按计划进行！

正确管理互斥锁可能极其棘手，这也正是许多人热衷于通道的原因。不过，得益于 Rust 的类型系统和所有权规则，你不会错误地加锁或解锁。

#### `Mutex<T>` 的 API

为了展示如何使用互斥锁，我们先在单线程环境中使用它，以简化问题，如示例 16-12 所示。

<Listing number="16-12" file-name="src/main.rs" caption="为简单起见，在单线程环境中探索 `Mutex<T>` 的 API">

```rust
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

与许多类型一样，我们使用关联函数 `new` 创建 `Mutex<T>`。要访问互斥锁内的数据，使用 `lock` 方法取得锁。这个调用会阻塞当前线程，使其无法执行任何工作，直到轮到它取得锁。

如果另一个持有锁的线程发生 panic，`lock` 调用就会失败。在这种情况下，谁都无法再取得锁，所以我们选择调用 `unwrap`，让当前线程在遇到该情况时 panic。

取得锁后，可以把返回值（本例中名为 `num`）当作指向内部数据的可变引用。类型系统会确保我们在使用 `m` 中的值前取得锁。`m` 的类型是 `Mutex<i32>`，而不是 `i32`，所以要使用 `i32` 值就<em>必须</em>调用 `lock`。我们不会忘记这一点，否则类型系统不会允许访问内部的 `i32`。

`lock` 调用返回名为 `MutexGuard` 的类型，它包裹在 `LockResult` 中，而我们使用 `unwrap` 调用处理了后者。`MutexGuard` 类型实现 `Deref`，指向内部数据；它还实现了 `Drop`，当 `MutexGuard` 离开作用域（即内部作用域结束）时自动释放锁。因此，我们不会冒着忘记释放锁、导致其他线程无法使用互斥锁的风险，因为锁会自动释放。

丢弃锁之后，可以打印互斥锁的值，看到我们成功把内部的 `i32` 改为了 `6`。

<!-- Old headings. Do not remove or links may break. -->

<a id="sharing-a-mutext-between-multiple-threads"></a>

<a id="shared-access-to-mutext"></a>

#### 共享访问 `Mutex<T>`

现在尝试用 `Mutex<T>` 在多个线程间共享一个值。我们会启动 10 个线程，让每个线程把计数器的值加 1，使计数器从 0 变为 10。示例 16-13 会产生编译错误，我们将借助这个错误进一步了解如何使用 `Mutex<T>`，以及 Rust 如何帮助我们正确使用它。

<Listing number="16-13" file-name="src/main.rs" caption="十个线程分别将 `Mutex<T>` 守护的计数器加 1">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

与示例 16-12 一样，我们创建变量 `counter`，在 `Mutex<T>` 中保存一个 `i32`。然后通过迭代一个数字范围创建 10 个线程。我们使用 `thread::spawn`，向所有线程提供同一个闭包：它把计数器移入线程，通过调用 `lock` 方法取得 `Mutex<T>` 上的锁，再把互斥锁中的值加 1。线程运行完闭包后，`num` 会离开作用域并释放锁，让另一个线程能够取得它。

在主线程中，我们收集所有 join 句柄。然后与示例 16-2 一样，对每个句柄调用 `join`，确保所有线程结束。此时主线程会取得锁并打印程序结果。

前面已经暗示这个示例无法编译。现在来看看原因！

```console
{{#include ../../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

错误消息指出，`counter` 值已在循环的上一次迭代中被移动。Rust 告诉我们，不能把锁 `counter` 的所有权移入多个线程。下面使用第 15 章讨论的多重所有权方法修复这个编译错误。

#### 多线程中的多重所有权

第 15 章中，我们使用智能指针 `Rc<T>` 创建引用计数值，让一个值拥有多个所有者。这里也这样做，看看会发生什么。示例 16-14 用 `Rc<T>` 包裹 `Mutex<T>`，并在把所有权移入线程前克隆 `Rc<T>`。

<Listing number="16-14" file-name="src/main.rs" caption="尝试使用 `Rc<T>` 让多个线程拥有 `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

再次编译，然后得到了……不同的错误！编译器教给了我们很多东西：

```console
{{#include ../../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

哇，这条错误消息非常冗长！需要关注的关键部分是：`` `Rc<Mutex<i32>>` cannot be sent between threads safely ``。编译器还告诉了我们原因：`` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``。下一节会讨论 `Send`：它是确保与线程一起使用的类型适合并发场景的 trait 之一。

遗憾的是，跨线程共享 `Rc<T>` 并不安全。`Rc<T>` 管理引用计数时，每次调用 `clone` 都会增加计数，每个克隆被丢弃时都会减少计数。但它不使用任何并发原语来确保修改计数的操作不会被另一个线程打断。这可能导致错误的计数，产生微妙的 bug，继而导致内存泄漏，或让值在我们用完之前就被丢弃。我们需要一个与 `Rc<T>` 完全相同、但能以线程安全方式修改引用计数的类型。

#### 使用 `Arc<T>` 进行原子引用计数

幸运的是，`Arc<T>` <em>就是</em>一种类似 `Rc<T>`、且能安全用于并发场景的类型。其中的 <em>a</em> 代表<em>原子(atomic)</em>，意味着它是一种<em>原子引用计数(atomically reference-counted)</em>类型。原子是另一种并发原语，本书不会在这里详细介绍；更多信息请参阅标准库中 [`std::sync::atomic` 的文档][atomic]<!-- ignore -->。现在只需知道，原子类型的工作方式类似于基本类型，但可以安全地在线程间共享。

你可能会疑惑，为什么所有基本类型都不是原子的，标准库类型为何不默认使用 `Arc<T>` 实现。原因在于，线程安全会带来性能开销，只有确实需要时才值得付出。如果只在单个线程内对值执行操作，代码无需执行原子类型所提供的保证，因此可以运行得更快。

回到我们的示例：`Arc<T>` 和 `Rc<T>` 具有相同的 API，所以只需修改 `use` 行、`new` 调用和 `clone` 调用就能修复程序。示例 16-15 中的代码终于可以编译和运行。

<Listing number="16-15" file-name="src/main.rs" caption="使用 `Arc<T>` 包裹 `Mutex<T>`，使所有权能够在多个线程间共享">

```rust
{{#rustdoc_include ../../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

这段代码会打印以下内容：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

成功了！我们从 0 数到了 10，这看起来可能不算了不起，但确实让我们学到了许多关于 `Mutex<T>` 和线程安全的知识。还可以利用这个程序的结构，执行比简单递增计数器更复杂的操作。采用这种策略，可以把一项计算分成多个独立部分，将它们分配到不同线程，再使用 `Mutex<T>` 让每个线程以自己的计算结果更新最终结果。

请注意，如果执行的是简单数值操作，标准库的 [`std::sync::atomic` 模块][atomic]<!-- ignore -->提供了比 `Mutex<T>` 更简单的类型。这些类型提供对基本类型安全、并发、原子的访问。本例选择将 `Mutex<T>` 与基本类型配合使用，是为了专注于 `Mutex<T>` 的工作方式。

<!-- Old headings. Do not remove or links may break. -->

<a id="similarities-between-refcelltrct-and-mutextarct"></a>

### 比较 `RefCell<T>`/`Rc<T>` 与 `Mutex<T>`/`Arc<T>`

你可能已经注意到，`counter` 是不可变的，但我们能够获得指向其内部值的可变引用；这意味着 `Mutex<T>` 与 `Cell` 家族一样，提供了内部可变性。正如第 15 章使用 `RefCell<T>` 来修改 `Rc<T>` 内的内容一样，这里使用 `Mutex<T>` 修改 `Arc<T>` 内的内容。

另一个需要注意的细节是，使用 `Mutex<T>` 时，Rust 无法保护你免受所有类型的逻辑错误。回想第 15 章，使用 `Rc<T>` 存在产生引用循环的风险：两个 `Rc<T>` 值相互引用，导致内存泄漏。同样，`Mutex<T>` 也存在产生<em>死锁(deadlock)</em>的风险。当一项操作需要锁定两个资源，而两个线程各自取得了其中一个锁时，就会发生死锁，导致它们永远相互等待。如果你对死锁感兴趣，可以尝试创建一个会死锁的 Rust 程序；然后研究任何语言中互斥锁的死锁缓解策略，并尝试在 Rust 中实现它们。标准库中 `Mutex<T>` 和 `MutexGuard` 的 API 文档提供了有用的信息。

本章最后将讨论 `Send` 和 `Sync` trait，以及如何将它们用于自定义类型。

[atomic]: https://doc.rust-lang.org/std/sync/atomic/index.html
