## 使用 `Box<T>` 指向堆上的数据

最直接的智能指针是<em>箱子(box)</em>，其类型写作 `Box<T>`。箱子允许把数据存储在堆上，而不是栈上。留在栈上的是指向堆数据的指针。有关栈与堆区别的回顾，请参阅第 4 章。

除了把数据存储在堆上而不是栈上，箱子不会产生性能开销。不过，它们也没有太多额外能力。以下场景最常使用箱子：

- 拥有一个在编译时无法得知大小的类型，并希望在需要确切大小的上下文中使用该类型的值
- 拥有大量数据，并希望转移所有权，同时确保数据不会在转移时被复制
- 希望拥有一个值，而且只关心它是实现某个特定特征的类型，而不关心具体类型

我们会在[“使用箱子支持递归类型”](#enabling-recursive-types-with-boxes)中展示第一种场景。在第二种情况下，转移大量数据的所有权可能花费很长时间，因为数据会在栈上来回复制。要提高这种情况下的性能，可以把大量数据存入堆上的箱子。这样，只有少量指针数据在栈上复制，而其引用的数据留在堆上的同一位置。第三种情况称为<em>特征对象(trait object)</em>，第 18 章[“使用特征对象抽象共同行为”][trait-objects]专门讨论该主题。因此，这里学到的知识还会在那一节再次应用！

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-store-data-on-the-heap"></a>

### 把数据存储在堆上

在讨论 `Box<T>` 的堆存储用例之前，先介绍语法以及如何与存储在 `Box<T>` 中的值交互。

示例 15-1 展示了如何使用箱子把 `i32` 值存储在堆上。

<Listing number="15-1" file-name="src/main.rs" caption="使用箱子在堆上存储 `i32` 值">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

我们把变量 `b` 定义为一个 `Box` 值，它指向在堆上分配的值 `5`。该程序会打印 `b = 5`；在这种情况下，可以像数据位于栈上一样访问箱子中的数据。与任何拥有所有权的值一样，当箱子离开作用域时（这里是 `main` 结束时的 `b`），它会被释放。箱子本身（存储在栈上）和它所指向的数据（存储在堆上）都会释放。

把单个值放在堆上并不太有用，因此不会经常以这种方式单独使用箱子。在大多数情况下，把单个 `i32` 之类的值放在默认存储位置栈上更合适。接下来看看箱子如何让我们定义没有它就不允许定义的类型。

<a id="enabling-recursive-types-with-boxes"></a>

### 使用箱子支持递归类型

<em>递归类型(recursive type)</em>的值可以包含另一个相同类型的值。递归类型会带来一个问题，因为 Rust 需要在编译时知道类型占用多少空间。然而，递归类型值的嵌套理论上可能无限延续，所以 Rust 无法知道值需要多少空间。由于箱子的大小已知，可以在递归类型定义中插入箱子，以支持递归类型。

作为递归类型的例子，让我们探索 cons 列表。这是一种常见于函数式编程语言的数据类型。我们定义的 cons 列表类型除了递归以外十分直接，因此这个例子中的概念适用于以后遇到的更复杂递归类型场景。

<!-- Old headings. Do not remove or links may break. -->

<a id="more-information-about-the-cons-list"></a>

#### 理解 cons 列表

<em>cons 列表(cons list)</em>是一种源自 Lisp 编程语言及其方言的数据结构，由嵌套的数对组成，是 Lisp 版本的链表。其名称来自 Lisp 中的 `cons` 函数（<em>构造函数(construct function)</em>的缩写），该函数从两个实参构造一个新数对。对由一个值和另一个数对组成的数对调用 `cons`，就可以构造由递归数对组成的 cons 列表。

例如，下面是包含列表 `1, 2, 3` 的 cons 列表伪代码表示，其中每个数对都位于圆括号内：

```text
(1, (2, (3, Nil)))
```

cons 列表中的每一项包含两个元素：当前项的值和下一项。列表最后一项只包含名为 `Nil` 的值，没有下一项。cons 列表通过递归调用 `cons` 函数产生。表示递归基本情况的规范名称是 `Nil`。请注意，它与第 6 章讨论的“null”或“nil”概念不同；后者表示无效或缺失值。

cons 列表不是 Rust 中常用的数据结构。大多数时候，如果在 Rust 中有一列条目，使用 `Vec<T>` 是更好的选择。其他更复杂的递归数据类型<em>确实</em>适用于各种场景；但本章从 cons 列表开始，可以在较少干扰下探索箱子如何让我们定义递归数据类型。

示例 15-2 包含 cons 列表的枚举定义。请注意，这段代码还无法编译，因为 `List` 类型没有已知大小，接下来会展示这一点。

<Listing number="15-2" file-name="src/main.rs" caption="首次尝试定义表示 `i32` 值 cons 列表数据结构的枚举">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> 注意：出于示例目的，我们实现的 cons 列表只存放 `i32` 值。也可以像第 10 章讨论的那样使用泛型实现它，定义能够存储任意类型值的 cons 列表类型。

使用 `List` 类型存储列表 `1, 2, 3` 的代码如示例 15-3 所示。

<Listing number="15-3" file-name="src/main.rs" caption="使用 `List` 枚举存储列表 `1, 2, 3`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

第一个 `Cons` 值存放 `1` 和另一个 `List` 值。这个 `List` 值又是另一个 `Cons` 值，存放 `2` 和另一个 `List` 值。这个 `List` 值还是一个 `Cons` 值，存放 `3` 和一个 `List` 值；后者最终是表示列表结束的非递归变体 `Nil`。

尝试编译示例 15-3 中的代码时，会得到示例 15-4 所示的错误。

<Listing number="15-4" caption="尝试定义递归枚举时得到的错误">

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

错误表明该类型“具有无限大小”。原因是我们定义的 `List` 有一个递归变体：它直接存放自身类型的另一个值。因此，Rust 无法确定存储 `List` 值需要多少空间。让我们拆解出现该错误的原因。首先看看 Rust 如何判断存储非递归类型的值需要多少空间。

#### 计算非递归类型的大小

回想第 6 章讨论枚举定义时，在示例 6-2 中定义的 `Message` 枚举：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

为了判断为 `Message` 值分配多少空间，Rust 会逐一检查各个变体，找出需要空间最多的变体。Rust 发现 `Message::Quit` 不需要任何空间，`Message::Move` 需要足够存储两个 `i32` 值的空间，依此类推。由于只会使用一个变体，`Message` 值最多需要的空间就是存储其最大变体所需的空间。

再对比 Rust 尝试判断示例 15-2 中 `List` 枚举这类递归类型需要多少空间时发生的情况。编译器首先查看 `Cons` 变体，它存放一个 `i32` 类型值和一个 `List` 类型值。因此，`Cons` 需要的空间等于一个 `i32` 的大小加上一个 `List` 的大小。为了确定 `List` 类型需要多少内存，编译器会查看变体，仍从 `Cons` 开始。`Cons` 变体存放一个 `i32` 类型值和一个 `List` 类型值；这个过程会无限持续，如图 15-1 所示。

<img alt="无限的 Cons 列表：标为 Cons 的矩形分成两个较小矩形。第一个较小矩形标为 i32，第二个标为 Cons，并包含外部 Cons 矩形的较小版本。Cons 矩形不断包含越来越小的自身版本，直到最小矩形中显示无穷符号，表明这种重复会永远继续。" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">图 15-1：由无限多个 `Cons` 变体组成的无限 `List`</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-get-a-recursive-type-with-a-known-size"></a>

#### 获得大小已知的递归类型

由于 Rust 无法判断应该为递归定义的类型分配多少空间，编译器会给出错误和以下有用建议：

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

在这个建议中，<em>间接寻址(indirection)</em>表示不直接存储值，而是通过存储指向值的指针，修改数据结构以间接存储值。

因为 `Box<T>` 是指针，Rust 始终知道 `Box<T>` 需要多少空间：指针大小不会随其指向的数据量而改变。这意味着可以在 `Cons` 变体中放入 `Box<T>`，而不是直接放入另一个 `List` 值。`Box<T>` 会指向下一个位于堆上的 `List` 值，而不是让它位于 `Cons` 变体内部。从概念上看，我们仍然拥有一个由列表包含其他列表而创建的列表，但该实现现在更像是把各项彼此相邻放置，而不是彼此嵌套。

可以把示例 15-2 中 `List` 枚举的定义和示例 15-3 中 `List` 的用法修改为示例 15-5 中的代码，它可以编译。

<Listing number="15-5" file-name="src/main.rs" caption="使用 `Box<T>` 使大小已知的 `List` 定义">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

`Cons` 变体需要一个 `i32` 的大小，加上存储箱子指针数据所需的空间。`Nil` 变体不存储任何值，所以在栈上所需空间少于 `Cons` 变体。现在我们知道，任何 `List` 值都会占用一个 `i32` 的大小加上箱子指针数据的大小。通过使用箱子，我们打破了无限递归链，所以编译器能够确定存储 `List` 值所需的大小。图 15-2 展示了现在 `Cons` 变体的样子。

<img alt="标为 Cons 的矩形分成两个较小矩形。第一个较小矩形标为 i32，第二个标为 Box，并包含一个标为 usize 的内部矩形，表示箱子指针的有限大小。" src="img/trpl15-02.svg" class="center" />

<span class="caption">图 15-2：大小不是无限的 `List`，因为 `Cons` 存放 `Box`</span>

箱子只提供间接寻址和堆分配；它们没有其他特殊能力，而我们会在其他智能指针类型中看到这类能力。它们也不会产生这些特殊能力造成的性能开销，因此在 cons 列表这类只需要间接寻址的情况下很有用。第 18 章会介绍箱子的更多用例。

`Box<T>` 类型是智能指针，因为它实现 `Deref` 特征，使 `Box<T>` 值可以像引用一样处理。当 `Box<T>` 值离开作用域时，由于 `Drop` 特征实现，箱子指向的堆数据也会被清理。对于本章剩余部分讨论的其他智能指针类型，这两个特征会更加重要。让我们更详细地探索它们。

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
