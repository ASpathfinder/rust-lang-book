## 实现面向对象设计模式

<em>状态模式(state pattern)</em>是一种面向对象设计模式。其核心是定义一个值在内部可能具有的一组状态。状态由一组<em>状态对象(state object)</em>表示，而值的行为会根据其状态改变。我们将逐步完成一个博客文章结构体示例，它有一个保存状态的字段；这个状态是“草稿”“审核”或“已发布”集合中的一个状态对象。

状态对象共享功能：在 Rust 中，我们当然使用结构体和 trait，而不是对象和继承。每个状态对象负责自己的行为，并决定应当何时变为另一状态。保存状态对象的值不了解不同状态的行为，也不知道何时应在状态间转换。

使用状态模式的优点是，当程序的业务需求变化时，无需修改保存状态之值的代码，也无需修改使用该值的代码。只需更新某个状态对象内的代码以改变其规则，或添加更多状态对象。

首先，我们将以更传统的面向对象方式实现状态模式。然后，再采用一种在 Rust 中略显自然的方法。下面深入探索，使用状态模式逐步实现博客文章工作流。

最终功能如下：

1. 博客文章以空草稿开始。
1. 草稿完成后，请求审核文章。
1. 文章获批后得到发布。
1. 只有已发布的博客文章才返回可供打印的内容，以免未获批准的文章被意外发布。

尝试对文章进行的任何其他更改都不应产生效果。例如，如果在请求审核前尝试批准博客文章草稿，文章应当仍是未发布的草稿。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### 尝试传统面向对象风格

组织代码来解决同一问题的方式数不胜数，每种方式都有不同权衡。本节的实现更接近传统面向对象风格；这种风格可以用 Rust 编写，但没有利用 Rust 的一些优势。稍后将展示一种不同的解决方案，它仍然使用面向对象设计模式，但其组织方式对有面向对象经验的程序员而言可能不那么熟悉。我们会比较两种方案，体会以不同于其他语言的方式设计 Rust 代码所带来的权衡。

示例 18-11 以代码形式展示了这个工作流：这是我们将在名为 `blog` 的库 crate 中实现之 API 的使用示例。由于还没有实现 `blog` crate，它暂时无法编译。

<Listing number="18-11" file-name="src/main.rs" caption="演示希望 `blog` crate 具备的行为">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

我们希望允许用户使用 `Post::new` 创建新的博客文章草稿，并允许向文章添加文本。如果在文章获得批准前立即尝试获取其内容，就不应得到任何文本，因为文章仍是草稿。为演示目的，我们在代码中添加了 `assert_eq!`。一个很好的单元测试是断言博客文章草稿的 `content` 方法返回空字符串，但本例不会编写测试。

接下来，希望能够请求审核文章，并让 `content` 在等待审核期间返回空字符串。文章获得批准后，应当被发布，这意味着调用 `content` 时会返回文章文本。

请注意，我们从 crate 中交互的唯一类型是 `Post`。该类型将使用状态模式，并保存一个值；它是表示文章可能处于草稿、审核或已发布状态的三个状态对象之一。从一种状态变为另一种状态会在 `Post` 类型内部管理。状态会响应库用户对 `Post` 实例调用的方法而改变，但用户不必直接管理状态变化。用户也无法在状态方面犯错，例如未经审核就发布文章。

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### 定义 `Post` 并创建新实例

现在开始实现这个库！我们知道，需要一个保存内容的公开 `Post` 结构体，所以先定义该结构体和一个关联的公开 `new` 函数，用于创建 `Post` 实例，如示例 18-12 所示。还会创建私有的 `State` trait，定义 `Post` 的所有状态对象都必须具有的行为。

然后，`Post` 会在名为 `state` 的私有字段中保存一个位于 `Option<T>` 内的 `Box<dyn State>` 特征对象，用来保存状态对象。稍后会看到为什么需要 `Option<T>`。

<Listing number="18-12" file-name="src/lib.rs" caption="定义 `Post` 结构体、创建新 `Post` 实例的 `new` 函数、`State` trait 和 `Draft` 结构体">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

`State` trait 定义不同文章状态所共享的行为。状态对象包括 `Draft`、`PendingReview` 和 `Published`，它们都会实现 `State` trait。现在 trait 还没有任何方法；我们先只定义 `Draft` 状态，因为希望文章以该状态开始。

创建新的 `Post` 时，把其 `state` 字段设为保存 `Box` 的 `Some` 值。这个 `Box` 指向 `Draft` 结构体的新实例。这样，每当创建 `Post` 的新实例时，它都会以草稿开始。由于 `Post` 的 `state` 字段是私有的，不可能以任何其他状态创建 `Post`！在 `Post::new` 函数中，把 `content` 字段设为新的空 `String`。

#### 存储文章内容文本

示例 18-11 表明，我们希望能够调用名为 `add_text` 的方法并传入 `&str`，随后把它添加为博客文章的文本内容。我们把它实现为方法，而不是把 `content` 字段公开为 `pub`，这样以后可以实现一个方法来控制如何读取 `content` 字段的数据。`add_text` 方法非常直接，因此把示例 18-13 中的实现添加到 `impl Post` 块。

<Listing number="18-13" file-name="src/lib.rs" caption="实现 `add_text` 方法，向文章的 `content` 添加文本">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

`add_text` 方法接收对 `self` 的可变引用，因为我们要修改调用 `add_text` 的 `Post` 实例。然后，对 `content` 中的 `String` 调用 `push_str`，传入 `text` 实参，把它添加到保存的 `content`。这种行为不依赖文章所处状态，所以不属于状态模式。`add_text` 方法完全不与 `state` 字段交互，但它是我们希望支持的行为之一。

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### 确保文章草稿的内容为空

即使已经调用 `add_text` 向文章添加了一些内容，我们仍然希望 `content` 方法返回空字符串切片，因为文章仍处于草稿状态；示例 18-11 中第一个 `assert_eq!` 展示了这一点。现在先用能满足该要求的最简单方式实现 `content` 方法：始终返回空字符串切片。稍后实现改变文章状态、使其能够发布的功能后，再修改这一点。目前，文章只能处于草稿状态，所以文章内容应始终为空。示例 18-14 展示了这个占位实现。

<Listing number="18-14" file-name="src/lib.rs" caption="为 `Post` 上的 `content` 方法添加占位实现，它始终返回空字符串切片">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

添加这个 `content` 方法后，示例 18-11 中直到第一个 `assert_eq!` 为止的所有内容都会按预期工作。

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### 请求审核，从而改变文章状态

接下来，需要添加请求审核文章的功能，它应当把文章状态从 `Draft` 改为 `PendingReview`。示例 18-15 展示了这些代码。

<Listing number="18-15" file-name="src/lib.rs" caption="在 `Post` 和 `State` trait 上实现 `request_review` 方法">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

我们为 `Post` 提供一个名为 `request_review` 的公开方法，它接收对 `self` 的可变引用。然后，对 `Post` 的当前状态调用内部 `request_review` 方法；第二个 `request_review` 方法会消费当前状态并返回新状态。

我们把 `request_review` 方法添加到 `State` trait；现在，实现该 trait 的所有类型都必须实现 `request_review` 方法。请注意，该方法的第一个形参不是 `self`、`&self` 或 `&mut self`，而是 `self: Box<Self>`。这种语法意味着，只有对保存该类型的 `Box` 调用时，这个方法才有效。它会取得 `Box<Self>` 的所有权，使旧状态失效，从而让 `Post` 的状态值转变为新状态。

要消费旧状态，`request_review` 方法需要取得状态值的所有权。`Post` 的 `state` 字段中的 `Option` 正是在此发挥作用：我们调用 `take` 方法，从 `state` 字段中取出 `Some` 值，并在原处留下 `None`，因为 Rust 不允许结构体包含未填充字段。这样便能把 `state` 值移出 `Post`，而不是借用它。然后，再把文章的 `state` 值设为这次操作的结果。

为了取得 `state` 值的所有权，需要暂时把 `state` 设为 `None`，而不能用 `self.state = self.state.request_review();` 这样的代码直接设置。这会确保把旧 `state` 转换为新状态后，`Post` 无法再使用旧值。

`Draft` 上的 `request_review` 方法返回新 `PendingReview` 结构体的新箱装实例，表示文章正在等待审核的状态。`PendingReview` 结构体也实现 `request_review` 方法，但不执行任何转换，而是返回自身，因为对已经处于 `PendingReview` 状态的文章请求审核时，它应当保持 `PendingReview` 状态。

现在可以开始看到状态模式的优势：无论 `state` 具有何值，`Post` 上的 `request_review` 方法都完全相同。每个状态负责自己的规则。

我们让 `Post` 上的 `content` 方法保持不变，继续返回空字符串切片。现在 `Post` 除了可以处于 `Draft` 状态，也可以处于 `PendingReview` 状态；而在 `PendingReview` 状态下，我们希望行为相同。示例 18-11 现在可以一直工作到第二个 `assert_eq!` 调用！

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### 添加 `approve` 以改变 `content` 的行为

`approve` 方法与 `request_review` 方法相似：它会把 `state` 设为当前状态表示其被批准时应具有的值，如示例 18-16 所示。

<Listing number="18-16" file-name="src/lib.rs" caption="在 `Post` 和 `State` trait 上实现 `approve` 方法">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

我们把 `approve` 方法添加到 `State` trait，并添加一个实现 `State` 的新结构体，即 `Published` 状态。

与 `PendingReview` 上的 `request_review` 类似，如果对 `Draft` 调用 `approve` 方法，它不会产生效果，因为 `approve` 会返回 `self`。对 `PendingReview` 调用 `approve` 时，它会返回 `Published` 结构体的新箱装实例。`Published` 结构体实现 `State` trait；其 `request_review` 和 `approve` 方法都会返回自身，因为在这些情况下，文章应当保持 `Published` 状态。

现在需要更新 `Post` 上的 `content` 方法。我们希望 `content` 的返回值取决于 `Post` 的当前状态，所以让 `Post` 把工作委托给其 `state` 上定义的 `content` 方法，如示例 18-17 所示。

<Listing number="18-17" file-name="src/lib.rs" caption="更新 `Post` 上的 `content` 方法，把工作委托给 `State` 上的 `content` 方法">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

由于目标是把所有规则都保留在实现 `State` 的结构体中，我们对 `state` 中的值调用 `content` 方法，并把文章实例（即 `self`）作为实参传入。然后，返回对 `state` 值使用 `content` 方法所返回的值。

我们对 `Option` 调用 `as_ref` 方法，因为希望获得对 `Option` 内部值的引用，而不是该值的所有权。由于 `state` 是 `Option<Box<dyn State>>`，调用 `as_ref` 会返回 `Option<&Box<dyn State>>`。如果不调用 `as_ref`，就会报错，因为不能把 `state` 移出函数形参中借用的 `&self`。

然后调用 `unwrap` 方法。我们知道它永远不会 panic，因为 `Post` 上的方法能够确保这些方法执行完毕时，`state` 始终包含 `Some` 值。这属于第 9 章[“当你比编译器掌握更多信息时”][more-info-than-rustc]<!-- ignore -->一节讨论过的情况：我们知道 `None` 值绝不可能出现，尽管编译器无法理解这一点。

此时，对 `&Box<dyn State>` 调用 `content` 时，解引用强制转换会作用于 `&` 和 `Box`，最终在实现 `State` trait 的类型上调用 `content` 方法。这意味着需要把 `content` 添加到 `State` trait 定义；我们会在那里放置根据当前状态决定返回何种内容的逻辑，如示例 18-18 所示。

<Listing number="18-18" file-name="src/lib.rs" caption="把 `content` 方法添加到 `State` trait">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

我们为 `content` 方法添加一个返回空字符串切片的默认实现。这意味着无需在 `Draft` 和 `PendingReview` 结构体上实现 `content`。`Published` 结构体会重载 `content` 方法，返回 `post.content` 中的值。虽然这样很方便，但让 `State` 上的 `content` 方法决定 `Post` 的内容，模糊了 `State` 与 `Post` 之间的职责边界。

请注意，这个方法需要生命周期标注，正如第 10 章所讨论的。我们接收对 `post` 的引用作为实参，并返回对该 `post` 一部分的引用，因此返回引用的生命周期与 `post` 实参的生命周期相关。

至此全部完成，示例 18-11 中的一切现在都能工作！我们按照博客文章工作流的规则实现了状态模式。与这些规则有关的逻辑位于状态对象中，而不是分散在整个 `Post` 中。

> ### 为什么不使用枚举？
>
> 你可能一直在想，为什么不使用以各种可能文章状态为变体的枚举。这当然是一种可行方案；尝试实现它，并比较最终结果，看看更喜欢哪一种！使用枚举的一个缺点是，检查枚举值的每个位置都需要 `match` 表达式或类似结构来处理所有可能变体。这可能比特征对象方案更加重复。

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### 评估状态模式

我们已经展示，Rust 能够实现面向对象状态模式，从而封装文章在每种状态下应具有的不同行为。`Post` 上的方法不了解各种行为。由于代码的组织方式，只需查看一个位置，就能知道已发布文章可能有哪些不同的行为：`Published` 结构体上的 `State` trait 实现。

如果创建一个不使用状态模式的替代实现，可能会在 `Post` 上的方法中，甚至在检查文章状态并据此改变行为的 `main` 代码中使用 `match` 表达式。这样，要理解文章处于已发布状态所带来的全部影响，就必须查看多个位置。

使用状态模式时，`Post` 方法和使用 `Post` 的位置都不需要 `match` 表达式；添加新状态时，只需添加新结构体，并在一个位置为该结构体实现 trait 方法。

采用状态模式的实现很容易扩展，以添加更多功能。为了体会维护状态模式代码的简便性，可以尝试以下建议：

- 添加 `reject` 方法，把文章状态从 `PendingReview` 改回 `Draft`。
- 要求调用两次 `approve` 才能把状态改为 `Published`。
- 只允许用户在文章处于 `Draft` 状态时添加文本内容。提示：让状态对象负责决定内容可能发生什么变化，但不要让它负责修改 `Post`。

状态模式的一个缺点是，由于状态实现了状态间的转换，某些状态会相互耦合。如果在 `PendingReview` 和 `Published` 之间添加另一个状态，例如 `Scheduled`，就必须修改 `PendingReview` 中的代码，使其改为转换到 `Scheduled`。如果添加新状态时无需修改 `PendingReview`，工作量会更少，但那意味着要改用另一种设计模式。

另一个缺点是，我们重复了一些逻辑。为了消除部分重复，可能会尝试在 `State` trait 上为 `request_review` 和 `approve` 方法提供返回 `self` 的默认实现。然而，这行不通：把 `State` 用作特征对象时，trait 不知道具体的 `self` 究竟是什么，所以编译期无法得知返回类型。（这是前面提到的 dyn 兼容性规则之一。）

其他重复内容包括 `Post` 上 `request_review` 和 `approve` 方法的相似实现。两个方法都对 `Post` 的 `state` 字段使用 `Option::take`；如果 `state` 是 `Some`，就委托给包裹值对同名方法的实现，并把 `state` 字段的新值设为结果。如果 `Post` 上有许多遵循这种模式的方法，可以考虑定义宏消除重复（见第 20 章[“宏”][macros]<!-- ignore -->一节）。

严格按照面向对象语言所定义的方式实现状态模式，没有尽可能充分地利用 Rust 的优势。下面看看可以对 `blog` crate 进行哪些修改，把无效状态和无效转换变成编译期错误。

<a id="encoding-states-and-behavior-as-types"></a>

### 把状态与行为编码为类型

我们将展示如何重新思考状态模式，获得另一组权衡。不再完全封装状态和转换、让外部代码一无所知，而是把状态编码为不同类型。这样，尝试在只允许已发布文章的位置使用文章草稿时，Rust 的类型检查系统会发出编译错误，从而阻止这种行为。

思考示例 18-11 中 `main` 的第一部分：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

仍然允许使用 `Post::new` 创建处于草稿状态的新文章，并向文章内容添加文本。但不再为文章草稿提供返回空字符串的 `content` 方法，而是让文章草稿根本没有 `content` 方法。这样，如果尝试取得文章草稿的内容，就会得到编译错误，指出该方法不存在。因此，我们不可能在生产环境中意外显示文章草稿内容，因为这样的代码根本无法编译。示例 18-19 展示了 `Post` 结构体和 `DraftPost` 结构体的定义，以及分别在两者上定义的方法。

<Listing number="18-19" file-name="src/lib.rs" caption="具有 `content` 方法的 `Post`，以及没有 `content` 方法的 `DraftPost`">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

`Post` 和 `DraftPost` 结构体都有一个存储博客文章文本的私有 `content` 字段。结构体不再具有 `state` 字段，因为我们把状态编码转移到了结构体类型上。`Post` 结构体将表示已发布文章，并有一个返回 `content` 的 `content` 方法。

仍然有 `Post::new` 函数，但它不再返回 `Post` 实例，而是返回 `DraftPost` 实例。由于 `content` 是私有的，而且没有任何函数返回 `Post`，目前无法创建 `Post` 实例。

`DraftPost` 结构体有 `add_text` 方法，所以仍然可以像以前一样向 `content` 添加文本；但请注意，`DraftPost` 没有定义 `content` 方法！因此，现在程序能确保所有文章都以文章草稿开始，而且文章草稿的内容不能用于显示。任何绕过这些约束的尝试都会产生编译错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

那么，如何得到已发布文章？我们希望执行以下规则：文章草稿必须经过审核和批准才能发布。处于等待审核状态的文章仍然不应显示任何内容。为实现这些约束，我们添加另一个结构体 `PendingReviewPost`；在 `DraftPost` 上定义返回 `PendingReviewPost` 的 `request_review` 方法，并在 `PendingReviewPost` 上定义返回 `Post` 的 `approve` 方法，如示例 18-20 所示。

<Listing number="18-20" file-name="src/lib.rs" caption="通过对 `DraftPost` 调用 `request_review` 创建的 `PendingReviewPost`，以及把 `PendingReviewPost` 转换为已发布 `Post` 的 `approve` 方法">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

`request_review` 和 `approve` 方法取得 `self` 的所有权，因而会分别消费 `DraftPost` 和 `PendingReviewPost` 实例，并将它们转换为 `PendingReviewPost` 和已发布的 `Post`。这样，对 `DraftPost` 调用 `request_review` 后，不会留下仍然存在的 `DraftPost` 实例，其他情况以此类推。`PendingReviewPost` 结构体也没有定义 `content` 方法，因此与 `DraftPost` 一样，尝试读取其内容会产生编译错误。要得到定义了 `content` 方法的已发布 `Post` 实例，只能对 `PendingReviewPost` 调用 `approve` 方法；而要得到 `PendingReviewPost`，只能对 `DraftPost` 调用 `request_review` 方法。因此，我们现在已经把博客文章工作流编码进类型系统。

不过，还需要对 `main` 进行一些小修改。`request_review` 和 `approve` 方法返回新实例，而不是修改调用它们的结构体，因此需要添加更多 `let post =` 遮蔽赋值来保存返回的实例。我们也不能再断言草稿和等待审核文章的内容为空字符串，而且也不需要这些断言：尝试使用处于这些状态之文章内容的代码已经无法编译。更新后的 `main` 代码如示例 18-21 所示。

<Listing number="18-21" file-name="src/main.rs" caption="修改 `main`，使用博客文章工作流的新实现">

```rust,ignore
{{#rustdoc_include ../../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

为了重新赋值 `post` 而必须对 `main` 进行的修改，意味着这个实现不再完全遵循面向对象状态模式：状态之间的转换不再完全封装在 `Post` 实现内部。不过，我们得到的好处是：由于类型系统以及编译期进行的类型检查，无效状态现在已不可能存在！这能确保某些 bug（例如显示未发布文章的内容）在进入生产环境前就被发现。

尝试对示例 18-21 之后的 `blog` crate 完成本节开头建议的任务，看看你如何评价这一版本的代码设计。请注意，有些任务在这个设计中可能已经完成。

我们已经看到，尽管 Rust 能够实现面向对象设计模式，但 Rust 也可以使用其他模式，例如把状态编码进类型系统。这些模式具有不同权衡。即使你非常熟悉面向对象模式，重新思考问题以利用 Rust 的功能，也能带来在编译期防止某些 bug 等好处。由于所有权等面向对象语言所没有的功能，面向对象模式在 Rust 中并不总是最佳解决方案。

## 小结

无论读完本章后你是否认为 Rust 是面向对象语言，现在都已经知道，可以使用特征对象在 Rust 中获得某些面向对象功能。动态分派以少许运行时性能为代价，为代码提供了一些灵活性。可以利用这种灵活性实现有助于提高代码可维护性的面向对象模式。Rust 还拥有所有权等面向对象语言所没有的功能。面向对象模式并不总是发挥 Rust 优势的最佳方式，但它是一种可用选择。

接下来将介绍模式，这是 Rust 另一个能带来高度灵活性的功能。全书一直在简要使用模式，但尚未见识它的完整能力。开始吧！

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
