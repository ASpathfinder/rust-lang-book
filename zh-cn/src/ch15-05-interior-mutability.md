## `RefCell<T>` 与内部可变性模式

<em>内部可变性(interior mutability)</em>是 Rust 中的一种设计模式，即使存在指向数据的不可变引用，也允许修改该数据；通常借用规则不允许这种操作。为了修改数据，这种模式会在数据结构内部使用 `unsafe` 代码，适度放宽 Rust 通常用于管理修改和借用的规则。不安全代码告诉编译器，我们会手动检查这些规则，而不依赖编译器替我们检查；第 20 章将讨论不安全代码。

只有在能够确保运行时遵守借用规则、而编译器无法保证这一点时，才能使用采用内部可变性模式的类型。相关的 `unsafe` 代码会被包装在安全 API 中，而外部类型仍然是不可变的。

下面通过遵循内部可变性模式的 `RefCell<T>` 类型来探索这一概念。

<!-- Old headings. Do not remove or links may break. -->

<a id="enforcing-borrowing-rules-at-runtime-with-refcellt"></a>

### 在运行时执行借用规则

与 `Rc<T>` 不同，`RefCell<T>` 类型表示对其所保存数据的单一所有权。那么，`RefCell<T>` 与 `Box<T>` 之类的类型有何区别？回顾一下第 4 章学过的借用规则：

- 在任意给定时刻，要么只能有一个可变引用，要么可以有任意数量的不可变引用（但不能两者同时存在）。
- 引用必须始终有效。

对于引用和 `Box<T>`，借用规则的不变量在编译期执行；对于 `RefCell<T>`，这些不变量则在<em>运行时</em>执行。使用引用时，违反规则会得到编译错误；使用 `RefCell<T>` 时，违反规则会使程序 panic 并退出。

在编译期检查借用规则的优点是，错误能在开发过程的更早阶段被发现；并且所有分析都已预先完成，不会影响运行时性能。基于这些原因，在绝大多数情况下，编译期检查借用规则是最佳选择，所以这也是 Rust 的默认做法。

改在运行时检查借用规则的优点是，某些内存安全的场景能够得到允许，而编译期检查本会拒绝它们。像 Rust 编译器这样的静态分析天然倾向于保守。代码的某些性质不可能仅通过分析代码检测出来：最著名的例子是停机问题；它超出了本书范围，但值得研究。

因为有些分析无法完成，当 Rust 编译器不能确定代码遵守所有权规则时，它可能会拒绝一个正确的程序；在这个意义上，它是保守的。如果 Rust 接受了错误的程序，用户将无法信任 Rust 所作的保证。反之，如果 Rust 拒绝了正确的程序，程序员只是会感到不便，并不会发生灾难性的后果。当你确信代码遵循借用规则，而编译器却无法理解并保证这一点时，`RefCell<T>` 类型就很有用。

与 `Rc<T>` 类似，`RefCell<T>` 也只适用于单线程场景；如果尝试在多线程环境中使用它，会得到编译错误。第 16 章将讨论如何在多线程程序中获得 `RefCell<T>` 的功能。

下面回顾选择 `Box<T>`、`Rc<T>` 或 `RefCell<T>` 的理由：

- `Rc<T>` 允许同一数据有多个所有者；`Box<T>` 和 `RefCell<T>` 只有一个所有者。
- `Box<T>` 允许在编译期检查不可变或可变借用；`Rc<T>` 只允许在编译期检查不可变借用；`RefCell<T>` 允许在运行时检查不可变或可变借用。
- 因为 `RefCell<T>` 允许在运行时检查可变借用，所以即使 `RefCell<T>` 本身不可变，也能修改其中的值。

修改不可变值内部的值，就是内部可变性模式。下面看看内部可变性在哪种情况下有用，并研究它何以可行。

<!-- Old headings. Do not remove or links may break. -->

<a id="interior-mutability-a-mutable-borrow-to-an-immutable-value"></a>

### 使用内部可变性

借用规则带来的一个结果是：当你有一个不可变值时，不能对它进行可变借用。例如，以下代码无法编译：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

尝试编译这段代码会得到以下错误：

```console
{{#include ../../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

然而，在某些情况下，让一个值能在自己的方法中修改自身，同时对其他代码表现为不可变会很有用。值的方法外部的代码将不能修改它。使用 `RefCell<T>` 是获得内部可变性能力的一种方式，但 `RefCell<T>` 并没有完全绕过借用规则：编译器中的借用检查器允许这种内部可变性，借用规则则改在运行时进行检查。如果违反规则，得到的将是 `panic!`，而不是编译错误。

下面通过一个实际例子，看看如何使用 `RefCell<T>` 修改不可变值，以及这样做为何有用。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-use-case-for-interior-mutability-mock-objects"></a>

#### 使用模拟对象进行测试

有时在测试期间，程序员会使用一种类型代替另一种类型，以观察某种特定行为并断言它得到了正确实现。这种占位类型称为<em>测试替身(test double)</em>。可以把它类比为电影拍摄中的特技替身：由另一个人代替演员完成特别困难的场景。运行测试时，测试替身会代替其他类型。<em>模拟对象(mock object)</em>是一类特定的测试替身，它会记录测试期间发生的事情，以便断言发生了正确的操作。

Rust 中的对象与其他语言中的对象并不完全相同，Rust 标准库也没有像某些其他语言那样内置模拟对象功能。不过，完全可以创建一个结构体来实现与模拟对象相同的目的。

下面是要测试的场景：我们将创建一个库，记录某个值相对于最大值的位置，并根据当前值距离最大值有多近来发送消息。例如，这个库可以用来跟踪用户获准发起的 API 调用次数配额。

我们的库只提供跟踪值与最大值接近程度的功能，并确定应当在何时发送什么消息。使用该库的应用程序需要提供发送消息的机制：应用程序可以直接向用户显示消息、发送电子邮件、发送短信，或执行其他操作。库不需要知道这些细节，只需要某个实现了我们提供的 `Messenger` trait 的类型。示例 15-20 展示了库代码。

<Listing number="15-20" file-name="src/lib.rs" caption="一个跟踪值与最大值接近程度，并在值达到特定水平时发出警告的库">

```rust,noplayground
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

这段代码有一个重要部分：`Messenger` trait 有一个名为 `send` 的方法，它接收对 `self` 的不可变引用和消息文本。这个 trait 是模拟对象必须实现的接口，让模拟对象能以与真实对象相同的方式使用。另一个重要部分是，我们希望测试 `LimitTracker` 上 `set_value` 方法的行为。可以改变传给 `value` 形参的内容，但 `set_value` 不返回任何可供断言的值。我们希望能够判断：如果用一个实现了 `Messenger` trait 的类型和一个特定的 `max` 值创建 `LimitTracker`，那么向 `value` 传入不同数字时，消息发送器会被要求发送恰当的消息。

我们需要一个模拟对象：调用 `send` 时，它不发送电子邮件或短信，只记录被要求发送的消息。可以创建模拟对象的新实例，创建一个使用该模拟对象的 `LimitTracker`，调用 `LimitTracker` 上的 `set_value` 方法，然后检查模拟对象是否包含预期的消息。示例 15-21 尝试实现这样的模拟对象，但借用检查器不允许这么做。

<Listing number="15-21" file-name="src/lib.rs" caption="尝试实现借用检查器不允许的 `MockMessenger`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

这段测试代码定义了一个 `MockMessenger` 结构体，其中的 `sent_messages` 字段包含一个由 `String` 值构成的 `Vec`，用于记录它被要求发送的消息。我们还定义了关联函数 `new`，方便创建消息列表初始为空的 `MockMessenger` 值。然后为 `MockMessenger` 实现 `Messenger` trait，以便把 `MockMessenger` 提供给 `LimitTracker`。在 `send` 方法的定义中，我们取得作为形参传入的消息，并将其存入 `MockMessenger` 的 `sent_messages` 列表。

在这个测试中，我们要测试当 `LimitTracker` 被要求把 `value` 设为超过 `max` 值 75% 的数时会发生什么。首先创建一个新的 `MockMessenger`，它一开始拥有空的消息列表。然后创建一个新的 `LimitTracker`，向它提供指向新 `MockMessenger` 的引用和 `100` 这一 `max` 值。我们用 `80` 调用 `LimitTracker` 的 `set_value` 方法；80 超过了 100 的 75%。然后断言 `MockMessenger` 所跟踪的消息列表现在应当包含一条消息。

不过，这个测试存在一个问题，如下所示：

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

我们不能修改 `MockMessenger` 来记录消息，因为 `send` 方法接收的是对 `self` 的不可变引用。也不能采纳错误文本中的建议，在 `impl` 方法和 trait 定义中都使用 `&mut self`。我们不希望只为测试而修改 `Messenger` trait，而需要找到一种办法，让测试代码在现有设计下正确工作。

这正是内部可变性能够发挥作用的场景！我们会把 `sent_messages` 存储在 `RefCell<T>` 中，这样 `send` 方法就能修改 `sent_messages`，保存已经见过的消息。示例 15-22 展示了具体做法。

<Listing number="15-22" file-name="src/lib.rs" caption="在外部值被视为不可变时，使用 `RefCell<T>` 修改内部值">

```rust,noplayground
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

现在，`sent_messages` 字段的类型是 `RefCell<Vec<String>>`，而不是 `Vec<String>`。在 `new` 函数中，我们用空向量创建一个新的 `RefCell<Vec<String>>` 实例。

在 `send` 方法的实现中，第一个形参仍然是对 `self` 的不可变借用，与 trait 定义相匹配。我们对 `self.sent_messages` 中的 `RefCell<Vec<String>>` 调用 `borrow_mut`，获得一个指向 `RefCell<Vec<String>>` 内部值（即该向量）的可变引用。随后就能对这个指向向量的可变引用调用 `push`，记录测试期间发送的消息。

最后还需要修改断言：为了查看内部向量中有多少项，我们对 `RefCell<Vec<String>>` 调用 `borrow`，获得指向向量的不可变引用。

现在已经了解如何使用 `RefCell<T>`，下面深入看看它的工作原理！

<!-- Old headings. Do not remove or links may break. -->

<a id="keeping-track-of-borrows-at-runtime-with-refcellt"></a>

#### 在运行时跟踪借用

创建不可变引用和可变引用时，分别使用 `&` 和 `&mut` 语法。对于 `RefCell<T>`，则使用属于其安全 API 的 `borrow` 和 `borrow_mut` 方法。`borrow` 方法返回智能指针类型 `Ref<T>`，`borrow_mut` 返回智能指针类型 `RefMut<T>`。这两个类型都实现了 `Deref`，所以可以像普通引用一样使用它们。

`RefCell<T>` 会跟踪当前有多少个 `Ref<T>` 和 `RefMut<T>` 智能指针处于活动状态。每次调用 `borrow`，`RefCell<T>` 都会增加当前活动的不可变借用计数。当 `Ref<T>` 值离开作用域时，不可变借用计数减 1。与编译期借用规则一样，`RefCell<T>` 允许我们在任意时刻拥有多个不可变借用或一个可变借用。

如果尝试违反这些规则，与使用引用时得到编译错误不同，`RefCell<T>` 的实现会在运行时 panic。示例 15-23 修改了示例 15-22 中 `send` 的实现。我们故意尝试在同一作用域中创建两个同时活动的可变借用，以说明 `RefCell<T>` 会在运行时阻止这种行为。

<Listing number="15-23" file-name="src/lib.rs" caption="在同一作用域中创建两个可变引用，以观察 `RefCell<T>` 如何 panic">

```rust,ignore,panics
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

我们为 `borrow_mut` 返回的 `RefMut<T>` 智能指针创建变量 `one_borrow`。然后以同样的方式在变量 `two_borrow` 中创建另一个可变借用。这样，同一作用域中就存在两个可变引用，这是不允许的。运行库测试时，示例 15-23 中的代码编译不会报错，但测试会失败：

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

请注意，代码 panic 时显示了消息 `already borrowed: BorrowMutError`。这就是 `RefCell<T>` 在运行时处理违反借用规则行为的方式。

像这里一样选择在运行时而不是编译期捕获借用错误，意味着你可能会在开发过程的更晚阶段才发现代码中的错误，甚至可能要等到代码部署到生产环境之后。此外，由于在运行时而非编译期跟踪借用，代码会承担少量运行时性能开销。不过，使用 `RefCell<T>` 可以编写一个能够修改自身、记录所见消息的模拟对象，即使使用它的上下文只允许不可变值。尽管存在这些权衡，仍然可以使用 `RefCell<T>` 获得比普通引用更多的功能。

<!-- Old headings. Do not remove or links may break. -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>
<a id="allowing-multiple-owners-of-mutable-data-with-rct-and-refcellt"></a>

### 让可变数据拥有多个所有者

`RefCell<T>` 的一种常见用法是与 `Rc<T>` 结合。回想一下，`Rc<T>` 允许某些数据有多个所有者，但只提供对这些数据的不可变访问。如果有一个保存 `RefCell<T>` 的 `Rc<T>`，就能得到一个既可以拥有多个所有者、<em>又</em>可以修改的值！

例如，回顾示例 15-18 中的 cons 列表，我们使用 `Rc<T>` 让多个列表能够共同拥有另一个列表。由于 `Rc<T>` 只保存不可变值，列表一旦创建，就无法更改其中的任何值。现在加入 `RefCell<T>`，利用它修改列表值的能力。示例 15-24 展示了如何通过在 `Cons` 定义中使用 `RefCell<T>`，修改所有列表中存储的值。

<Listing number="15-24" file-name="src/main.rs" caption="使用 `Rc<RefCell<i32>>` 创建可以修改的 `List`">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

我们创建一个 `Rc<RefCell<i32>>` 实例，并将它存入名为 `value` 的变量，以便稍后直接访问。接着在 `a` 中创建一个 `List`，其 `Cons` 变体保存 `value`。需要克隆 `value`，让 `a` 和 `value` 都拥有内部的 `5`，而不是把所有权从 `value` 转移给 `a`，或者让 `a` 借用 `value`。

我们把列表 `a` 包装在 `Rc<T>` 中，这样创建列表 `b` 和 `c` 时，它们就都能引用 `a`，与示例 15-18 中的做法相同。

创建 `a`、`b` 和 `c` 中的列表后，我们想给 `value` 中的值加 10。为此，对 `value` 调用 `borrow_mut`；它会利用第 5 章[“`->` 运算符去哪儿了？”][wheres-the---operator]<!-- ignore -->一节讨论的自动解引用功能，把 `Rc<T>` 解引用为内部的 `RefCell<T>` 值。`borrow_mut` 方法返回一个 `RefMut<T>` 智能指针，我们对它使用解引用运算符并修改内部值。

打印 `a`、`b` 和 `c` 时，可以看到它们都包含修改后的值 `15`，而不是 `5`：

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

这种技巧相当巧妙！通过使用 `RefCell<T>`，我们得到了一个表面上不可变的 `List` 值，但可以借助 `RefCell<T>` 提供内部可变性访问的方法，在需要时修改数据。运行时借用规则检查能保护我们免受数据竞争，有时值得用少许速度换取数据结构的这种灵活性。请注意，`RefCell<T>` 不适用于多线程代码！`Mutex<T>` 是 `RefCell<T>` 的线程安全版本，我们将在第 16 章讨论 `Mutex<T>`。

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
