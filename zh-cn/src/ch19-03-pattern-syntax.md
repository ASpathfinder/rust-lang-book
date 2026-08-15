## 模式语法

本节汇总模式中所有有效语法，并讨论为何以及何时可能希望使用每一种语法。

### 匹配字面量

正如第 6 章所见，可以直接把模式与字面量匹配。以下代码给出了一些示例：

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

这段代码会打印 `one`，因为 `x` 中的值是 `1`。当希望代码在得到某个特定具体值时执行操作，这种语法很有用。

### 匹配命名变量

命名变量是与任何值都匹配的不可反驳模式，本书已经多次使用。不过，在 `match`、`if let` 或 `while let` 表达式中使用命名变量时，情况会略显复杂。由于每种表达式都会开启新作用域，在这些表达式内部作为模式一部分声明的变量，会像所有变量一样遮蔽结构外部的同名变量。在示例 19-11 中，我们声明一个名为 `x`、值为 `Some(5)` 的变量，以及一个名为 `y`、值为 `10` 的变量。然后对值 `x` 创建 `match` 表达式。看看匹配分支中的模式和末尾的 `println!`，尝试在运行代码或继续阅读前判断代码会打印什么。

<Listing number="19-11" file-name="src/main.rs" caption="一个 `match` 表达式，其中某个分支引入新变量，遮蔽现有变量 `y`">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

下面逐步分析 `match` 表达式运行时发生的事情。第一个匹配分支的模式不匹配已定义的 `x` 值，因此代码继续执行。

第二个匹配分支的模式引入名为 `y` 的新变量，它会匹配 `Some` 值内部的任意值。由于现在位于 `match` 表达式内部的新作用域，这是一个新的 `y` 变量，而不是开头声明且值为 `10` 的那个 `y`。这个新的 `y` 绑定会匹配 `Some` 内部的任意值，而 `x` 正是这样的值。因此，新 `y` 会绑定到 `x` 中 `Some` 的内部值。该值为 `5`，所以这个分支的表达式执行并打印 `Matched, y = 5`。

如果 `x` 是 `None` 而不是 `Some(5)`，前两个分支中的模式都不会匹配，因此该值会匹配下划线。下划线分支的模式没有引入 `x` 变量，所以表达式中的 `x` 仍是未被遮蔽的外部 `x`。在这种假设情形中，`match` 会打印 `Default case, x = None`。

`match` 表达式结束时，其作用域结束，内部 `y` 的作用域也随之结束。最后一条 `println!` 会产生 `at the end: x = Some(5), y = 10`。

如果想创建一个比较外部 `x` 和 `y` 值的 `match` 表达式，而不是引入遮蔽现有 `y` 的新变量，就需要改用匹配守卫条件。稍后的[“使用匹配守卫添加条件”](#adding-conditionals-with-match-guards)<!-- ignore -->一节将讨论匹配守卫。

<!-- Old headings. Do not remove or links may break. -->
<a id="multiple-patterns"></a>

### 匹配多个模式

在 `match` 表达式中，可以使用 `|` 语法匹配多个模式，它是模式的<em>或(or)</em>运算符。例如，在以下代码中，我们把 `x` 的值与各匹配分支进行比较；第一个分支包含“或”选项，意味着只要 `x` 的值匹配该分支中的任一值，就会运行该分支的代码：

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

这段代码会打印 `one or two`。

### 使用 `..=` 匹配值的范围

`..=` 语法允许匹配一个闭区间中的值。在以下代码中，当模式匹配给定范围内的任一值时，就会执行相应分支：

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

如果 `x` 是 `1`、`2`、`3`、`4` 或 `5`，第一个分支就会匹配。对于多个匹配值，这种语法比使用 `|` 运算符表达相同概念更方便；如果使用 `|`，就必须指定 `1 | 2 | 3 | 4 | 5`。指定范围要短得多，尤其是希望匹配 1 到 1,000 之间的任意数字时！

编译器会在编译期检查范围是否为空。由于 Rust 只能判断 `char` 和数值类型的范围是否为空，范围只允许用于数值或 `char` 值。

下面是一个使用 `char` 值范围的示例：

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust 能够判断 `'c'` 位于第一个模式的范围内，并打印 `early ASCII letter`。

### 通过解构拆开值

还可以使用模式解构结构体、枚举和元组，以便使用这些值的不同部分。下面逐一介绍。

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs"></a>

#### 结构体

示例 19-12 展示了包含 `x` 和 `y` 两个字段的 `Point` 结构体；可以使用 `let` 语句中的模式将其拆开。

<Listing number="19-12" file-name="src/main.rs" caption="把结构体字段解构为独立变量">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

这段代码创建变量 `a` 和 `b`，分别匹配结构体 `p` 的 `x` 和 `y` 字段值。这个示例表明，模式中的变量名不必与结构体字段名相同。不过，通常会让变量名与字段名相同，以便更容易记住每个变量来自哪个字段。由于这种用法很常见，而且编写 `let Point { x: x, y: y } = p;` 包含大量重复，Rust 为匹配结构体字段的模式提供了简写：只需列出结构体字段名，模式创建的变量会具有相同名称。示例 19-13 的行为与示例 19-12 中的代码相同，但 `let` 模式创建的是变量 `x` 和 `y`，而不是 `a` 和 `b`。

<Listing number="19-13" file-name="src/main.rs" caption="使用结构体字段简写解构结构体字段">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

这段代码创建变量 `x` 和 `y`，匹配变量 `p` 的 `x` 和 `y` 字段。结果是变量 `x` 和 `y` 包含 `p` 结构体中的值。

还可以把字面量作为结构体模式的一部分进行解构，而不是为所有字段创建变量。这样可以测试某些字段是否具有特定值，同时创建变量来解构其他字段。

示例 19-14 中的 `match` 表达式把 `Point` 值分为三种情况：正好位于 `x` 轴上的点（`y = 0` 时成立）、位于 `y` 轴上的点（`x = 0`），以及不在任何一条轴上的点。

<Listing number="19-14" file-name="src/main.rs" caption="在同一模式中解构并匹配字面量值">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

第一个分支通过指定 `y` 字段的值与字面量 `0` 匹配，来匹配位于 `x` 轴上的任意点。该模式仍会创建一个可在此分支代码中使用的 `x` 变量。

类似地，第二个分支通过指定 `x` 字段的值为 `0`，匹配位于 `y` 轴上的任意点，并为 `y` 字段的值创建变量 `y`。第三个分支不指定任何字面量，因此匹配任何其他 `Point`，并为 `x` 和 `y` 字段都创建变量。

在这个示例中，由于 `x` 包含 `0`，值 `p` 匹配第二个分支，所以代码会打印 `On the y axis at 7`。

请记住，`match` 表达式找到第一个匹配模式后就停止检查分支。因此，即使 `Point { x: 0, y: 0 }` 同时位于 `x` 轴和 `y` 轴上，这段代码也只会打印 `On the x axis at 0`。

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

#### 枚举

本书已经解构过枚举（例如第 6 章的示例 6-5），但尚未明确讨论：解构枚举的模式与枚举内部数据的定义方式相对应。作为示例，在示例 19-15 中，我们使用示例 6-2 的 `Message` 枚举，并编写包含模式的 `match`，解构每个内部值。

<Listing number="19-15" file-name="src/main.rs" caption="解构保存不同种类值的枚举变体">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

这段代码会打印 `Change color to red 0, green 160, and blue 255`。尝试修改 `msg` 的值，观察其他分支的代码运行。

对于没有任何数据的枚举变体（例如 `Message::Quit`），无法进一步解构其值。只能匹配字面量值 `Message::Quit`，该模式中没有任何变量。

对于类似结构体的枚举变体（例如 `Message::Move`），可以使用类似于匹配结构体时所指定的模式。在变体名称之后放置花括号，然后列出字段和变量，从而拆开各部分供这个分支的代码使用。这里采用与示例 19-13 相同的简写形式。

对于类似元组的枚举变体，例如保存单元素元组的 `Message::Write` 和保存三元素元组的 `Message::ChangeColor`，模式类似于匹配元组时所指定的模式。模式中的变量数必须与所匹配变体中的元素数相同。

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

#### 嵌套结构体与枚举

目前为止，所有示例都只匹配了一层深度的结构体或枚举，但匹配也适用于嵌套项！例如，可以重构示例 19-15 中的代码，让 `ChangeColor` 消息支持 RGB 和 HSV 颜色，如示例 19-16 所示。

<Listing number="19-16" caption="匹配嵌套枚举">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

`match` 表达式第一个分支的模式匹配包含 `Color::Rgb` 变体的 `Message::ChangeColor` 枚举变体，然后把三个内部 `i32` 值进行绑定。第二个分支的模式也匹配 `Message::ChangeColor` 枚举变体，但内部枚举改为匹配 `Color::Hsv`。即使涉及两个枚举，也可以在一个 `match` 表达式中指定这些复杂条件。

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

#### 结构体与元组

可以以更加复杂的方式混合、匹配和嵌套解构模式。以下示例展示了一种复杂的解构：在元组内部嵌套结构体和元组，并解构出所有基本值：

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

这段代码让我们能够把复杂类型拆分为组成部分，从而分别使用感兴趣的值。

使用模式解构，是一种分别使用值的各个部分（例如结构体每个字段中的值）的便捷方式。

<a id="ignoring-values-in-a-pattern"></a>

### 忽略模式中的值

你已经看到，有时忽略模式中的值很有用。例如，在 `match` 的最后一个分支中，可以得到一个实际不执行任何操作、但会涵盖所有剩余可能值的全匹配模式。有几种方式可以忽略模式中的整个值或值的一部分：使用已经见过的 `_` 模式；在另一个模式中使用 `_`；使用以下划线开头的名称；或使用 `..` 忽略值的剩余部分。下面探索使用每种模式的方式和原因。

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

#### 使用 `_` 忽略整个值

我们已经把下划线当作<em>通配符模式(wildcard pattern)</em>使用：它会匹配任何值，但不绑定该值。这尤其适合 `match` 表达式的最后一个分支，不过也能用于任何模式，包括函数形参，如示例 19-17 所示。

<Listing number="19-17" file-name="src/main.rs" caption="在函数签名中使用 `_`">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

这段代码会完全忽略作为第一个实参传入的值 `3`，并打印 `This code only uses the y parameter: 4`。

在大多数情况下，如果不再需要某个函数形参，会修改签名，使其不包含未使用的形参。不过，忽略函数形参在某些情况下格外有用。例如，实现 trait 时需要某种类型签名，但你的函数体不需要其中一个形参。这样就能避免编译器对未使用的函数形参发出警告，而使用名称时会收到该警告。

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### 使用嵌套的 `_` 忽略值的一部分

还可以在另一个模式内部使用 `_`，只忽略值的一部分。例如，希望只测试值的一部分，而对应代码不需要使用其他部分时。示例 19-18 展示了负责管理某项设置之值的代码。业务要求是，不允许用户覆盖已有的设置自定义值；但如果该设置当前未设置，则允许用户取消设置并为它提供值。

<Listing number="19-18" caption="不需要使用 `Some` 内部的值时，在匹配 `Some` 变体的模式中使用下划线">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

这段代码会打印 `Can't overwrite an existing customized value`，然后打印 `setting is Some(5)`。在第一个匹配分支中，我们不需要匹配或使用任一 `Some` 变体内部的值，但确实需要测试 `setting_value` 和 `new_setting_value` 都是 `Some` 变体的情况。在这种情况下，打印不更改 `setting_value` 的原因，而它也不会被改变。

在第二个分支的 `_` 模式表示的所有其他情况下（即 `setting_value` 或 `new_setting_value` 中至少一个为 `None`），我们希望允许 `new_setting_value` 成为 `setting_value`。

还可以在一个模式中的多个位置使用下划线，忽略特定值。示例 19-19 展示了忽略五元素元组中第二个和第四个值的例子。

<Listing number="19-19" caption="忽略元组的多个部分">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

这段代码会打印 `Some numbers: 2, 8, 32`，而值 `4` 和 `16` 会被忽略。

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### 通过以下划线开头的名称忽略未使用变量

如果创建变量却没有在任何地方使用它，Rust 通常会发出警告，因为未使用的变量可能是 bug。不过，有时创建一个暂时不使用的变量会很有用，例如制作原型或刚开始项目时。在这种情况下，可以通过让变量名以下划线开头，告诉 Rust 不要对未使用变量发出警告。在示例 19-20 中，我们创建两个未使用的变量，但编译代码时应当只收到其中一个的警告。

<Listing number="19-20" file-name="src/main.rs" caption="让变量名以下划线开头，以避免未使用变量警告">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

这里会收到未使用变量 `y` 的警告，但不会收到未使用 `_x` 的警告。

请注意，只使用 `_` 与使用以下划线开头的名称之间有一个微妙区别。`_x` 语法仍然把值绑定到变量，而 `_` 完全不绑定。为了展示这种区别在哪种情况下很重要，示例 19-21 会产生错误。

<Listing number="19-21" caption="以下划线开头的未使用变量仍会绑定值，这可能取得值的所有权。">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

我们会收到错误，因为值 `s` 仍然会移入 `_s`，使我们无法再次使用 `s`。不过，单独使用下划线永远不会绑定值。示例 19-22 可以无错误地编译，因为 `s` 不会移入 `_`。

<Listing number="19-22" caption="使用下划线不会绑定值。">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

这段代码工作正常，因为我们从未把 `s` 绑定到任何内容；它没有被移动。

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### 使用 `..` 忽略值的剩余部分

对于包含许多部分的值，可以使用 `..` 语法来使用特定部分并忽略其余部分，避免为每个被忽略的值列出下划线。`..` 模式会忽略值中没有在模式其余部分显式匹配的任何部分。在示例 19-23 中，`Point` 结构体保存三维空间中的一个坐标。在 `match` 表达式中，我们只想操作 `x` 坐标，而忽略 `y` 和 `z` 字段中的值。

<Listing number="19-23" caption="使用 `..` 忽略 `Point` 中除 `x` 外的所有字段">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

我们列出 `x` 值，然后只包含 `..` 模式。这比必须列出 `y: _` 和 `z: _` 更快捷，尤其是处理包含许多字段、而只有一两个字段相关的结构体时。

`..` 语法会根据需要扩展为任意数量的值。示例 19-24 展示了如何对元组使用 `..`。

<Listing number="19-24" file-name="src/main.rs" caption="只匹配元组的第一个和最后一个值，忽略所有其他值">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

在这段代码中，第一个和最后一个值分别与 `first` 和 `last` 匹配。`..` 会匹配并忽略中间的一切。

不过，`..` 的用法必须没有歧义。如果无法明确哪些值用于匹配、哪些值应当被忽略，Rust 就会报错。示例 19-25 展示了以有歧义的方式使用 `..`，因此无法编译。

<Listing number="19-25" file-name="src/main.rs" caption="尝试以有歧义的方式使用 `..`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

编译这个示例会得到以下错误：

```console
{{#include ../../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Rust 无法判断，在用 `second` 匹配一个值之前应忽略元组中的多少个值，以及之后又应忽略多少个值。这段代码可能意味着要忽略 `2`，把 `second` 绑定到 `4`，再忽略 `8`、`16` 和 `32`；也可能意味着要忽略 `2` 和 `4`，把 `second` 绑定到 `8`，再忽略 `16` 和 `32`；以此类推。变量名 `second` 对 Rust 没有特殊含义，所以像这样在两个位置使用 `..` 存在歧义，会得到编译错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>
<a id="adding-conditionals-with-match-guards"></a>

### 使用匹配守卫添加条件

<em>匹配守卫(match guard)</em>是在 `match` 分支的模式之后指定的额外 `if` 条件；要选择该分支，这个条件也必须匹配。匹配守卫适合表达仅靠模式无法表达的更复杂概念。不过请注意，它只适用于 `match` 表达式，不适用于 `if let` 或 `while let` 表达式。

条件可以使用模式中创建的变量。示例 19-26 展示了一个 `match`：第一个分支包含模式 `Some(x)`，还有匹配守卫 `if x % 2 == 0`（如果数字为偶数，它就是 `true`）。

<Listing number="19-26" caption="为模式添加匹配守卫">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

这个示例会打印 `The number 4 is even`。把 `num` 与第一个分支中的模式比较时，两者匹配，因为 `Some(4)` 匹配 `Some(x)`。然后，匹配守卫检查 `x` 除以 2 的余数是否等于 0；由于确实如此，第一个分支被选中。

如果 `num` 改为 `Some(5)`，第一个分支中的匹配守卫会为 `false`，因为 5 除以 2 的余数是 1，不等于 0。Rust 随后会转到第二个分支；该分支没有匹配守卫，因此匹配任意 `Some` 变体。

无法在模式内部表达 `if x % 2 == 0` 条件，所以匹配守卫让我们能够表达这种逻辑。额外表达能力的缺点是，涉及匹配守卫表达式时，编译器不会尝试检查穷尽性。

讨论示例 19-11 时，我们提到可以使用匹配守卫解决模式遮蔽问题。回想一下，我们在 `match` 表达式的模式内部创建了新变量，而不是使用 `match` 外部的变量。这个新变量使我们无法与外部变量的值进行比较。示例 19-27 展示了如何使用匹配守卫修复这个问题。

<Listing number="19-27" file-name="src/main.rs" caption="使用匹配守卫测试是否等于外部变量">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

这段代码现在会打印 `Default case, x = Some(5)`。第二个匹配分支中的模式没有引入会遮蔽外部 `y` 的新变量 `y`，这意味着可以在匹配守卫中使用外部 `y`。我们不把模式指定为会遮蔽外部 `y` 的 `Some(y)`，而是指定为 `Some(n)`。这会创建一个不遮蔽任何内容的新变量 `n`，因为 `match` 外部没有变量 `n`。

匹配守卫 `if n == y` 不是模式，因此不会引入新变量。这个 `y` <em>就是</em>外部 `y`，而不是遮蔽它的新 `y`；通过比较 `n` 和 `y`，可以查找与外部 `y` 具有相同值的值。

还可以在匹配守卫中使用“或”运算符 `|` 指定多个模式；匹配守卫条件会应用于所有模式。示例 19-28 展示了把使用 `|` 的模式与匹配守卫组合时的优先级。本例的重点是：匹配守卫 `if y` 会应用于 `4`、`5` <em>和</em> `6`，即使它看起来可能只应用于 `6`。

<Listing number="19-28" caption="把多个模式与匹配守卫组合起来">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

匹配条件表示，只有当 `x` 的值等于 `4`、`5` 或 `6`，<em>并且</em> `y` 为 `true` 时，该分支才匹配。运行这段代码时，第一个分支的模式会匹配，因为 `x` 是 `4`；但匹配守卫 `if y` 为 `false`，所以不会选择第一个分支。代码转到第二个分支，该分支确实匹配，程序打印 `no`。原因在于 `if` 条件应用于整个模式 `4 | 5 | 6`，而不只是最后一个值 `6`。换句话说，匹配守卫相对于模式的优先级表现如下：

```text
(4 | 5 | 6) if y => ...
```

而不是：

```text
4 | 5 | (6 if y) => ...
```

运行代码后，优先级行为非常明显：如果匹配守卫只应用于用 `|` 运算符指定的值列表中的最后一个值，该分支就会匹配，程序将打印 `yes`。

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

### 使用 `@` 绑定

<em>at 运算符(at operator)</em> `@` 让我们能够在测试某个值是否与模式匹配的同时，创建保存该值的变量。在示例 19-29 中，我们希望测试 `Message::Hello` 的 `id` 字段是否位于 `3..=7` 范围内，还希望把该值绑定到变量 `id`，以便在与分支关联的代码中使用它。

<Listing number="19-29" caption="使用 `@` 在模式中测试值的同时绑定该值">

```rust
{{#rustdoc_include ../../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

这个示例会打印 `Found an id in range: 5`。通过在范围 `3..=7` 前指定 `id @`，我们把与该范围匹配的任意值捕获到名为 `id` 的变量中，同时还测试该值与范围模式是否匹配。

在第二个分支中，模式里只指定了范围，与该分支关联的代码没有包含 `id` 字段实际值的变量。`id` 字段的值可能是 10、11 或 12，但对应模式的代码不知道具体是哪一个。由于没有把 `id` 值保存到变量中，模式代码无法使用 `id` 字段的值。

在最后一个分支中，我们指定了一个不带范围的变量，因此分支代码可以通过名为 `id` 的变量使用该值。这是因为使用了结构体字段简写语法。但与前两个分支不同，这个分支没有对 `id` 字段中的值应用任何测试：任何值都会匹配该模式。

使用 `@` 可以在一个模式中测试值并把它保存到变量。

## 小结

Rust 的模式非常适合区分不同种类的数据。在 `match` 表达式中使用模式时，Rust 会确保模式涵盖每个可能值，否则程序无法编译。`let` 语句和函数形参中的模式让这些结构更加实用，使我们能够把值解构为更小的部分，并把这些部分赋给变量。可以根据需要创建简单或复杂的模式。

下一章是本书倒数第二章，我们将了解 Rust 各种功能的一些高级方面。
