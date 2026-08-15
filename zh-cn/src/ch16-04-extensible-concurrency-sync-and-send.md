<!-- Old headings. Do not remove or links may break. -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>
<a id="extensible-concurrency-with-the-send-and-sync-traits"></a>

## 使用 `Send` 和 `Sync` 实现可扩展并发

有趣的是，本章到目前为止讨论的几乎所有并发功能都是标准库的一部分，而不是语言本身的一部分。处理并发的选择并不限于语言或标准库；你可以编写自己的并发功能，也可以使用他人编写的功能。

不过，嵌入语言而非标准库中的关键并发概念，包括 `std::marker` 中的 `Send` 和 `Sync` trait。

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-transference-of-ownership-between-threads-with-send"></a>

### 在线程间转移所有权

`Send` <em>标记 trait(marker trait)</em> 表明，实现 `Send` 的类型的值可以在线程间转移所有权。几乎所有 Rust 类型都实现了 `Send`，但也有一些例外，包括 `Rc<T>`：它不能实现 `Send`，因为如果克隆一个 `Rc<T>` 值并尝试把克隆的所有权转移给另一个线程，两个线程可能同时更新引用计数。因此，`Rc<T>` 是为单线程场景实现的，在这种场景中不必承担线程安全所带来的性能开销。

因此，Rust 的类型系统和 trait 约束能够确保你绝不会意外地以不安全方式跨线程发送 `Rc<T>` 值。在示例 16-14 中尝试这样做时，我们得到了错误 `` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``。改用实现了 `Send` 的 `Arc<T>` 后，代码便能编译。

任何完全由 `Send` 类型组成的类型，也会被自动标记为 `Send`。除第 20 章将讨论的裸指针外，几乎所有基本类型都是 `Send`。

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-access-from-multiple-threads-with-sync"></a>

### 从多个线程访问

`Sync` 标记 trait 表明，从多个线程引用实现 `Sync` 的类型是安全的。换句话说，如果 `&T`（对 `T` 的不可变引用）实现了 `Send`，意味着该引用可以安全地发送到另一个线程，那么任何类型 `T` 都实现 `Sync`。与 `Send` 类似，所有基本类型都实现 `Sync`；完全由实现 `Sync` 的类型构成的类型，也会实现 `Sync`。

智能指针 `Rc<T>` 不实现 `Sync`，原因与它不实现 `Send` 相同。第 15 章讨论过的 `RefCell<T>` 类型及相关的 `Cell<T>` 类型家族也不实现 `Sync`。`RefCell<T>` 在运行时执行的借用检查实现并非线程安全。智能指针 `Mutex<T>` 实现了 `Sync`，可以用于与多个线程共享访问，正如[“共享访问 `Mutex<T>`”][shared-access]<!-- ignore -->一节所见。

### 手动实现 `Send` 和 `Sync` 是不安全的

由于完全由其他实现 `Send` 和 `Sync` trait 的类型构成的类型，也会自动实现 `Send` 和 `Sync`，所以不必手动实现这些 trait。作为标记 trait，它们甚至没有任何需要实现的方法，只用于执行与并发有关的不变量。

手动实现这些 trait 涉及编写不安全 Rust 代码。第 20 章将讨论如何使用不安全 Rust 代码；现在要知道的重要一点是，构建并非由 `Send` 和 `Sync` 部分组成的新并发类型，需要仔细考虑如何维持安全保证。[《Rustonomicon》][nomicon]提供了有关这些保证以及如何维持它们的更多信息。

## 小结

这并不是本书最后一次讨论并发：下一章重点介绍异步编程，第 21 章的项目还会把本章概念用于比这里的小型示例更贴近实际的场景。

如前所述，由于 Rust 处理并发的方式很少属于语言本身，许多并发解决方案都以 crate 实现。这些 crate 的演进速度比标准库更快，因此请务必在线搜索当前适合多线程场景的先进 crate。

Rust 标准库为消息传递提供了通道，还提供 `Mutex<T>` 和 `Arc<T>` 等能在并发环境中安全使用的智能指针类型。类型系统和借用检查器确保使用这些方案的代码不会产生数据竞争或无效引用。代码一旦通过编译，你就可以确信它能在多个线程上顺利运行，不会出现其他语言中常见的那类难以追踪的 bug。并发编程不再是一个需要害怕的概念：大胆前进，无畏地让程序并发运行吧！

[shared-access]: ch16-03-shared-state.html#shared-access-to-mutext
[nomicon]: https://doc.rust-lang.org/nomicon/
