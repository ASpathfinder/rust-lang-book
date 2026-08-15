<a id="methods"></a>

## 方法

方法与函数相似：使用 `fn` 关键字和名称声明，可以有形参和返回值，并包含从其他
位置调用方法时运行的代码。不同之处是，方法定义在结构体的上下文中（也可以是
枚举或特征对象，[第 6 章][enums]<!-- ignore -->和[第 18 章][trait-objects]
<!-- ignore -->会分别介绍），而且第一个形参始终是 `self`，
表示调用该方法的结构体实例。

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

<a id="method-syntax"></a>

### 方法语法

把接收 `Rectangle` 实例的 `area` 函数改为定义在 `Rectangle` 结构体上的 `area`
方法，如示例 5-13 所示。

<Listing number="5-13" file-name="src/main.rs" caption="在 `Rectangle` 结构体上定义 `area` 方法">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

为了在 `Rectangle` 的上下文中定义函数，先为它建立 `impl`（implementation，
实现）块。块中的所有内容都与 `Rectangle` 类型关联。接着把 `area` 函数移进
`impl` 的花括号，并把签名和函数体中的第一个（本例也是唯一一个）形参改为
`self`。在 `main` 中，原来调用 `area` 函数并传入 `rect1`，现在可以使用<em>方法
语法(method syntax)</em>在 `Rectangle` 实例上调用 `area`：实例后依次写点号、
方法名、圆括号及所有实参。

`area` 的签名使用 `&self`，而不是 `rectangle: &Rectangle`。`&self` 实际是
`self: &Self` 的简写。在 `impl` 块中，`Self` 是该块所对应类型的别名。方法的
第一个形参必须名为 `self` 且类型为 `Self`，因此 Rust 允许只写 `self`。仍需在
简写前使用 `&`，表示方法借用 `Self` 实例，就像 `rectangle: &Rectangle` 一样。
方法可以取得 `self` 的所有权、像这里一样不可变地借用它，也可以可变地借用它，
与其他形参完全相同。

选择 `&self` 的理由与函数版本使用 `&Rectangle` 相同：只想读取结构体数据，既
不取得所有权，也不写入数据。如果方法需要修改调用它的实例，第一个形参应为
`&mut self`。只使用 `self` 取得实例所有权的方法较少见；通常用于把 `self` 转换
成其他内容，并阻止调用者在转换后继续使用原实例。

除了方法语法和无须在每个签名中重复 `self` 类型之外，使用方法而非函数的主要
理由是便于组织。类型实例能执行的操作都集中在一个 `impl` 块中，代码使用者不必
在库的不同位置寻找 `Rectangle` 的功能。

方法可以与结构体字段同名。例如，可以为 `Rectangle` 定义同样名为 `width` 的
方法：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

这里让 `width` 方法在实例的 `width` 字段大于 `0` 时返回 `true`，等于 `0` 时
返回 `false`。同名方法可以出于任何目的使用字段。在 `main` 中，`rect1.width`
后有圆括号时，Rust 知道指的是 `width` 方法；没有圆括号时则指 `width` 字段。

方法与字段同名时，通常（但并非总是）只返回字段值，不执行其他操作。这种方法称为
<em>获取器(getter)</em>。与某些语言不同，Rust 不会自动为结构体字段实现获取器。
获取器很有用：可以把字段设为私有、方法设为公开，从而在类型的公开 API 中提供
对字段的只读访问。[第 7 章][public]<!-- ignore -->会讨论公开与私有，以及如何将
字段或方法指定为公开或私有。

<a id="wheres-the---operator"></a>

> ### `->` 运算符在哪里？
>
> C 和 C++ 使用两种不同运算符调用方法：直接在对象上调用时使用 `.`；在对象指针
> 上调用、需要先解引用指针时使用 `->`。换句话说，如果 `object` 是指针，
> `object->something()` 类似于 `(*object).something()`。
>
> Rust 没有与 `->` 对应的运算符，而是提供<em>自动引用和解引用(automatic
> referencing and dereferencing)</em>。调用方法是 Rust 中少数具有这种行为的地方。
>
> 工作方式如下：调用 `object.something()` 时，Rust 会自动添加 `&`、`&mut` 或
> `*`，使 `object` 与方法签名匹配。因此，下面两种写法相同：
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 第一种写法明显更简洁。自动引用能够工作，是因为方法有明确的<em>接收者
> (receiver)</em>，即 `self` 的类型。根据接收者和方法名，Rust 可以明确判断方法
> 是读取（`&self`）、修改（`&mut self`）还是消耗（`self`）值。Rust 对方法接收者
> 隐式借用，是所有权机制在实践中易于使用的重要原因。

### 带有更多形参的方法

再为 `Rectangle` 实现第二个方法进行练习。这次希望一个 `Rectangle` 实例接收
另一个实例；如果第二个矩形能完全放入 `self`（第一个矩形）就返回 `true`，否则
返回 `false`。定义 `can_hold` 后，希望能编写示例 5-14 所示的程序。

<Listing number="5-14" file-name="src/main.rs" caption="使用尚未编写的 `can_hold` 方法">

```rust,ignore
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

因为 `rect2` 的两个尺寸都小于 `rect1`，而 `rect3` 比 `rect1` 宽，预期输出如下：

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

要定义方法，所以它位于 `impl Rectangle` 块中。方法名为 `can_hold`，以另一个
`Rectangle` 的不可变借用作为形参。从调用代码可以判断形参类型：
`rect1.can_hold(&rect2)` 传入 `&rect2`，也就是 `Rectangle` 实例 `rect2` 的不可变
借用。这很合理，因为只需读取而非写入 `rect2`；同时希望 `main` 保留其所有权，
以便调用后继续使用。`can_hold` 返回布尔值，实现会分别检查 `self` 的宽高是否都
大于另一个 `Rectangle` 的宽高。示例 5-15 把新方法加入示例 5-13 的 `impl` 块。

<Listing number="5-15" file-name="src/main.rs" caption="在 `Rectangle` 上实现以另一个 `Rectangle` 实例为形参的 `can_hold` 方法">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

把这段代码与示例 5-14 的 `main` 一起运行，会得到预期输出。方法可以在 `self`
之后添加多个形参，它们与函数形参的工作方式相同。

### 关联函数

`impl` 块中定义的所有函数都称为<em>关联函数(associated function)</em>，因为它们与
`impl` 后指定的类型相关联。也可以定义第一个形参不是 `self` 的关联函数（因此
它们不是方法），因为工作时不需要类型的实例。我们已经使用过一个：定义在
`String` 类型上的 `String::from`。

不是方法的关联函数通常用作返回结构体新实例的构造函数。它们往往名为 `new`，
但 `new` 并非特殊名称，也不是语言内置功能。例如，可以提供名为 `square` 的
关联函数，接收一个尺寸并同时用作宽和高，这样创建正方形 `Rectangle` 时就不必
两次指定相同值：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

返回类型和函数体中的 `Self` 关键字，是 `impl` 关键字后类型的别名，本例即
`Rectangle`。

调用关联函数时，使用结构体名称和 `::` 语法，例如
`let sq = Rectangle::square(3);`。该函数位于结构体的命名空间中：关联函数和
模块创建的命名空间都使用 `::`。[第 7 章][modules]<!-- ignore -->会讨论模块。

### 多个 `impl` 块

每个结构体可以拥有多个 `impl` 块。例如，示例 5-15 等价于示例 5-16；后者把
每个方法分别放在自己的 `impl` 块中。

<Listing number="5-16" caption="使用多个 `impl` 块重写示例 5-15">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

这里没有理由把方法拆到多个 `impl` 块，但语法完全有效。第 10 章讨论泛型和特征
时，会看到多个 `impl` 块派上用场的情况。

## 小结

结构体允许你创建对程序领域有意义的自定义类型。使用结构体可以让相关数据保持
关联，并为每项数据命名，使代码更清晰。在 `impl` 块中，可以定义与类型关联的
函数；方法则是一类关联函数，用来指定结构体实例具有的行为。

不过，结构体并非创建自定义类型的唯一方式。接下来转向 Rust 的枚举，再为工具箱
增添一种工具。

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.html
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
