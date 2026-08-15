<!-- Old headings. Do not remove or links may break. -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>
<a id="treating-smart-pointers-like-regular-references-with-deref"></a>

<a id="treating-smart-pointers-like-regular-references"></a>

## 像普通引用一样使用智能指针

实现 `Deref` trait 可以自定义<em>解引用运算符(dereference operator)</em> `*` 的行为（不要将它与乘法运算符或 glob 运算符混淆）。如果以一种让智能指针能被当作普通引用使用的方式实现 `Deref`，就能编写操作引用的代码，并将这些代码用于智能指针。

我们先来看看解引用运算符如何作用于普通引用。然后，我们会尝试定义一个行为类似 `Box<T>` 的自定义类型，并了解为什么解引用运算符不能像处理引用那样处理这个新类型。接着会探讨实现 `Deref` trait 如何让智能指针以类似引用的方式工作。最后，我们会了解 Rust 的<em>解引用强制转换(deref coercion)</em>功能，以及它如何让我们既能使用引用也能使用智能指针。

<!-- Old headings. Do not remove or links may break. -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
<a id="following-the-pointer-to-the-value"></a>

### 沿着引用访问值

普通引用是一种指针。你可以把指针想象成一个箭头，指向存储在其他位置的值。在示例 15-6 中，我们创建了一个指向 `i32` 值的引用，然后使用解引用运算符沿着引用访问该值。

<Listing number="15-6" file-name="src/main.rs" caption="使用解引用运算符沿着引用访问一个 `i32` 值">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

变量 `x` 保存 `i32` 值 `5`。我们令 `y` 等于对 `x` 的引用。可以断言 `x` 等于 `5`。然而，如果想对 `y` 中的值进行断言，就必须使用 `*y` 沿着引用访问它指向的值（因此称为“解引用”），这样编译器才能比较实际的值。解引用 `y` 后，我们便能访问 `y` 所指向的整数值，并将其与 `5` 比较。

如果改为编写 `assert_eq!(5, y);`，就会得到以下编译错误：

```console
{{#include ../../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

数字和指向数字的引用类型不同，不能相互比较。必须使用解引用运算符沿着引用访问它所指向的值。

### 像引用一样使用 `Box<T>`

可以改写示例 15-6，用 `Box<T>` 代替引用；示例 15-7 中作用于 `Box<T>` 的解引用运算符，与示例 15-6 中作用于引用的解引用运算符工作方式相同。

<Listing number="15-7" file-name="src/main.rs" caption="对 `Box<i32>` 使用解引用运算符">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

示例 15-7 与示例 15-6 的主要区别是：这里将 `y` 设为一个箱子实例，它指向 `x` 的值的副本，而不是一个指向 `x` 值的引用。在最后一个断言中，可以像 `y` 是引用时那样，使用解引用运算符沿着箱子的指针访问值。接下来，我们将定义自己的箱子类型，以探究 `Box<T>` 有何特别之处，使我们能够对它使用解引用运算符。

### 定义自己的智能指针

我们来构建一个类似于标准库所提供 `Box<T>` 的包装类型，亲自看看智能指针类型在默认情况下与引用有何不同。然后，再了解如何为它添加使用解引用运算符的能力。

> 注意：我们即将构建的 `MyBox<T>` 类型与真正的 `Box<T>` 有一个重大区别：这个版本不会把数据存储在堆上。本例关注的是 `Deref`，所以数据实际存储在哪里不如其类似指针的行为重要。

`Box<T>` 类型最终被定义为一个包含单个元素的元组结构体，因此示例 15-8 也以相同方式定义 `MyBox<T>` 类型。我们还会定义一个 `new` 函数，与 `Box<T>` 上定义的 `new` 函数相对应。

<Listing number="15-8" file-name="src/main.rs" caption="定义 `MyBox<T>` 类型">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

我们定义了一个名为 `MyBox` 的结构体，并声明泛型参数 `T`，因为希望这个类型能够保存任意类型的值。`MyBox` 类型是一个包含单个 `T` 类型元素的元组结构体。`MyBox::new` 函数接收一个 `T` 类型的参数，并返回一个保存了传入值的 `MyBox` 实例。

现在尝试把示例 15-7 中的 `main` 函数添加到示例 15-8，并将其改为使用我们定义的 `MyBox<T>` 类型，而不是 `Box<T>`。示例 15-9 中的代码无法编译，因为 Rust 不知道如何解引用 `MyBox`。

<Listing number="15-9" file-name="src/main.rs" caption="尝试以使用引用和 `Box<T>` 的相同方式使用 `MyBox<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

由此产生的编译错误如下：

```console
{{#include ../../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

`MyBox<T>` 类型无法被解引用，因为我们还没有为这个类型实现该能力。为了能够使用 `*` 运算符解引用，我们要实现 `Deref` trait。

<!-- Old headings. Do not remove or links may break. -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### 实现 `Deref` Trait

正如第 10 章[“为类型实现 Trait”][impl-trait]<!-- ignore -->中所讨论的，要实现一个 trait，就需要为该 trait 的必需方法提供实现。标准库提供的 `Deref` trait 要求实现一个名为 `deref` 的方法；它借用 `self` 并返回对内部数据的引用。示例 15-10 包含了要添加到 `MyBox<T>` 定义中的 `Deref` 实现。

<Listing number="15-10" file-name="src/main.rs" caption="为 `MyBox<T>` 实现 `Deref`">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

`type Target = T;` 语法定义了供 `Deref` trait 使用的关联类型。关联类型是声明泛型参数的一种略有不同的方式，不过现在不必操心它；我们将在第 20 章更详细地介绍。

我们用 `&self.0` 填充 `deref` 方法的方法体，让 `deref` 返回对我们希望通过 `*` 运算符访问的值的引用。回想一下第 5 章[“使用元组结构体创建不同的类型”][tuple-structs]<!-- ignore -->一节，`.0` 会访问元组结构体中的第一个值。现在，示例 15-9 中对 `MyBox<T>` 值调用 `*` 的 `main` 函数可以编译，并且断言也能通过！

如果没有 `Deref` trait，编译器只能解引用 `&` 引用。`deref` 方法让编译器能够取得任何实现了 `Deref` 的类型的值，并调用其 `deref` 方法来获得一个编译器知道如何解引用的引用。

在示例 15-9 中输入 `*y` 时，Rust 在幕后实际运行的是以下代码：

```rust,ignore
*(y.deref())
```

Rust 将 `*` 运算符替换为一次 `deref` 方法调用，再进行一次普通解引用，这样我们就不必考虑是否需要调用 `deref` 方法。Rust 的这一功能让代码无论面对普通引用还是实现了 `Deref` 的类型，都能以完全相同的方式工作。

`deref` 方法之所以返回对值的引用，并且 `*(y.deref())` 中括号外的普通解引用仍然必不可少，与所有权系统有关。如果 `deref` 方法直接返回值，而不是对值的引用，这个值就会被移出 `self`。在这个例子以及使用解引用运算符的大多数情况下，我们都不希望取得 `MyBox<T>` 内部值的所有权。

请注意，每当代码中使用一次 `*`，该运算符只会被替换为一次 `deref` 方法调用和一次 `*` 运算符调用。由于对 `*` 运算符的替换不会无限递归，最终会得到 `i32` 类型的数据，与示例 15-9 中 `assert_eq!` 里的 `5` 相匹配。

<!-- Old headings. Do not remove or links may break. -->

<a id="implicit-deref-coercions-with-functions-and-methods"></a>
<a id="using-deref-coercions-in-functions-and-methods"></a>

### 在函数和方法中使用解引用强制转换

<em>解引用强制转换(deref coercion)</em>会把指向某个实现了 `Deref` trait 的类型的引用，转换为指向另一类型的引用。例如，解引用强制转换可以把 `&String` 转换为 `&str`，因为 `String` 实现 `Deref` trait 时会返回 `&str`。解引用强制转换是 Rust 对函数和方法的实参执行的一项便利功能，仅适用于实现了 `Deref` trait 的类型。当把对某种类型值的引用作为实参传给函数或方法，而它与函数或方法定义中的形参类型不匹配时，这种转换会自动发生。通过一系列 `deref` 方法调用，Rust 会把我们提供的类型转换为形参所需的类型。

Rust 加入解引用强制转换，是为了让程序员在编写函数和方法调用时，不必使用 `&` 和 `*` 添加那么多显式引用与解引用。解引用强制转换还让我们能够编写更多既适用于引用、也适用于智能指针的代码。

为了观察解引用强制转换的实际效果，我们将使用示例 15-8 中定义的 `MyBox<T>` 类型，以及示例 15-10 中添加的 `Deref` 实现。示例 15-11 展示了一个形参为字符串切片的函数定义。

<Listing number="15-11" file-name="src/main.rs" caption="形参 `name` 的类型为 `&str` 的 `hello` 函数">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

可以用字符串切片作为实参调用 `hello` 函数，例如 `hello("Rust");`。解引用强制转换也使我们能够用一个指向 `MyBox<String>` 类型值的引用来调用 `hello`，如示例 15-12 所示。

<Listing number="15-12" file-name="src/main.rs" caption="用指向 `MyBox<String>` 值的引用调用 `hello`；由于解引用强制转换，这段代码可以工作">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

这里使用实参 `&m` 调用 `hello` 函数，它是一个指向 `MyBox<String>` 值的引用。由于我们在示例 15-10 中为 `MyBox<T>` 实现了 `Deref` trait，Rust 可以通过调用 `deref` 将 `&MyBox<String>` 转换为 `&String`。标准库为 `String` 提供了一个返回字符串切片的 `Deref` 实现，这一点记录在 `Deref` 的 API 文档中。Rust 再次调用 `deref`，将 `&String` 转换为 `&str`，从而与 `hello` 函数的定义相匹配。

如果 Rust 没有实现解引用强制转换，为了用 `&MyBox<String>` 类型的值调用 `hello`，就必须编写示例 15-13 中的代码，而不是示例 15-12 中的代码。

<Listing number="15-13" file-name="src/main.rs" caption="假如 Rust 没有解引用强制转换，我们就必须编写这样的代码">

```rust
{{#rustdoc_include ../../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

`(*m)` 将 `MyBox<String>` 解引用为一个 `String`。随后，`&` 和 `[..]` 获取一个涵盖整个 `String` 的字符串切片，以匹配 `hello` 的签名。如果没有解引用强制转换，这段包含许多符号的代码会更难阅读、编写和理解。解引用强制转换让 Rust 自动为我们处理这些转换。

当相关类型定义了 `Deref` trait 时，Rust 会分析类型，并根据需要多次使用 `Deref::deref`，直到得到与形参类型匹配的引用。需要插入多少次 `Deref::deref` 会在编译期确定，因此利用解引用强制转换不会产生运行时开销！

<!-- Old headings. Do not remove or links may break. -->

<a id="how-deref-coercion-interacts-with-mutability"></a>

### 处理可变引用的解引用强制转换

正如使用 `Deref` trait 可以重载不可变引用上的 `*` 运算符一样，也可以使用 `DerefMut` trait 重载可变引用上的 `*` 运算符。

Rust 会在发现以下三种类型与 trait 实现的组合时进行解引用强制转换：

1. 当 `T: Deref<Target=U>` 时，从 `&T` 转换为 `&U`
2. 当 `T: DerefMut<Target=U>` 时，从 `&mut T` 转换为 `&mut U`
3. 当 `T: Deref<Target=U>` 时，从 `&mut T` 转换为 `&U`

前两种情况除了第二种涉及可变性以外完全相同。第一种表示：如果有一个 `&T`，且 `T` 实现了到某个类型 `U` 的 `Deref`，就可以透明地得到 `&U`。第二种表示，可变引用也会发生同样的解引用强制转换。

第三种情况更微妙：Rust 还会把可变引用强制转换为不可变引用。但反过来<em>不</em>可行：不可变引用永远不会被强制转换为可变引用。根据借用规则，如果有一个可变引用，它就必须是指向该数据的唯一引用（否则程序无法编译）。把一个可变引用转换为一个不可变引用绝不会违反借用规则。要把不可变引用转换为可变引用，则要求最初的不可变引用是指向该数据的唯一不可变引用，但借用规则并不保证这一点。因此，Rust 不能假定将不可变引用转换为可变引用是可行的。

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
