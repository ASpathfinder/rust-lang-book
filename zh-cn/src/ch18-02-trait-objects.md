<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

<a id="using-trait-objects-to-abstract-over-shared-behavior"></a>

## 使用特征对象抽象共同行为

第 8 章曾提到，向量的一项限制是只能存储单一类型的元素。在示例 8-9 中，我们定义 `SpreadsheetCell` 枚举作为变通办法，其中有分别保存整数、浮点数和文本的变体。这样，每个单元格中可以存储不同类型的数据，同时仍然使用一个向量表示一行单元格。当可互换的项属于一组固定类型，而且代码编译时已知这些类型时，这是完全合适的解决方案。

然而，有时我们希望库的用户能够扩展在特定场景中有效的类型集合。为了展示如何实现这一点，我们将创建一个图形用户界面（GUI）工具示例：它遍历一个项目列表，对每一项调用 `draw` 方法，把它绘制到屏幕上；这是 GUI 工具的常用技巧。我们将创建一个名为 `gui` 的库 crate，其中包含 GUI 库的结构。这个 crate 可能包含一些供人们使用的类型，例如 `Button` 或 `TextField`。此外，`gui` 用户还希望创建自己可绘制的类型：例如，一名程序员可能添加 `Image`，另一名可能添加 `SelectBox`。

编写库时，我们不可能知道并定义其他程序员可能希望创建的所有类型。但我们知道，`gui` 需要跟踪许多不同类型的值，并对这些不同类型的每个值调用 `draw` 方法。它不需要确切知道调用 `draw` 方法时会发生什么，只需知道该值提供了可供调用的这个方法。

在具有继承的语言中，为实现这一点，可能会定义一个名为 `Component` 的类，其中有一个 `draw` 方法。`Button`、`Image` 和 `SelectBox` 等其他类会继承 `Component`，从而继承 `draw` 方法。每个类都可以重载 `draw` 方法来定义自己的行为，但框架可以把所有类型视为 `Component` 实例，并对它们调用 `draw`。不过，由于 Rust 没有继承，我们需要用另一种方式组织 `gui` 库，让用户能够创建与该库兼容的新类型。

### 为共同行为定义 Trait

为了实现我们希望 `gui` 拥有的行为，将定义名为 `Draw` 的 trait，其中包含一个名为 `draw` 的方法。然后，可以定义一个接收特征对象的向量。<em>特征对象(trait object)</em>既指向实现了指定 trait 的类型实例，也指向一张在运行时用于查找该类型 trait 方法的表。创建特征对象时，需要指定某种指针，例如引用或 `Box<T>` 智能指针，然后写 `dyn` 关键字，再指定相关 trait。（第 20 章[“动态大小类型与 `Sized` Trait”][dynamically-sized]<!-- ignore -->一节将讨论特征对象必须使用指针的原因。）可以用特征对象代替泛型或具体类型。无论在哪里使用特征对象，Rust 的类型系统都会在编译期确保该上下文中使用的任何值都实现特征对象对应的 trait。因此，无需在编译期知道所有可能的类型。

前面提到，在 Rust 中，我们避免把结构体和枚举称为“对象”，以区别于其他语言中的对象。在结构体或枚举中，结构体字段中的数据与 `impl` 块中的行为相互分离；而在其他语言中，组合在一个概念中的数据与行为通常被称为对象。特征对象与其他语言中的对象有所不同，因为不能向特征对象添加数据。特征对象不像其他语言中的对象那样具有广泛用途，其特定目的在于跨越共同行为进行抽象。

示例 18-3 展示了如何定义名为 `Draw`、包含一个 `draw` 方法的 trait。

<Listing number="18-3" file-name="src/lib.rs" caption="`Draw` trait 的定义">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

第 10 章讨论如何定义 trait 时已经见过这种语法。接下来是一些新语法：示例 18-4 定义名为 `Screen` 的结构体，其中保存一个名为 `components` 的向量。这个向量的类型是 `Box<dyn Draw>`，即一个特征对象；它代表 `Box` 中实现 `Draw` trait 的任意类型。

<Listing number="18-4" file-name="src/lib.rs" caption="`Screen` 结构体的定义，其 `components` 字段保存由实现 `Draw` trait 的特征对象构成的向量">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

我们在 `Screen` 结构体上定义名为 `run` 的方法，它会对每个 `components` 调用 `draw` 方法，如示例 18-5 所示。

<Listing number="18-5" file-name="src/lib.rs" caption="`Screen` 上的 `run` 方法，对每个组件调用 `draw` 方法">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

这与定义一个使用带 trait 约束之泛型类型参数的结构体不同。泛型类型参数每次只能由一种具体类型替换，而特征对象允许在运行时由多种具体类型代替。例如，可以像示例 18-6 那样，使用泛型类型和 trait 约束定义 `Screen` 结构体。

<Listing number="18-6" file-name="src/lib.rs" caption="使用泛型与 trait 约束对 `Screen` 结构体及其 `run` 方法的另一种实现">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

这样就把我们限制为：一个 `Screen` 实例所包含的组件列表要么全部为 `Button` 类型，要么全部为 `TextField` 类型。如果始终只使用同构集合，那么更适合使用泛型和 trait 约束，因为定义会在编译期针对具体类型进行单态化。

反之，采用特征对象的方法，一个 `Screen` 实例所保存的 `Vec<T>` 可以同时包含 `Box<Button>` 和 `Box<TextField>`。下面看看它如何工作，然后讨论其运行时性能影响。

### 实现 Trait

现在添加一些实现 `Draw` trait 的类型。我们将提供 `Button` 类型。真正实现 GUI 库仍然超出本书范围，所以 `draw` 方法的方法体中不会包含任何有用的实现。为了想象实际实现可能是什么样子，`Button` 结构体可以包含 `width`、`height` 和 `label` 字段，如示例 18-7 所示。

<Listing number="18-7" file-name="src/lib.rs" caption="实现 `Draw` trait 的 `Button` 结构体">

```rust,noplayground
{{#rustdoc_include ../../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

`Button` 上的 `width`、`height` 和 `label` 字段将不同于其他组件的字段；例如，`TextField` 类型可能包含相同字段，再加上一个 `placeholder` 字段。希望绘制到屏幕上的每种类型都会实现 `Draw` trait，但在 `draw` 方法中使用不同代码，定义如何绘制该特定类型，就像这里的 `Button` 一样（如前所述，不包含实际 GUI 代码）。例如，`Button` 类型还可能有一个额外的 `impl` 块，其中包含与用户点击按钮时所发生事情有关的方法。这类方法不适用于 `TextField` 等类型。

如果库的某位用户决定实现一个包含 `width`、`height` 和 `options` 字段的 `SelectBox` 结构体，他们也会为 `SelectBox` 类型实现 `Draw` trait，如示例 18-8 所示。

<Listing number="18-8" file-name="src/main.rs" caption="另一个 crate 使用 `gui`，并为 `SelectBox` 结构体实现 `Draw` trait">

```rust,ignore
{{#rustdoc_include ../../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

现在，库用户可以编写 `main` 函数来创建 `Screen` 实例。他们可以向 `Screen` 实例添加 `SelectBox` 和 `Button`，把每个值放入 `Box<T>`，使其成为特征对象。然后可以对 `Screen` 实例调用 `run` 方法，它会对每个组件调用 `draw`。示例 18-9 展示了这一实现。

<Listing number="18-9" file-name="src/main.rs" caption="使用特征对象存储实现相同 trait 的不同类型值">

```rust,ignore
{{#rustdoc_include ../../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

编写库时，我们不知道有人可能添加 `SelectBox` 类型；但因为 `SelectBox` 实现了 `Draw` trait，也就是实现了 `draw` 方法，`Screen` 实现能够操作并绘制这个新类型。

只关心一个值会响应哪些消息，而不关心它的具体类型，这一概念类似动态类型语言中的<em>鸭子类型(duck typing)</em>：如果它走起来像鸭子、叫起来也像鸭子，那它就一定是鸭子！示例 18-5 中 `Screen` 上的 `run` 实现不需要知道每个组件的具体类型。它不检查组件是 `Button` 还是 `SelectBox` 实例，只对组件调用 `draw` 方法。通过把 `components` 向量中值的类型指定为 `Box<dyn Draw>`，我们定义了 `Screen` 需要能调用 `draw` 方法的值。

使用特征对象和 Rust 的类型系统编写类似鸭子类型代码的好处在于，永远不必在运行时检查值是否实现某个特定方法，也不必担心值没有实现某个方法却调用它而产生错误。如果值没有实现特征对象所需的 trait，Rust 就不会编译代码。

例如，示例 18-10 展示了尝试创建一个把 `String` 作为组件的 `Screen` 时会发生什么。

<Listing number="18-10" file-name="src/main.rs" caption="尝试使用未实现特征对象对应 trait 的类型">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

因为 `String` 没有实现 `Draw` trait，将得到以下错误：

```console
{{#include ../../listings/ch18-oop/listing-18-10/output.txt}}
```

这条错误告诉我们：要么向 `Screen` 传入了本不打算传入的内容，应改传另一类型；要么应当为 `String` 实现 `Draw`，让 `Screen` 能够对它调用 `draw`。

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### 执行动态分派

回想第 10 章[“使用泛型的代码性能”][performance-of-code-using-generics]<!-- ignore -->一节对编译器为泛型执行单态化过程的讨论：编译器会为用于替换泛型类型参数的每个具体类型，生成函数和方法的非泛型实现。单态化得到的代码执行<em>静态分派(static dispatch)</em>，即编译器在编译期知道调用哪个方法。与之相对的是<em>动态分派(dynamic dispatch)</em>，即编译器在编译期无法确定调用哪个方法。在动态分派场景中，编译器生成的代码会在运行时得知应调用哪个方法。

使用特征对象时，Rust 必须使用动态分派。编译器不知道使用特征对象的代码可能会搭配哪些类型，因此不知道应调用哪个类型上实现的哪个方法。相反，Rust 在运行时使用特征对象内的指针来确定调用哪个方法。这种查找会产生静态分派所没有的运行时成本。动态分派还会阻止编译器选择内联方法代码，进而阻碍某些优化；Rust 对何处可以和不可以使用动态分派还有一些称为 <em>dyn 兼容性(dyn compatibility)</em>的规则。这些规则超出了本节讨论范围，但可以在[参考手册][dyn-compatibility]<!-- ignore -->中进一步了解。不过，我们确实为示例 18-5 中编写并在示例 18-9 中支持的代码获得了额外灵活性，因此这是一项需要考虑的权衡。

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
