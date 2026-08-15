## 引用模块树中条目的路径

为了告诉 Rust 到模块树的何处查找条目，我们像在文件系统中导航一样使用路径。
调用函数时，需要知道它的路径。

路径有两种形式：

- <em>绝对路径(absolute path)</em>是从 crate 根开始的完整路径；外部 crate 的代码
  以 crate 名称开头，当前 crate 的代码以字面量 `crate` 开头。
- <em>相对路径(relative path)</em>从当前模块开始，使用 `self`、`super` 或当前模块
  中的标识符。

绝对路径和相对路径后都跟着一个或多个标识符，并用双冒号 `::` 分隔。

回到示例 7-1，假设要调用 `add_to_waitlist`，也就是要找出它的路径。示例 7-3
删去了示例 7-1 的部分模块和函数。

我们会展示两种方式，从 crate 根中新定义的 `eat_at_restaurant` 函数调用
`add_to_waitlist`。这些路径本身正确，但还有另一个问题使示例暂时无法编译，稍后
会解释原因。

`eat_at_restaurant` 属于库 crate 的公开 API，所以用 `pub` 标记它。[“使用 `pub`
关键字暴露路径”][pub]<!-- ignore -->一节会详细介绍 `pub`。

<Listing number="7-3" file-name="src/lib.rs" caption="使用绝对路径和相对路径调用 `add_to_waitlist`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

第一次调用使用绝对路径。`add_to_waitlist` 与 `eat_at_restaurant` 定义在同一 crate，
因此可以用 `crate` 关键字开始绝对路径，再依次写出每层模块，直到函数。可以想象
结构相同的文件系统：运行程序需要路径 `/front_of_house/hosting/add_to_waitlist`；
用 `crate` 从 crate 根开始，就像在 shell 中用 `/` 从文件系统根开始。

第二次调用使用相对路径，从与 `eat_at_restaurant` 位于模块树同一层的模块名
`front_of_house` 开始。对应的文件系统路径是
`front_of_house/hosting/add_to_waitlist`。以模块名开头表示路径是相对的。

选择相对路径还是绝对路径取决于项目，也取决于条目定义与使用代码更可能分开移动
还是一起移动。例如，把 `front_of_house` 和 `eat_at_restaurant` 一起移入
`customer_experience` 模块，需要更新绝对路径，但相对路径仍有效。如果单独把
`eat_at_restaurant` 移入 `dining` 模块，绝对路径不变，相对路径却要更新。我们
通常偏好绝对路径，因为代码定义和条目调用更可能独立移动。

尝试编译示例 7-3，看看为何仍无法编译。错误见示例 7-4。

<Listing number="7-4" caption="构建示例 7-3 代码时的编译器错误">

```console
{{#include ../../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

错误说明 `hosting` 模块是私有的。虽然它和 `add_to_waitlist` 的路径正确，Rust 却
不允许使用，因为无法访问私有部分。Rust 中所有条目（函数、方法、结构体、枚举、
模块和常量）默认都对父模块私有。要让函数或结构体等条目保持私有，可以把它放在
模块中。

父模块中的条目不能使用子模块的私有条目，但子模块中的条目可以使用祖先模块的
条目。这是因为子模块封装并隐藏实现细节，却能看到定义自身的上下文。继续用餐厅
比喻，私有规则就像餐厅后勤办公室：里面发生的事情对顾客保密，但办公室经理可以
查看和处理自己管理餐厅中的一切。

Rust 这样设计模块系统，让隐藏内部实现细节成为默认行为，从而明确哪些内部代码
可以修改而不破坏外部代码。不过，也可以用 `pub` 将条目设为公开，把子模块内部
部分暴露给外部祖先模块。

<a id="exposing-paths-with-the-pub-keyword"></a>

### 使用 `pub` 关键字暴露路径

回到示例 7-4 的错误。希望父模块中的 `eat_at_restaurant` 能访问子模块中的
`add_to_waitlist`，所以像示例 7-5 那样，用 `pub` 标记 `hosting` 模块。

<Listing number="7-5" file-name="src/lib.rs" caption="把 `hosting` 声明为 `pub`，以便从 `eat_at_restaurant` 使用它">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

遗憾的是，示例 7-5 仍会产生示例 7-6 中的编译错误。

<Listing number="7-6" caption="构建示例 7-5 代码时的编译器错误">

```console
{{#include ../../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

发生了什么？在 `mod hosting` 前添加 `pub` 会公开模块；能访问 `front_of_house`
时，也就能访问 `hosting`。但 `hosting` 的<em>内容</em>仍然私有，公开模块并不会自动
公开其中内容。模块上的 `pub` 只允许祖先模块中的代码引用模块本身，不能访问内部
代码。模块只是容器，单独公开它能做的事情不多；还必须选择公开其中一个或多个条目。

示例 7-6 的错误指出 `add_to_waitlist` 是私有函数。私有规则既适用于模块，也适用
于结构体、枚举、函数和方法。

像示例 7-7 一样，在定义前添加 `pub`，把 `add_to_waitlist` 也设为公开。

<Listing number="7-7" file-name="src/lib.rs" caption="为 `mod hosting` 和 `fn add_to_waitlist` 添加 `pub`，便能从 `eat_at_restaurant` 调用该函数">

```rust,noplayground,test_harness
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

现在代码能够编译！分别看看绝对路径和相对路径，理解添加 `pub` 后为何符合私有性
规则。

绝对路径从模块树根 `crate` 开始。`front_of_house` 定义在 crate 根中。虽然它不
公开，但 `eat_at_restaurant` 与它定义在同一模块中（两者是同级条目），因此前者
可以引用后者。接下来 `hosting` 标记了 `pub`；我们能访问它的父模块，所以能访问
它。最后 `add_to_waitlist` 也标记 `pub`，且能访问其父模块，因此调用有效。

相对路径的逻辑相同，只有第一步不同：它不从 crate 根开始，而从
`front_of_house` 开始。后者与 `eat_at_restaurant` 定义在同一模块中，因此相对
路径起点有效；`hosting` 和 `add_to_waitlist` 都标记了 `pub`，路径其余部分也有效。

如果计划共享库 crate 供其他项目使用，公开 API 就是你与 crate 用户之间的契约，
决定他们如何与代码交互。管理公开 API 的变更、让他人更容易依赖 crate，需要考虑
许多因素，超出本书范围；如有兴趣，请参阅 [Rust API 指南][api-guidelines]。

> #### 同时包含二进制和库的包的最佳实践
>
> 包可以同时包含二进制 crate 根 <em>src/main.rs</em> 和库 crate 根
> <em>src/lib.rs</em>，默认情况下二者都与包同名。通常，这类包的二进制 crate 只
> 包含足以启动可执行文件并调用库 crate 代码的少量代码。这样，库代码能够共享，
> 其他项目可以利用包提供的大部分功能。
>
> 模块树应定义在 <em>src/lib.rs</em> 中。二进制 crate 可以从包名开始写路径，使用
> 所有公开条目。它就像完全外部的 crate 一样成为库 crate 的用户，只能使用公开
> API。这有助于设计良好 API：你不仅是作者，也是客户！
>
> [第 12 章][ch12]<!-- ignore -->会用同时包含二进制和库 crate 的命令行程序演示
> 这种组织方式。

### 使用 `super` 开始相对路径

路径开头使用 `super`，可以从父模块而非当前模块或 crate 根开始构造相对路径。
这就像用表示前往父目录的 `..` 开始文件系统路径。`super` 可以引用确定存在于父
模块中的条目；当模块与父模块关系密切、但将来可能一起移到模块树其他位置时，
这会让重新组织更加容易。

示例 7-8 模拟厨师修正错误订单并亲自送给顾客的场景。`back_of_house` 模块中的
`fix_incorrect_order` 使用以 `super` 开始的路径，调用父模块中的 `deliver_order`。

<Listing number="7-8" file-name="src/lib.rs" caption="使用以 `super` 开始的相对路径调用函数">

```rust,noplayground,test_harness
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

`fix_incorrect_order` 位于 `back_of_house` 中，所以可以用 `super` 前往其父模块，
本例即根模块 `crate`，再从那里找到 `deliver_order`。成功！我们认为二者很可能
保持相对关系，重组模块树时也会一起移动，因此使用 `super` 可以减少将来移动代码
时需要更新的位置。

### 公开结构体和枚举

也可以用 `pub` 公开结构体和枚举，但两者各有一些细节。在结构体定义前使用 `pub`
会公开结构体，但字段仍然私有；可以逐个决定是否公开字段。示例 7-9 定义公开的
`back_of_house::Breakfast`，其中 `toast` 公开，`seasonal_fruit` 私有。这模拟顾客
可以选择餐点搭配的面包，但厨师根据季节和库存决定水果的情况。可用水果变化很快，
所以顾客不能选择，甚至无法预先知道会得到什么。

<Listing number="7-9" file-name="src/lib.rs" caption="包含部分公开字段和部分私有字段的结构体">

```rust,noplayground
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

由于 `toast` 公开，`eat_at_restaurant` 可以用点号表示法读写它。`seasonal_fruit`
私有，因此不能在该函数中使用。尝试取消修改它那一行的注释，看看会得到什么错误！

另外，因为 `Breakfast` 有私有字段，结构体必须提供公开关联函数来构造实例（这里
名为 `summer`）。如果没有这个函数，就无法在 `eat_at_restaurant` 中创建实例，
因为在那里不能设置私有字段 `seasonal_fruit`。

相比之下，公开枚举后，其所有变体都会公开。只需在 `enum` 前使用 `pub`，如示例
7-10 所示。

<Listing number="7-10" file-name="src/lib.rs" caption="把枚举设为公开会公开它的所有变体">

```rust,noplayground
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

因为 `Appetizer` 已公开，所以能在 `eat_at_restaurant` 中使用 `Soup` 和 `Salad`。

如果枚举变体不公开，枚举就没什么用；每次都给全部变体标注 `pub` 也很麻烦，所以
变体默认公开。结构体即使字段不公开也经常有用，因此字段遵循一般规则：除非标注
`pub`，否则默认私有。

还有一种涉及 `pub` 的情况尚未介绍，也就是模块系统的最后一项功能：`use` 关键字。
下一节先单独介绍 `use`，再展示怎样组合 `pub` 和 `use`。

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
