<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## 使用模块控制作用域与私有性

本节将讨论模块以及模块系统的其他部分：用于命名条目的<em>路径(path)</em>、把路径
引入作用域的 `use` 关键字，以及将条目设为公开的 `pub` 关键字。还会介绍 `as`
关键字、外部包和 glob 运算符。

### 模块速查表

深入模块与路径的细节之前，先快速总结模块、路径、`use` 和 `pub` 在编译器中的
工作方式，以及大多数开发者如何组织代码。本章后面会逐一展示这些规则的示例，
这里则适合作为模块工作方式的备忘参考。

- <strong>从 crate 根开始</strong>：编译 crate 时，编译器首先在 crate 根文件中查找
  要编译的代码；库 crate 通常是 <em>src/lib.rs</em>，二进制 crate 通常是
  <em>src/main.rs</em>。
- <strong>声明模块</strong>：可以在 crate 根中声明新模块。例如用 `mod garden;`
  声明“garden”模块后，编译器会在以下位置查找模块代码：
  - 直接内联在花括号中，用花括号替代 `mod garden` 后的分号；
  - 文件 <em>src/garden.rs</em>；
  - 文件 <em>src/garden/mod.rs</em>。
- <strong>声明子模块</strong>：可以在 crate 根以外的任何文件中声明子模块。例如，
  在 <em>src/garden.rs</em> 中声明 `mod vegetables;` 后，编译器会在以父模块命名的
  目录中查找子模块代码：
  - 直接内联在 `mod vegetables` 后的花括号中，用花括号替代分号；
  - 文件 <em>src/garden/vegetables.rs</em>；
  - 文件 <em>src/garden/vegetables/mod.rs</em>。
- <strong>模块中代码的路径</strong>：模块成为 crate 的一部分后，只要私有性规则允许，
  就能通过路径从同一 crate 的任何地方引用其中的代码。例如，garden 的 vegetables
  模块中的 `Asparagus` 类型位于 `crate::garden::vegetables::Asparagus`。
- <strong>私有与公开</strong>：模块内的代码默认对其父模块私有。要公开模块，用
  `pub mod` 代替 `mod` 声明；要同时公开公共模块中的条目，在其声明前使用 `pub`。
- <strong>`use` 关键字</strong>：`use` 在作用域中为条目创建快捷方式，减少长路径的
  重复书写。在能引用 `crate::garden::vegetables::Asparagus` 的作用域中，可以用
  `use crate::garden::vegetables::Asparagus;` 创建快捷方式，此后只写 `Asparagus`
  就能在该作用域中使用该类型。

下面创建名为 `backyard` 的二进制 crate 来展示这些规则。名为 <em>backyard</em> 的
crate 目录包含以下文件和目录：

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

这里的 crate 根是 <em>src/main.rs</em>，内容如下：

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

`pub mod garden;` 告诉编译器包含在 <em>src/garden.rs</em> 中找到的代码：

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

这里的 `pub mod vegetables;` 表示还要包含 <em>src/garden/vegetables.rs</em> 中的代码：

```rust,noplayground,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

下面深入这些规则的细节，看看它们如何实际运作！

### 在模块中组织相关代码

<em>模块(module)</em>让我们在 crate 内组织代码，使其易于阅读和复用。模块还允许
控制条目的<em>私有性(privacy)</em>，因为模块中的代码默认私有。私有条目是不能从外部
使用的内部实现细节。可以选择公开模块及其中的条目，将其暴露给外部代码使用和
依赖。

例如，编写一个提供餐厅功能的库 crate。我们只定义函数签名并留空函数体，以便
专注于代码组织，而不是餐厅功能的具体实现。

餐饮业把餐厅的一些区域称为前台，另一些称为后台。<em>前台(front of house)</em>是
顾客所在的区域，包括接待员安排座位、服务员接受订单和付款、调酒师制作饮品的
地方。<em>后台(back of house)</em>是厨师在厨房工作、洗碗工清洁餐具、经理处理行政
事务的地方。

可以把函数放入嵌套模块，按这种方式组织 crate。运行 `cargo new restaurant --lib`
创建名为 `restaurant` 的新库，再把示例 7-1 的代码写入 <em>src/lib.rs</em>，定义一些
模块和函数签名；这些代码属于前台部分。

<Listing number="7-1" file-name="src/lib.rs" caption="包含其他模块，而这些模块又包含函数的 `front_of_house` 模块">

```rust,noplayground
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

定义模块时，先写 `mod` 关键字和模块名（本例为 `front_of_house`），再把模块体
放入花括号。模块内部还可以放置其他模块，例如这里的 `hosting` 和 `serving`。
模块也可以包含结构体、枚举、常量、特征等其他条目的定义，以及示例 7-1 中的函数。

模块可以把相关定义组合在一起，并用名称说明它们为何相关。代码使用者可以按分组
浏览，无须通读全部定义，更容易找到所需内容；添加新功能的程序员也知道应把代码
放在哪里，保持程序组织有序。

前面提到，<em>src/main.rs</em> 和 <em>src/lib.rs</em> 称为 <em>crate 根(crate root)</em>。
这是因为其中任一文件的内容都会在 crate 模块结构的根部形成名为 `crate` 的模块，
这个结构称为<em>模块树(module tree)</em>。

示例 7-2 展示了示例 7-1 结构对应的模块树。

<Listing number="7-2" caption="示例 7-1 代码的模块树">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

这棵树展示模块如何嵌套，例如 `hosting` 嵌套在 `front_of_house` 中。它还显示某些
模块是<em>同级(sibling)</em>的，即定义在同一模块中；`hosting` 和 `serving` 是定义
在 `front_of_house` 中的同级模块。如果模块 A 包含在模块 B 中，就称 A 是 B 的
<em>子模块(child)</em>，B 是 A 的<em>父模块(parent)</em>。整棵模块树都以隐式模块
`crate` 为根。

模块树可能让你想到计算机文件系统的目录树，这个类比十分贴切！就像用目录组织
文件一样，可以用模块组织代码；而就像需要在目录中查找文件一样，我们也需要一种
查找模块的方式。
