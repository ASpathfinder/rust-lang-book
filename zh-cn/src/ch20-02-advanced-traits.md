<a id="advanced-traits"></a>

## 高级 Trait

我们在第 10 章的[“使用 Trait 定义共同行为”][traits]<!-- ignore -->一节首次介绍了 trait，但没有讨论更高级的细节。现在你对 Rust 有了更多了解，我们可以深入这些细枝末节了。

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>
<a id="associated-types"></a>

### 使用关联类型定义 Trait

<em>关联类型(associated type)</em>把类型占位符与 trait 连接起来，使 trait 方法定义能在签名中使用这些占位符类型。trait 的实现者会为具体实现指定用于替代占位符的具体类型。这样，我们可以定义使用某些类型的 trait，而在实现 trait 之前无需确切知道这些类型是什么。

本章介绍的大多数高级功能都很少需要使用。关联类型介于两者之间：它们的使用频率低于本书其余部分介绍的功能，却高于本章讨论的许多其他功能。

标准库提供的 `Iterator` trait 就是带有关联类型的一个例子。这个关联类型名为 `Item`，代表实现 `Iterator` trait 的类型所迭代的值的类型。`Iterator` trait 的定义如示例 20-13 所示。

<Listing number="20-13" caption="带有关联类型 `Item` 的 `Iterator` trait 定义">

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

类型 `Item` 是一个占位符，而 `next` 方法的定义表明它将返回 `Option<Self::Item>` 类型的值。`Iterator` trait 的实现者会为 `Item` 指定具体类型，`next` 方法则返回包含该具体类型值的 `Option`。

关联类型可能看起来与泛型概念相似，因为泛型也允许我们在不指定函数能够处理哪些类型的情况下定义函数。为了考察这两个概念的区别，我们来看 `Iterator` trait 在名为 `Counter` 的类型上的一种实现，它指定 `Item` 类型为 `u32`：

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

这种语法看起来与泛型相当。那么，为什么不直接像示例 20-14 那样用泛型定义 `Iterator` trait 呢？

<Listing number="20-14" caption="使用泛型的假想 `Iterator` trait 定义">

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

区别在于，像示例 20-14 那样使用泛型时，我们必须在每个实现中标注类型；因为还可以为 `Counter` 实现 `Iterator<String>` 或任何其他类型，所以 `Counter` 可以拥有多个 `Iterator` 实现。换句话说，当 trait 带有泛型参数时，可以为同一种类型多次实现它，并在每次实现中改变泛型类型参数的具体类型。在 `Counter` 上使用 `next` 方法时，我们必须提供类型标注，指出想使用哪个 `Iterator` 实现。

使用关联类型则不需要标注类型，因为我们不能为同一种类型多次实现一个 trait。在示例 20-13 使用关联类型的定义中，我们只能选择一次 `Item` 的类型，因为只能存在一个 `impl Iterator for Counter`。在 `Counter` 上调用 `next` 的每个位置，都无需指定我们想要一个 `u32` 值的迭代器。

关联类型也会成为 trait 契约的一部分：trait 的实现者必须提供一个类型来替代关联类型占位符。关联类型通常具有能够描述其用途的名称；在 API 文档中记录关联类型是一种良好实践。

<!-- Old headings. Do not remove or links may break. -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### 使用默认泛型参数与运算符重载

使用泛型类型参数时，可以为泛型类型指定默认的具体类型。如果默认类型适用，trait 的实现者便无需再指定具体类型。声明泛型类型时，使用 `<PlaceholderType=ConcreteType>` 语法指定默认类型。

这项技术很适合<em>运算符重载(operator overloading)</em>：即在特定情况下自定义运算符（例如 `+`）的行为。

Rust 不允许创建自己的运算符，也不允许重载任意运算符。不过，通过实现与运算符关联的 trait，你可以重载 `std::ops` 中列出的操作及其对应 trait。例如，在示例 20-15 中，我们重载 `+` 运算符，将两个 `Point` 实例相加。具体做法是为 `Point` 结构体实现 `Add` trait。

<Listing number="20-15" file-name="src/main.rs" caption="实现 `Add` trait，为 `Point` 实例重载 `+` 运算符">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

`add` 方法把两个 `Point` 实例的 `x` 值以及两个实例的 `y` 值分别相加，以创建新的 `Point`。`Add` trait 有一个名为 `Output` 的关联类型，用来确定 `add` 方法的返回类型。

这段代码中的默认泛型类型位于 `Add` trait 内。其定义如下：

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

这段代码总体上应该很熟悉：它是一个包含一个方法和一个关联类型的 trait。新内容是 `Rhs=Self`：这种语法称为<em>默认类型参数(default type parameter)</em>。泛型类型参数 `Rhs`（“right-hand side”，右侧的缩写）定义了 `add` 方法中 `rhs` 形参的类型。如果实现 `Add` trait 时没有为 `Rhs` 指定具体类型，`Rhs` 的类型就默认为 `Self`，也就是我们正在为其实现 `Add` 的类型。

为 `Point` 实现 `Add` 时，我们使用了 `Rhs` 的默认值，因为想把两个 `Point` 实例相加。下面来看一个实现 `Add` trait 时不使用默认值，而是自定义 `Rhs` 类型的例子。

我们有两个结构体 `Millimeters` 和 `Meters`，它们保存采用不同单位的值。这种用另一个结构体薄薄地包装现有类型的方式称为 <em>newtype 模式(newtype pattern)</em>，我们会在[“使用 Newtype 模式实现外部 Trait”][newtype]<!-- ignore -->一节进一步说明。我们想把以毫米表示的值与以米表示的值相加，并让 `Add` 的实现正确完成换算。可以为 `Millimeters` 实现 `Add`，并以 `Meters` 作为 `Rhs`，如示例 20-16 所示。

<Listing number="20-16" file-name="src/lib.rs" caption="为 `Millimeters` 实现 `Add` trait，以便将 `Millimeters` 与 `Meters` 相加">

```rust,noplayground
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

为了将 `Millimeters` 与 `Meters` 相加，我们指定 `impl Add<Meters>` 来设置 `Rhs` 类型参数的值，而不是使用默认的 `Self`。

你主要会以两种方式使用默认类型参数：

1. 在不破坏现有代码的情况下扩展类型
2. 允许在大多数用户不需要的特定情况下进行自定义

标准库的 `Add` trait 是第二种用途的例子：通常会把两个同类值相加，但 `Add` trait 也提供了超越这一点的自定义能力。在 `Add` trait 定义中使用默认类型参数，意味着大多数时候都不必指定额外参数。换言之，无需编写一些实现样板代码，trait 使用起来也就更容易。

第一种用途与第二种用途相似，但方向相反：如果想为现有 trait 添加类型参数，可以为它提供默认值，从而在不破坏现有实现代码的情况下扩展 trait 的功能。

<!-- Old headings. Do not remove or links may break. -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>
<a id="disambiguating-between-methods-with-the-same-name"></a>

### 区分同名方法

Rust 并不阻止一个 trait 拥有与另一个 trait 中方法同名的方法，也不阻止你为一种类型同时实现这两个 trait。还可以直接在类型上实现一个与 trait 方法同名的方法。

调用同名方法时，需要告诉 Rust 想使用哪一个。来看示例 20-17 的代码：我们定义了 `Pilot` 和 `Wizard` 两个 trait，它们都拥有名为 `fly` 的方法。然后在已经直接实现了 `fly` 方法的 `Human` 类型上实现这两个 trait。每个 `fly` 方法执行的操作都不同。

<Listing number="20-17" file-name="src/main.rs" caption="定义两个都包含 `fly` 方法的 trait 并为 `Human` 类型实现它们，同时直接在 `Human` 上实现 `fly` 方法">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

在 `Human` 实例上调用 `fly` 时，编译器默认调用直接在该类型上实现的方法，如示例 20-18 所示。

<Listing number="20-18" file-name="src/main.rs" caption="在 `Human` 实例上调用 `fly`">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

运行这段代码会打印 `*waving arms furiously*`，表明 Rust 调用了直接在 `Human` 上实现的 `fly` 方法。

要调用来自 `Pilot` trait 或 `Wizard` trait 的 `fly` 方法，需要使用更明确的语法指定所指的 `fly` 方法。示例 20-19 展示了这种语法。

<Listing number="20-19" file-name="src/main.rs" caption="指定要调用哪个 trait 的 `fly` 方法">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

在方法名称前指定 trait 名称，就向 Rust 明确了想调用哪个 `fly` 实现。也可以写成 `Human::fly(&person)`，它等价于示例 20-19 中使用的 `person.fly()`；不过，如果不需要消除歧义，这种写法稍长一些。

运行这段代码会打印：

```console
{{#include ../../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

因为 `fly` 方法接收 `self` 形参，所以如果有两个<em>类型</em>都实现同一个 <em>trait</em>，Rust 可以根据 `self` 的类型判断应使用哪个 trait 实现。

然而，不是方法的关联函数没有 `self` 形参。当多个类型或 trait 定义了同名的非方法函数时，如果不使用完全限定语法，Rust 并非总能知道你所指的是哪个类型。例如，在示例 20-20 中，我们为一家想把所有幼犬都命名为 Spot 的动物收容所创建一个 trait。我们创建一个带有非方法关联函数 `baby_name` 的 `Animal` trait。为结构体 `Dog` 实现 `Animal` trait，同时还直接在 `Dog` 上提供一个非方法关联函数 `baby_name`。

<Listing number="20-20" file-name="src/main.rs" caption="一个带有关联函数的 trait，以及一个带有同名关联函数并实现该 trait 的类型">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

我们在 `Dog` 上定义的 `baby_name` 关联函数中实现了把所有幼犬命名为 Spot 的代码。`Dog` 类型还实现了描述所有动物共有特征的 `Animal` trait。幼犬称为 puppy，这一点在 `Dog` 的 `Animal` trait 实现中，通过与 `Animal` trait 关联的 `baby_name` 函数表达出来。

在 `main` 中，我们调用 `Dog::baby_name` 函数，它会调用直接在 `Dog` 上定义的关联函数。这段代码打印：

```console
{{#include ../../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

这不是我们想要的输出。我们想调用为 `Dog` 实现的 `Animal` trait 中的 `baby_name` 函数，使代码打印 `A baby dog is called a puppy`。示例 20-19 中指定 trait 名称的技术在这里无济于事；如果把 `main` 改为示例 20-21 中的代码，会得到编译错误。

<Listing number="20-21" file-name="src/main.rs" caption="尝试调用 `Animal` trait 中的 `baby_name` 函数，但 Rust 不知道应使用哪个实现">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

因为 `Animal::baby_name` 没有 `self` 形参，而且可能有其他类型实现 `Animal` trait，所以 Rust 无法确定我们想使用哪个 `Animal::baby_name` 实现。会得到以下编译器错误：

```console
{{#include ../../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

为了消除歧义，告诉 Rust 我们想使用 `Dog` 的 `Animal` 实现，而不是其他某种类型的 `Animal` 实现，需要使用<em>完全限定语法(fully qualified syntax)</em>。示例 20-22 展示了如何使用完全限定语法。

<Listing number="20-22" file-name="src/main.rs" caption="使用完全限定语法，指定要调用为 `Dog` 实现的 `Animal` trait 中的 `baby_name` 函数">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

我们在尖括号中向 Rust 提供类型标注，表示希望调用为 `Dog` 实现的 `Animal` trait 中的 `baby_name` 方法，也就是在本次函数调用中把 `Dog` 类型视为 `Animal`。现在，这段代码会打印我们想要的内容：

```console
{{#include ../../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

一般而言，完全限定语法的定义如下：

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

对于不是方法的关联函数，不会有 `receiver`，只有其他实参的列表。每次调用函数或方法时都可以使用完全限定语法。不过，这套语法中凡是 Rust 能根据程序中的其他信息推断出的部分，都允许省略。只有在存在多个使用相同名称的实现、Rust 需要帮助才能识别你想调用哪个实现时，才需要使用这种更冗长的语法。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### 使用超 Trait

有时你可能编写依赖另一个 trait 的 trait 定义：要让某种类型实现第一个 trait，你希望要求该类型同时实现第二个 trait。这样做是为了让自己的 trait 定义能够使用第二个 trait 的关联项。你的 trait 定义所依赖的 trait 称为它的<em>超 trait(supertrait)</em>。

例如，假设我们想创建一个带有 `outline_print` 方法的 `OutlinePrint` trait，该方法会打印给定值，并以星号构成的边框包围格式化结果。也就是说，给定一个实现标准库 `Display` trait、显示结果为 `(x, y)` 的 `Point` 结构体，在 `x` 为 `1`、`y` 为 `3` 的 `Point` 实例上调用 `outline_print` 时，应打印：

```text
**********
*        *
* (1, 3) *
*        *
**********
```

在 `outline_print` 方法的实现中，我们想使用 `Display` trait 的功能。因此需要指定：`OutlinePrint` trait 只适用于同时实现 `Display`、能够提供 `OutlinePrint` 所需功能的类型。可以在 trait 定义中指定 `OutlinePrint: Display` 来做到这一点。这项技术类似于给 trait 添加 trait bound。示例 20-23 展示了 `OutlinePrint` trait 的实现。

<Listing number="20-23" file-name="src/main.rs" caption="实现要求具备 `Display` 功能的 `OutlinePrint` trait">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

因为指定了 `OutlinePrint` 要求 `Display` trait，所以我们可以使用会为任何实现 `Display` 的类型自动实现的 `to_string` 函数。如果尝试使用 `to_string`，却没有在 trait 名称后添加冒号并指定 `Display` trait，就会得到错误，指出当前作用域中没有为类型 `&Self` 找到名为 `to_string` 的方法。

来看看尝试为一个没有实现 `Display` 的类型（例如 `Point` 结构体）实现 `OutlinePrint` 时会发生什么：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

我们会得到错误，指出要求实现 `Display`，但它尚未实现：

```console
{{#include ../../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

要修复这个问题，可以像下面这样为 `Point` 实现 `Display`，满足 `OutlinePrint` 的约束：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

然后，为 `Point` 实现 `OutlinePrint` trait 就能成功编译，而且可以在 `Point` 实例上调用 `outline_print`，让它显示在星号边框内。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>
<a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### 使用 Newtype 模式实现外部 Trait

第 10 章[“为类型实现 Trait”][implementing-a-trait-on-a-type]<!-- ignore -->一节提到过<em>孤儿规则(orphan rule)</em>：只有当 trait 或类型中的至少一方位于当前 crate 中时，才允许为该类型实现该 trait。可以利用 newtype 模式绕过这项限制：在元组结构体中创建一个新类型。（第 5 章的[“使用元组结构体创建不同类型”][tuple-structs]<!-- ignore -->一节介绍过元组结构体。）这个元组结构体只有一个字段，作为我们想为其实现 trait 的类型的一层薄包装。这样一来，包装类型位于当前 crate 中，我们就能为它实现 trait。<em>Newtype</em> 一词源自 Haskell 编程语言。使用这种模式没有运行时性能损失，包装类型会在编译时被消除。

例如，假设想为 `Vec<T>` 实现 `Display`，但孤儿规则不允许我们直接这样做，因为 `Display` trait 和 `Vec<T>` 类型都定义在 crate 之外。可以创建一个容纳 `Vec<T>` 实例的 `Wrapper` 结构体，然后为 `Wrapper` 实现 `Display` 并使用其中的 `Vec<T>` 值，如示例 20-24 所示。

<Listing number="20-24" file-name="src/main.rs" caption="围绕 `Vec<String>` 创建 `Wrapper` 类型，以便实现 `Display`">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

`Display` 的实现使用 `self.0` 访问内部的 `Vec<T>`，因为 `Wrapper` 是元组结构体，而 `Vec<T>` 是元组中索引为 0 的项。然后，我们便可以在 `Wrapper` 上使用 `Display` trait 的功能。

这项技术的缺点是 `Wrapper` 是新类型，所以没有它所容纳的值拥有的那些方法。必须直接在 `Wrapper` 上实现 `Vec<T>` 的所有方法，并让这些方法委托给 `self.0`，这样才能把 `Wrapper` 完全当作 `Vec<T>` 使用。如果希望新类型拥有内部类型的所有方法，一种解决方案是为 `Wrapper` 实现 `Deref` trait，使其返回内部类型（第 15 章的[“像普通引用一样使用智能指针”][smart-pointer-deref]<!-- ignore -->一节讨论了 `Deref` trait 的实现）。如果不希望 `Wrapper` 类型拥有内部类型的所有方法——例如为了限制 `Wrapper` 类型的行为——则必须手动只实现我们想要的方法。

即使不涉及 trait，这种 newtype 模式也很有用。现在让我们转换重点，看看与 Rust 类型系统交互的一些高级方式。

[newtype]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
