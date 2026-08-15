## 定义枚举

结构体把相关字段和数据组合起来，例如包含 `width` 和 `height` 的 `Rectangle`；
枚举则可以表示某个值属于一组可能值中的一个。例如，我们可能想表达
`Rectangle` 是一组形状中的一种，其他形状还有 `Circle` 和 `Triangle`。Rust
允许用枚举编码这些可能性。

来看一个适合用代码表达的场景，理解枚举为何有用，以及这里为何比结构体更合适。
假设需要处理 IP 地址。目前 IP 地址主要采用两个标准：版本 4 和版本 6。程序遇到
的 IP 地址只可能属于这两种情况，因此可以<em>列举(enumerate)</em>所有可能变体，
“枚举”之名正来源于此。

任何 IP 地址都只能是版本 4 或版本 6，不能同时属于二者。枚举值只能是其中一个
变体，因此这种性质很适合用枚举数据结构表示。两个版本的地址本质上仍然都是 IP
地址，所以在处理适用于任何 IP 地址的场景时，应将它们视为同一类型。

可以定义 `IpAddrKind` 枚举并列出 IP 地址可能的类型 `V4` 和 `V6`，从而在代码中
表达这个概念。它们是枚举的<em>变体(variant)</em>：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` 现在是可在其他代码中使用的自定义数据类型。

<a id="enum-values"></a>

### 枚举值

可以像这样创建 `IpAddrKind` 两个变体各自的实例：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

枚举变体位于枚举标识符的命名空间下，二者用双冒号分隔。这很有用，因为
`IpAddrKind::V4` 和 `IpAddrKind::V6` 现在具有相同类型 `IpAddrKind`。例如，可以
定义接收任意 `IpAddrKind` 的函数：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

两个变体都可以传给这个函数：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

枚举还有更多优点。继续思考 IP 地址类型：目前无法存储实际的 IP 地址<em>数据</em>，
只知道它属于哪种<em>类型</em>。刚学过第 5 章的结构体，你可能想用示例 6-1 所示的
结构体解决这个问题。

<Listing number="6-1" caption="使用 `struct` 存储 IP 地址的数据及 `IpAddrKind` 变体">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

这里定义了有两个字段的 `IpAddr` 结构体：`kind` 的类型是前面定义的
`IpAddrKind`，`address` 的类型是 `String`。结构体有两个实例。第一个是 `home`，
它的 `kind` 为 `IpAddrKind::V4`，关联的地址数据为 `127.0.0.1`。第二个是
`loopback`，其 `kind` 是另一个变体 `V6`，关联地址为 `::1`。结构体把 `kind`
和 `address` 打包在一起，使变体与地址值产生关联。

不过，只用枚举表示同一概念会更加简洁：不必在结构体中嵌入枚举，可以直接把数据
放进每个枚举变体。新的 `IpAddr` 定义表示 `V4` 和 `V6` 变体各自关联一个
`String` 值：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

数据直接附加到枚举变体上，不再需要额外结构体。这里还能更清楚地看到枚举工作的
另一个细节：定义的每个枚举变体名称也会成为构造枚举实例的函数。也就是说，
`IpAddr::V4()` 是一个函数调用，接收 `String` 实参并返回 `IpAddr` 实例。定义
枚举后，会自动获得这个构造函数。

枚举相对结构体还有一个优势：每个变体可以关联不同类型和数量的数据。版本 4 IP
地址始终由四个 0 到 255 之间的数字组成。假如想用四个 `u8` 值存储 `V4` 地址，
同时仍用一个 `String` 表示 `V6`，结构体无法做到，枚举却能轻松处理：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

上面展示了多种保存两个版本 IP 地址并编码其类型的数据结构。实际上，存储 IP
地址并标记其类型的需求非常普遍，[标准库已经提供了可用的定义][IpAddr]
<!-- ignore -->！标准库的 `IpAddr` 与我们定义并使用的枚举及变体完全相同，但它
把地址数据嵌入两种不同结构体中，每种变体的结构体定义也不同：

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

这说明枚举变体可以放入任何类型的数据，例如字符串、数值类型或结构体，甚至可以
包含另一个枚举！标准库类型往往也不会比你自己能想到的复杂多少。

即使标准库包含 `IpAddr` 定义，我们仍能无冲突地创建并使用自己的定义，因为尚未
把标准库定义引入作用域。第 7 章会进一步讨论如何把类型引入作用域。

示例 6-2 是另一个枚举，其变体嵌入了多种不同类型。

<Listing number="6-2" caption="各变体存储不同数量和类型值的 `Message` 枚举">

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

该枚举有四个不同类型的变体：

- `Quit`：不关联任何数据。
- `Move`：包含像结构体一样的命名字段。
- `Write`：包含一个 `String`。
- `ChangeColor`：包含三个 `i32` 值。

定义示例 6-2 这样的枚举，与定义多种结构体相似；不同之处是枚举不使用 `struct`
关键字，而且所有变体都归入 `Message` 类型。以下结构体可以保存与前述枚举变体
相同的数据：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

但如果使用这些不同结构体，每个结构体都有自己的类型，就无法像使用单一类型的
`Message` 枚举那样轻松定义一个接收所有消息类型的函数。

枚举与结构体还有一个相似之处：既然可以用 `impl` 为结构体定义方法，也可以为
枚举定义方法。下面在 `Message` 枚举上定义名为 `call` 的方法：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

方法体使用 `self` 取得调用方法的值。本例创建变量 `m`，其值为
`Message::Write(String::from("hello"))`；运行 `m.call()` 时，`call` 方法体中的
`self` 就是这个值。

下面看看标准库中另一个十分常见且实用的枚举：`Option`。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-option-enum-and-its-advantages-over-null-values"></a>

### `Option` 枚举

本节研究标准库定义的另一个枚举 `Option`。`Option` 类型编码一种十分常见的场景：
一个值可能存在，也可能不存在。

例如，请求非空列表的第一个元素会得到一个值；请求空列表的第一个元素则得不到
任何内容。用类型系统表达这个概念，编译器便能检查是否处理了所有必要情况；这种
功能可以避免其他编程语言中极其常见的错误。

谈论编程语言设计时，人们往往关注加入了哪些特性，但没有加入的特性同样重要。
Rust 没有许多其他语言具备的 null。<em>空值(null)</em>表示这里没有值。在具有 null
的语言中，变量总是处于两种状态之一：null 或非 null。

null 的发明者 Tony Hoare 在 2009 年的演讲“Null References: The Billion Dollar
Mistake”中说：

> 我把它称为自己的十亿美元错误。当时，我正在为一门面向对象语言设计首个完整的
> 引用类型系统。我的目标是确保所有引用用法都绝对安全，由编译器自动执行检查。
> 但我没能抵挡加入空引用的诱惑，只因为它太容易实现。这导致了无数错误、漏洞和
> 系统崩溃；过去四十年间造成的痛苦和损失很可能价值十亿美元。

空值的问题在于，如果把空值当作非空值使用，就会发生某种错误。由于“空或非空”
这种性质无处不在，所以极易犯下这类错误。

不过，null 想表达的概念本身仍然有用：某个值因为某种原因当前无效或不存在。
真正的问题不在概念，而在具体实现。因此 Rust 没有 null，却提供了一个能编码值
存在或不存在这一概念的枚举。它就是 `Option<T>`，[标准库中的定义][option]
<!-- ignore -->如下：

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` 非常实用，甚至包含在预导入模块中，无须显式引入作用域。它的变体也
包含在预导入模块中，可以直接使用 `Some` 和 `None`，不必加 `Option::` 前缀。
`Option<T>` 仍然只是普通枚举，`Some(T)` 和 `None` 仍然是 `Option<T>` 的变体。

`<T>` 是尚未介绍的 Rust 语法，表示<em>泛型类型形参(generic type parameter)</em>；
第 10 章会详细介绍泛型。目前只需知道，`<T>` 表示 `Option` 的 `Some` 变体可以
保存任意类型的一项数据，而每种代替 `T` 的具体类型都会产生不同的整体
`Option<T>` 类型。下面用 `Option` 值保存数值类型和字符类型：

```rust
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

`some_number` 的类型是 `Option<i32>`，`some_char` 的类型是 `Option<char>`，二者
不同。由于为 `Some` 变体指定了值，Rust 能够推断这些类型。对于 `absent_number`，
必须标注整个 `Option` 类型：只看到 `None`，编译器无法推断对应 `Some` 变体将
保存的类型。这里告诉 Rust，`absent_number` 的类型是 `Option<i32>`。

得到 `Some` 时，知道其中存在一个值，并由 `Some` 保存。得到 `None` 时，从某种
意义上说它与 null 相同：没有有效值。那么 `Option<T>` 为何比 null 更好？

简而言之，因为 `Option<T>` 与 `T`（`T` 可以是任意类型）是不同类型，编译器不会
允许把 `Option<T>` 当作必然有效的值使用。例如，下面的代码试图把 `i8` 与
`Option<i8>` 相加，因此无法编译：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

运行这段代码会得到类似下面的错误：

```console
{{#include ../../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

信息很多！实际上，它表示 Rust 不知道怎样将 `i8` 和 `Option<i8>` 相加，因为二者
类型不同。在 Rust 中拥有 `i8` 这类值时，编译器会确保它始终有效，可以放心使用，
无须预先检查 null。只有使用 `Option<i8>`（或其他值类型的 `Option`）时，才需要
考虑可能没有值；编译器会确保在使用前处理这种情况。

换句话说，必须先把 `Option<T>` 转换为 `T`，才能执行 `T` 的操作。总体而言，这
有助于发现最常见的 null 问题之一：错误地假定某个值不是 null。

消除错误假设非空值的风险，可以增强对代码的信心。如果某个值可能为空，必须显式
选择将其类型设为 `Option<T>`；使用时又必须显式处理值为空的情况。凡是类型不是
`Option<T>` 的值，都<em>可以</em>安全地假定它不为空。这是 Rust 有意作出的设计选择，
用来限制 null 的普遍影响并提高代码安全性。

拥有 `Option<T>` 后，怎样从 `Some` 变体中取出 `T` 值来使用？`Option<T>` 提供
大量适用于不同场景的方法，可以查看[它的文档][docs]<!-- ignore -->。熟悉这些方法
会对 Rust 学习之旅极有帮助。

一般而言，使用 `Option<T>` 值时，需要用代码处理每个变体。一部分代码只在值为
`Some(T)` 时运行，并可使用内部的 `T`；另一部分只在值为 `None` 时运行，而且没有
可用的 `T`。`match` 表达式正是一种与枚举配合完成此事的控制流结构：它根据枚举
变体运行不同代码，而代码可以使用匹配值内部的数据。

[IpAddr]: https://doc.rust-lang.org/std/net/enum.IpAddr.html
[option]: https://doc.rust-lang.org/std/option/enum.Option.html
[docs]: https://doc.rust-lang.org/std/option/enum.Option.html
