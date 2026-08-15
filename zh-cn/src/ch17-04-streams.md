<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## Stream：依次产生的 Future

回想本章前面[“消息传递”][17-02-messages]<!-- ignore -->一节中如何使用异步通道的接收器。异步 `recv` 方法会随时间推移产生一系列项。这是一种更通用模式的实例，称为 <em>stream</em>。许多概念都很自然地表示为 stream：队列中逐渐变得可用的项；完整数据集大到无法装入计算机内存时，从文件系统逐块增量提取的数据；或随时间通过网络到达的数据。因为 stream 是 future，所以可以把它们与任何其他类型的 future 结合，以有趣的方式组合。例如，可以批量处理事件以避免触发过多网络调用；为一系列耗时操作设置超时；或对用户界面事件进行限流，避免执行不必要的工作。

第 13 章[“`Iterator` Trait 与 `next` 方法”][iterator-trait]<!-- ignore -->一节介绍 Iterator trait 时，我们见过一系列项；但迭代器与异步通道接收器之间有两个区别。第一个区别是时间：迭代器是同步的，通道接收器是异步的。第二个区别是 API。直接使用 `Iterator` 时，调用其同步 `next` 方法；具体使用 `trpl::Receiver` stream 时，我们改为调用异步 `recv` 方法。除此以外，这些 API 感觉非常相似，而这种相似并非巧合。stream 就像异步形式的迭代。`trpl::Receiver` 专门等待接收消息，而通用 stream API 的范围要广得多：它像 `Iterator` 一样提供下一项，但以异步方式进行。

Rust 中迭代器与 stream 的相似性意味着，我们实际上可以从任何迭代器创建 stream。与迭代器一样，可以通过调用 stream 的 `next` 方法，再等待其输出，来处理 stream。示例 17-21 展示了这种做法，不过暂时无法编译。

<Listing number="17-21" caption="从迭代器创建 stream 并打印其中的值" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

我们从一个数字数组开始，把它转换为迭代器，然后调用 `map` 将所有值翻倍。接着使用 `trpl::stream_from_iter` 函数把迭代器转换为 stream。随后用 `while let` 循环遍历 stream 中陆续到达的项。

遗憾的是，尝试运行代码时，它无法编译，而是报告没有可用的 `next` 方法：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

正如这段输出所解释的，编译错误的原因是：必须把正确的 trait 引入作用域，才能使用 `next` 方法。根据目前的讨论，你可能会合理地认为这个 trait 是 `Stream`，但实际上是 `StreamExt`。`Ext` 是 <em>extension</em> 的缩写；在 Rust 社区中，用一个 trait 扩展另一个 trait 时常采用这种命名模式。

`Stream` trait 定义了一个低级接口，实际上结合了 `Iterator` 和 `Future` trait。`StreamExt` 在 `Stream` 之上提供一组更高级的 API，包括 `next` 方法以及与 `Iterator` trait 所提供方法相似的其他实用方法。`Stream` 和 `StreamExt` 尚未成为 Rust 标准库的一部分，但大多数生态系统 crate 都采用相似的定义。

修复编译错误的方法是添加 `use` 语句引入 `trpl::StreamExt`，如示例 17-22 所示。

<Listing number="17-22" caption="成功地以迭代器为基础使用 stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

把所有这些部分组合起来后，代码会按预期工作！不仅如此，现在 `StreamExt` 已在作用域中，还可以像使用迭代器那样使用其所有实用方法。

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
