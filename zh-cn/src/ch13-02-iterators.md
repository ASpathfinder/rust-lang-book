## 使用迭代器处理一系列项

<em>迭代器模式(iterator pattern)</em>让你可以依次对一系列项执行某项任务。迭代器负责遍历每一项并判断序列何时结束的逻辑。使用迭代器时，无需自行重新实现这些逻辑。

在 Rust 中，迭代器是<em>惰性的(lazy)</em>，这意味着在调用消耗迭代器的方法之前，它们不会产生任何效果。例如，示例 13-10 中的代码通过调用 `Vec<T>` 上定义的 `iter` 方法，创建遍历向量 `v1` 中各项的迭代器。这段代码本身并没有做任何有用的事情。

<Listing number="13-10" file-name="src/main.rs" caption="创建迭代器">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

迭代器存储在变量 `v1_iter` 中。创建迭代器后，可以以多种方式使用它。在示例 3-5 中，我们使用 `for` 循环遍历数组，对每一项执行一些代码。在底层，这会隐式创建并消耗迭代器，但我们直到现在才详细讨论具体过程。

示例 13-11 将迭代器的创建与 `for` 循环中迭代器的使用分开。使用 `v1_iter` 中的迭代器调用 `for` 循环时，迭代器中的每个元素会用于循环的一次迭代，从而打印每个值。

<Listing number="13-11" file-name="src/main.rs" caption="在 `for` 循环中使用迭代器">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

在标准库未提供迭代器的语言中，你可能会从索引 0 开始创建变量，使用该变量索引向量以取得值，再在循环中递增变量值，直到它达到向量中的总项数，以此编写相同功能。

迭代器替你处理所有这些逻辑，减少了容易写错的重复代码。迭代器还提供更大灵活性，让相同逻辑可以用于多种不同序列，而不只是向量这类可按索引访问的数据结构。让我们考察迭代器如何做到这一点。

<a id="the-iterator-trait-and-the-next-method"></a>

### `Iterator` 特征与 `next` 方法

所有迭代器都实现标准库中定义的 `Iterator` 特征。该特征的定义如下：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

请注意，该定义使用了一些新语法：`type Item` 和 `Self::Item`，它们为这个特征定义<em>关联类型(associated type)</em>。第 20 章会深入讨论关联类型。现在只需知道，这段代码表示实现 `Iterator` 特征时还必须定义 `Item` 类型，而且该类型用于 `next` 方法的返回类型。换句话说，`Item` 类型就是迭代器返回的类型。

`Iterator` 特征只要求实现者定义一个方法：`next`。它每次返回迭代器的一项，用 `Some` 包裹；迭代结束时返回 `None`。

可以直接对迭代器调用 `next` 方法；示例 13-12 展示了对向量创建的迭代器重复调用 `next` 时会返回哪些值。

<Listing number="13-12" file-name="src/lib.rs" caption="对迭代器调用 `next` 方法">

```rust,noplayground
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

请注意，需要把 `v1_iter` 设为可变：对迭代器调用 `next` 方法会改变迭代器用于跟踪其在序列中位置的内部状态。换句话说，这段代码会<em>消耗(consumes)</em>迭代器。每次调用 `next` 都会用掉迭代器中的一项。使用 `for` 循环时无需把 `v1_iter` 设为可变，是因为循环取得了 `v1_iter` 的所有权，并在幕后将其设为可变。

还请注意，调用 `next` 得到的值是对向量中值的不可变引用。`iter` 方法产生遍历不可变引用的迭代器。如果希望创建取得 `v1` 所有权并返回拥有所有权的值的迭代器，可以调用 `into_iter` 而不是 `iter`。类似地，如果想遍历可变引用，可以调用 `iter_mut` 而不是 `iter`。

### 消耗迭代器的方法

`Iterator` 特征有许多由标准库提供默认实现的方法；可以查阅标准库中 `Iterator` 特征的 API 文档了解它们。其中一些方法会在定义中调用 `next`，这也是实现 `Iterator` 特征时必须实现 `next` 方法的原因。

调用 `next` 的方法称为<em>消耗适配器(consuming adapter)</em>，因为调用它们会用掉迭代器。`sum` 方法就是一个例子：它取得迭代器的所有权，通过反复调用 `next` 遍历各项，从而消耗迭代器。遍历期间，它把每一项加入累计总数，迭代完成后返回总数。示例 13-13 中的测试展示了 `sum` 方法的一种用法。

<Listing number="13-13" file-name="src/lib.rs" caption="调用 `sum` 方法取得迭代器中所有项的总和">

```rust,noplayground
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

调用 `sum` 之后，不允许再使用 `v1_iter`，因为 `sum` 取得了调用它的迭代器的所有权。

### 产生其他迭代器的方法

<em>迭代器适配器(iterator adapter)</em>是在 `Iterator` 特征上定义、但不消耗迭代器的方法。相反，它们会改变原始迭代器的某个方面，产生不同的迭代器。

示例 13-14 展示了调用迭代器适配器方法 `map` 的例子。该方法接收一个闭包，在遍历期间对每一项调用闭包。`map` 方法返回一个产生修改后各项的新迭代器。这里的闭包创建一个新迭代器，其中来自向量的每一项都会加 1。

<Listing number="13-14" file-name="src/main.rs" caption="调用迭代器适配器 `map` 创建新迭代器">

```rust,not_desired_behavior
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

不过，这段代码会产生警告：

```console
{{#include ../../listings/ch13-functional-features/listing-13-14/output.txt}}
```

示例 13-14 中的代码没有做任何事情，指定的闭包从未被调用。警告提醒了原因：迭代器适配器是惰性的，我们需要在这里消耗迭代器。

为了修复警告并消耗迭代器，我们会使用示例 12-1 中与 `env::args` 一起使用过的 `collect` 方法。该方法会消耗迭代器，并把结果值收集到某种集合数据类型中。

示例 13-15 把调用 `map` 所返回迭代器的遍历结果收集到向量中。该向量最终会包含原向量中的每一项，各自加 1。

<Listing number="13-15" file-name="src/main.rs" caption="调用 `map` 方法创建新迭代器，再调用 `collect` 方法消耗新迭代器并创建向量">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

由于 `map` 接收闭包，可以指定想要对每一项执行的任意操作。这很好地说明了闭包如何让你定制某种行为，同时复用 `Iterator` 特征提供的迭代行为。

可以链式调用多个迭代器适配器，以可读方式执行复杂操作。但由于所有迭代器都是惰性的，必须调用某个消耗适配器方法，才能从迭代器适配器调用中获得结果。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-closures-that-capture-their-environment"></a>

### 捕获环境的闭包

许多迭代器适配器接收闭包作为实参；通常，我们指定为迭代器适配器实参的闭包会捕获其环境。

这个例子会使用接收闭包的 `filter` 方法。闭包从迭代器取得一项并返回 `bool`。如果闭包返回 `true`，该值会包含在 `filter` 产生的迭代器中；如果返回 `false`，则不会包含。

在示例 13-16 中，我们将 `filter` 与从环境捕获 `shoe_size` 变量的闭包一起使用，遍历 `Shoe` 结构体实例集合。它只返回指定尺码的鞋。

<Listing number="13-16" file-name="src/lib.rs" caption="将 `filter` 方法与捕获 `shoe_size` 的闭包一起使用">

```rust,noplayground
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

`shoes_in_size` 函数以鞋的向量和鞋码为形参，取得向量的所有权，并返回只包含指定尺码鞋的向量。

在 `shoes_in_size` 函数体中，我们调用 `into_iter` 创建取得向量所有权的迭代器。然后调用 `filter`，把该迭代器适配为一个只包含闭包返回 `true` 的元素的新迭代器。

闭包从环境捕获 `shoe_size` 形参，将其值与每双鞋的尺码比较，只保留指定尺码的鞋。最后，调用 `collect` 把经过适配的迭代器返回的值收集到由函数返回的向量中。

测试表明，调用 `shoes_in_size` 时，只会返回尺码与指定值相同的鞋。
