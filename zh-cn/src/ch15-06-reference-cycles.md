## 引用循环可能泄漏内存

Rust 的内存安全保证让意外创建永不被清理的内存（称为<em>内存泄漏(memory leak)</em>）变得很困难，但并非不可能。完全防止内存泄漏并不在 Rust 的保证范围内，这意味着在 Rust 中，内存泄漏仍然符合内存安全。通过 `Rc<T>` 和 `RefCell<T>` 可以看到 Rust 确实允许内存泄漏：可以创建其中各项循环引用彼此的引用。这样会产生内存泄漏，因为循环中每一项的引用计数永远不会降到 0，这些值也就永远不会被丢弃。

### 创建引用循环

下面从示例 15-25 中 `List` 枚举和 `tail` 方法的定义开始，看看<em>引用循环(reference cycle)</em>可能如何产生，以及如何防止它。

<Listing number="15-25" file-name="src/main.rs" caption="保存 `RefCell<T>` 的 cons 列表定义，使我们能够修改 `Cons` 变体所引用的对象">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-25/src/main.rs:here}}
```

</Listing>

这里使用的是示例 15-5 中 `List` 定义的另一种变体。现在，`Cons` 变体中的第二个元素是 `RefCell<Rc<List>>`，这意味着我们不再像示例 15-24 那样修改 `i32` 值，而是要修改 `Cons` 变体所指向的 `List` 值。我们还添加了一个 `tail` 方法，以便在拥有 `Cons` 变体时访问其中的第二项。

示例 15-26 添加了一个使用示例 15-25 中定义的 `main` 函数。这段代码在 `a` 中创建一个列表，并在 `b` 中创建一个指向 `a` 中列表的列表。然后，它修改 `a` 中的列表，使其指向 `b`，由此产生引用循环。整个过程中使用了多条 `println!` 语句，显示不同阶段的引用计数。

<Listing number="15-26" file-name="src/main.rs" caption="创建两个相互指向的 `List` 值所构成的引用循环">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

我们在变量 `a` 中创建一个保存 `List` 值的 `Rc<List>` 实例，其初始列表为 `5, Nil`。然后在变量 `b` 中创建另一个保存 `List` 值的 `Rc<List>` 实例，它包含值 `10`，并指向 `a` 中的列表。

接着修改 `a`，使其指向 `b` 而不是 `Nil`，由此形成循环。具体做法是使用 `tail` 方法取得对 `a` 中 `RefCell<Rc<List>>` 的引用，并将它放入变量 `link`。然后，对这个 `RefCell<Rc<List>>` 使用 `borrow_mut` 方法，将其中的值从保存 `Nil` 的 `Rc<List>` 改为 `b` 中的 `Rc<List>`。

运行这段代码时，暂时保持最后一条 `println!` 被注释掉，将得到以下输出：

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

修改 `a` 中的列表使其指向 `b` 后，`a` 和 `b` 中 `Rc<List>` 实例的引用计数都为 2。在 `main` 末尾，Rust 丢弃变量 `b`，使 `b` 中 `Rc<List>` 实例的引用计数从 2 降到 1。此时，这个 `Rc<List>` 在堆上的内存不会被丢弃，因为它的引用计数是 1 而不是 0。随后 Rust 丢弃 `a`，同样使 `a` 中 `Rc<List>` 实例的引用计数从 2 降到 1。这个实例的内存也无法被丢弃，因为另一个 `Rc<List>` 实例仍然引用它。分配给该列表的内存将永远得不到回收。为了直观展示这个引用循环，我们绘制了图 15-4。

<img alt="一个标签为 a 的矩形指向一个包含整数 5 的矩形。一个标签为 b 的矩形指向一个包含整数 10 的矩形。包含 5 的矩形指向包含 10 的矩形，包含 10 的矩形又指回包含 5 的矩形，从而形成循环。" src="img/trpl15-04.svg" class="center" />

<span class="caption">图 15-4：列表 `a` 和 `b` 相互指向形成的引用循环</span>

如果取消最后一条 `println!` 的注释并运行程序，Rust 会尝试打印这个循环：`a` 指向 `b`，`b` 又指向 `a`，如此反复，直到栈溢出。

与真实程序相比，本例中创建引用循环的后果并不十分严重：引用循环刚一创建，程序就结束了。然而，如果一个更复杂的程序在循环中分配大量内存并长时间持有它，程序就会使用超过实际所需的内存，甚至可能耗尽系统的可用内存。

引用循环并不容易产生，但也并非不可能。如果有包含 `Rc<T>` 值的 `RefCell<T>` 值，或内部可变性类型与引用计数类型以类似方式嵌套组合，就必须确保不会创建循环；不能依赖 Rust 捕获它们。创建引用循环属于程序中的逻辑错误，应通过自动化测试、代码审查和其他软件开发实践尽量减少这类错误。

避免引用循环的另一种办法，是重新组织数据结构，让一部分引用表达所有权，另一部分引用不表达所有权。这样，可以形成由某些所有权关系和某些非所有权关系构成的循环，而只有所有权关系会影响值能否被丢弃。在示例 15-25 中，我们始终希望 `Cons` 变体拥有其列表，因此无法重新组织这个数据结构。下面通过由父节点和子节点构成的图，看看何时适合用非所有权关系来防止引用循环。

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### 使用 `Weak<T>` 防止引用循环

到目前为止，我们已经展示了调用 `Rc::clone` 会增加 `Rc<T>` 实例的 `strong_count`，并且只有当 `strong_count` 为 0 时，`Rc<T>` 实例才会被清理。还可以调用 `Rc::downgrade` 并向它传入对 `Rc<T>` 的引用，创建指向 `Rc<T>` 实例中值的弱引用。<em>强引用(strong reference)</em>用于共享 `Rc<T>` 实例的所有权。<em>弱引用(weak reference)</em>不表示所有权关系，其数量也不影响何时清理 `Rc<T>` 实例。弱引用不会造成引用循环，因为一旦相关值的强引用计数降到 0，任何包含弱引用的循环都会被打破。

调用 `Rc::downgrade` 会得到 `Weak<T>` 类型的智能指针。它不会把 `Rc<T>` 实例中的 `strong_count` 加 1，而是把 `weak_count` 加 1。与 `strong_count` 类似，`Rc<T>` 类型使用 `weak_count` 跟踪存在多少个 `Weak<T>` 引用。区别在于，清理 `Rc<T>` 实例并不要求 `weak_count` 为 0。

由于 `Weak<T>` 所引用的值可能已经被丢弃，要对 `Weak<T>` 指向的值执行任何操作，都必须先确认该值仍然存在。为此，可以调用 `Weak<T>` 实例的 `upgrade` 方法，它会返回 `Option<Rc<T>>`。如果 `Rc<T>` 值尚未被丢弃，结果为 `Some`；如果已经被丢弃，结果为 `None`。由于 `upgrade` 返回 `Option<Rc<T>>`，Rust 会确保 `Some` 和 `None` 两种情况都得到处理，也就不会出现无效指针。

作为示例，我们不再使用每一项只知道下一项的列表，而是创建一棵树，其中每一项既知道自己的子项，<em>也</em>知道自己的父项。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-tree-data-structure-a-node-with-child-nodes"></a>

#### 创建树形数据结构

首先构建一棵节点知道其子节点的树。我们将创建名为 `Node` 的结构体，它既保存自己的 `i32` 值，也保存对其子 `Node` 值的引用：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

我们希望 `Node` 拥有其子节点，同时还希望与变量共享这份所有权，以便直接访问树中的每个 `Node`。为此，将 `Vec<T>` 的项定义为 `Rc<Node>` 类型的值。我们还希望修改哪些节点是另一个节点的子节点，因此在 `children` 中用 `RefCell<T>` 包裹 `Vec<Rc<Node>>`。

接下来使用这个结构体定义，创建一个名为 `leaf`、值为 `3` 且没有子节点的 `Node` 实例，再创建一个名为 `branch`、值为 `5` 且把 `leaf` 作为其子节点之一的实例，如示例 15-27 所示。

<Listing number="15-27" file-name="src/main.rs" caption="创建一个没有子节点的 `leaf` 节点，以及一个把 `leaf` 作为子节点之一的 `branch` 节点">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

我们克隆 `leaf` 中的 `Rc<Node>` 并将其存入 `branch`，这意味着 `leaf` 中的 `Node` 现在有两个所有者：`leaf` 和 `branch`。可以通过 `branch.children` 从 `branch` 到达 `leaf`，但无法从 `leaf` 到达 `branch`。这是因为 `leaf` 没有对 `branch` 的引用，也不知道两者相关。我们希望 `leaf` 知道 `branch` 是它的父节点，接下来就实现这一点。

#### 添加从子节点到父节点的引用

为了让子节点知道其父节点，需要在 `Node` 结构体定义中添加 `parent` 字段。难点在于决定 `parent` 应采用什么类型。我们知道它不能包含 `Rc<T>`，因为那会形成引用循环：`leaf.parent` 指向 `branch`，而 `branch.children` 指向 `leaf`，导致它们的 `strong_count` 永远不会为 0。

换个角度思考这些关系：父节点应该拥有其子节点；如果父节点被丢弃，其子节点也应该被丢弃。然而，子节点不应该拥有其父节点；如果丢弃一个子节点，父节点仍应存在。这正是弱引用的用武之地！

因此，`parent` 的类型不使用 `Rc<T>`，而是使用 `Weak<T>`，具体来说是 `RefCell<Weak<Node>>`。现在 `Node` 结构体的定义如下：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

节点可以引用其父节点，但不拥有父节点。在示例 15-28 中，我们更新 `main` 以使用这个新定义，让 `leaf` 节点能够引用其父节点 `branch`。

<Listing number="15-28" file-name="src/main.rs" caption="`leaf` 节点通过弱引用指向其父节点 `branch`">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

除了 `parent` 字段之外，创建 `leaf` 节点与示例 15-27 相似：`leaf` 一开始没有父节点，因此创建一个新的空 `Weak<Node>` 引用实例。

此时，尝试使用 `upgrade` 方法取得对 `leaf` 父节点的引用，会得到 `None` 值。第一条 `println!` 语句的输出体现了这一点：

```text
leaf parent = None
```

创建 `branch` 节点时，由于 `branch` 也没有父节点，它的 `parent` 字段同样包含一个新的 `Weak<Node>` 引用。`leaf` 仍然是 `branch` 的子节点之一。得到 `branch` 中的 `Node` 实例后，可以修改 `leaf`，为它添加一个指向父节点的 `Weak<Node>` 引用。我们对 `leaf` 的 `parent` 字段中的 `RefCell<Weak<Node>>` 使用 `borrow_mut` 方法，然后使用 `Rc::downgrade` 函数，从 `branch` 中的 `Rc<Node>` 创建一个指向 `branch` 的 `Weak<Node>` 引用。

再次打印 `leaf` 的父节点时，这次会得到一个保存 `branch` 的 `Some` 变体：现在 `leaf` 可以访问其父节点了！打印 `leaf` 时，也避免了示例 15-26 中最终导致栈溢出的循环；`Weak<Node>` 引用会显示为 `(Weak)`：

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

没有出现无限输出，说明这段代码没有创建引用循环。也可以通过查看调用 `Rc::strong_count` 和 `Rc::weak_count` 得到的值来判断这一点。

#### 直观观察 `strong_count` 和 `weak_count` 的变化

下面创建一个新的内部作用域，并把 `branch` 的创建移入其中，观察 `Rc<Node>` 实例的 `strong_count` 和 `weak_count` 如何变化。这样便能看到 `branch` 创建后，又在离开作用域时被丢弃所发生的事情。示例 15-29 展示了这些修改。

<Listing number="15-29" file-name="src/main.rs" caption="在内部作用域中创建 `branch`，并检查强引用计数和弱引用计数">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

创建 `leaf` 后，其中的 `Rc<Node>` 强引用计数为 1，弱引用计数为 0。在内部作用域中创建 `branch` 并将它与 `leaf` 关联后，打印计数时，`branch` 中的 `Rc<Node>` 强引用计数为 1，弱引用计数为 1（来自 `leaf.parent` 通过 `Weak<Node>` 指向 `branch`）。打印 `leaf` 的计数时，会看到其强引用计数为 2，因为 `branch` 现在把 `leaf` 的 `Rc<Node>` 克隆存储在 `branch.children` 中，但弱引用计数仍为 0。

内部作用域结束时，`branch` 离开作用域，`Rc<Node>` 的强引用计数降到 0，因此其中的 `Node` 被丢弃。来自 `leaf.parent` 的弱引用计数 1 不会影响 `Node` 是否被丢弃，所以不会发生任何内存泄漏！

如果在作用域结束后尝试访问 `leaf` 的父节点，将再次得到 `None`。程序结束时，`leaf` 中的 `Rc<Node>` 强引用计数为 1，弱引用计数为 0，因为变量 `leaf` 再次成为指向该 `Rc<Node>` 的唯一引用。

管理计数和值丢弃的所有逻辑，都内置于 `Rc<T>`、`Weak<T>` 及其 `Drop` trait 实现中。在 `Node` 定义中指定从子节点到父节点的关系应使用 `Weak<T>` 引用，就能让父节点指向子节点、子节点也指向父节点，而不会产生引用循环和内存泄漏。

## 小结

本章介绍了如何使用智能指针，获得不同于 Rust 使用普通引用时默认提供的保证与权衡。`Box<T>` 类型大小已知，指向在堆上分配的数据。`Rc<T>` 类型记录对堆上数据的引用数量，使数据能够拥有多个所有者。具有内部可变性的 `RefCell<T>` 类型，适用于我们需要一个不可变类型、却又需要修改其内部值的情况；它还会在运行时而非编译期执行借用规则。

本章还讨论了 `Deref` 和 `Drop` trait，它们实现了智能指针的许多功能。我们探讨了可能导致内存泄漏的引用循环，以及如何使用 `Weak<T>` 防止它们。

如果本章激发了你的兴趣，希望实现自己的智能指针，可以查阅 [《Rustonomicon》][nomicon]了解更多实用信息。

下一章将讨论 Rust 中的并发。你甚至还会学到几种新的智能指针。

[nomicon]: https://doc.rust-lang.org/nomicon/
