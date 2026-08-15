## 泛型数据类型

我们使用泛型为函数签名、结构体等条目创建定义，之后便可将其用于许多不同的具体数据类型。先看看如何使用泛型定义函数、结构体、枚举和方法，再讨论泛型如何影响代码性能。

### 在函数定义中

定义使用泛型的函数时，我们会把泛型放在函数签名中通常指定形参和返回值数据类型的位置。这样做可以让代码更加灵活，为函数调用者提供更多功能，同时避免代码重复。

继续使用 `largest` 函数。示例 10-4 展示了两个都能找出切片中最大值的函数。随后，我们会把它们合并为一个使用泛型的函数。

<Listing number="10-4" file-name="src/main.rs" caption="两个仅名称和签名中的类型不同的函数">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

`largest_i32` 就是示例 10-3 中提取出来、用于找出切片中最大 `i32` 的函数。`largest_char` 函数会找出切片中最大的 `char`。两个函数体的代码相同，因此让我们在单个函数中引入一个泛型类型形参来消除重复。

为了让新函数中的类型参数化，需要为类型形参命名，就像为函数的值形参命名一样。任何标识符都可以用作类型形参名。不过，我们会使用 `T`，因为按照约定，Rust 中的类型形参名很短，通常只有一个字母，而且 Rust 的类型命名约定是大驼峰命名法。`T` 是 <em>type</em> 的缩写，也是大多数 Rust 程序员的默认选择。

在函数体中使用形参时，必须在签名中声明形参名，让编译器知道该名称的含义。类似地，在函数签名中使用类型形参名时，也必须先声明它。为了定义泛型 `largest` 函数，我们把类型名称声明放在函数名与形参列表之间的尖括号 `<>` 内，如下所示：

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

这个定义可以读作：“函数 `largest` 对某种类型 `T` 是泛型的。”该函数有一个名为 `list` 的形参，它是由 `T` 类型值组成的切片。`largest` 函数会返回对相同 `T` 类型值的引用。

示例 10-5 展示了在签名中使用泛型数据类型合并后的 `largest` 函数定义。示例还展示了如何分别使用 `i32` 值切片或 `char` 值切片调用该函数。请注意，这段代码暂时还无法编译。

<Listing number="10-5" file-name="src/main.rs" caption="使用泛型类型形参的 `largest` 函数；目前还无法编译">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

如果现在编译这段代码，会得到以下错误：

```console
{{#include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

帮助文本提到了 `std::cmp::PartialOrd`，它是一个特征；下一节会讨论特征。现在只需知道，这个错误表示 `largest` 的函数体无法适用于 `T` 可能代表的所有类型。由于我们希望在函数体中比较 `T` 类型的值，只能使用值可以排序的类型。为了支持比较，标准库提供了可以在类型上实现的 `std::cmp::PartialOrd` 特征（有关该特征的更多信息，请参阅附录 C）。要修复示例 10-5，可以遵照帮助文本的建议，把 `T` 的有效类型限制为仅实现了 `PartialOrd` 的类型。随后示例便可以编译，因为标准库为 `i32` 和 `char` 都实现了 `PartialOrd`。

### 在结构体定义中

还可以使用 `<>` 语法定义结构体，使一个或多个字段使用泛型类型形参。示例 10-6 定义了一个 `Point<T>` 结构体，用于存放任意类型的 `x` 和 `y` 坐标值。

<Listing number="10-6" file-name="src/main.rs" caption="存放 `T` 类型的 `x` 和 `y` 值的 `Point<T>` 结构体">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

在结构体定义中使用泛型的语法与函数定义类似。首先，紧跟结构体名称之后，在尖括号中声明类型形参名。然后，在结构体定义中原本应指定具体数据类型的位置使用泛型类型。

请注意，由于定义 `Point<T>` 时只使用了一个泛型类型，该定义表示 `Point<T>` 结构体对某种类型 `T` 是泛型的，而且无论该类型是什么，字段 `x` 和 `y` 都是<em>相同的</em>类型。如果创建一个字段值类型不同的 `Point<T>` 实例，如示例 10-7 所示，代码将无法编译。

<Listing number="10-7" file-name="src/main.rs" caption="字段 `x` 和 `y` 必须具有相同类型，因为两者使用同一个泛型数据类型 `T`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

在这个例子中，把整数值 `5` 赋给 `x` 时，我们告诉编译器，这个 `Point<T>` 实例的泛型类型 `T` 是整数。接着为 `y` 指定 `4.0` 时，由于 `y` 被定义为与 `x` 具有相同类型，就会出现如下类型不匹配错误：

```console
{{#include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

要定义一个 `x` 和 `y` 都是泛型但类型可以不同的 `Point` 结构体，可以使用多个泛型类型形参。例如，在示例 10-8 中，我们把 `Point` 的定义改为对类型 `T` 和 `U` 都是泛型的，其中 `x` 的类型为 `T`，`y` 的类型为 `U`。

<Listing number="10-8" file-name="src/main.rs" caption="对两种类型使用泛型的 `Point<T, U>`，使 `x` 和 `y` 可以是不同类型的值">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

现在，示例中的所有 `Point` 实例都成立了！定义中可以使用任意多个泛型类型形参，但超过少数几个就会使代码难以阅读。如果发现代码需要大量泛型类型，可能意味着需要把代码重构为更小的部分。

### 在枚举定义中

与结构体一样，可以定义让变体存放泛型数据类型的枚举。让我们再次看看标准库提供、并在第 6 章中使用过的 `Option<T>` 枚举：

```rust
enum Option<T> {
    Some(T),
    None,
}
```

现在，这个定义应该更容易理解了。如你所见，`Option<T>` 枚举对类型 `T` 是泛型的，并有两个变体：`Some` 存放一个 `T` 类型的值，`None` 变体不存放任何值。通过使用 `Option<T>` 枚举，我们可以表达可选值这一抽象概念；又因为 `Option<T>` 是泛型的，无论可选值是什么类型，都能使用这种抽象。

枚举也可以使用多个泛型类型。第 9 章中使用的 `Result` 枚举定义就是一个例子：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result` 枚举对 `T` 和 `E` 两种类型是泛型的，并有两个变体：`Ok` 存放 `T` 类型的值，`Err` 存放 `E` 类型的值。对于任何可能成功（返回某种 `T` 类型的值）或失败（返回某种 `E` 类型的错误）的操作，这个定义都让使用 `Result` 枚举变得十分方便。事实上，示例 9-3 中打开文件时就使用了它：文件成功打开时，`T` 由类型 `std::fs::File` 填充；打开文件发生问题时，`E` 由类型 `std::io::Error` 填充。

如果发现代码中有多个结构体或枚举定义，彼此间的区别仅在于所存放值的类型，就可以使用泛型类型来避免重复。

### 在方法定义中

我们可以像第 5 章那样为结构体和枚举实现方法，也可以在方法定义中使用泛型类型。示例 10-9 展示了示例 10-6 中定义的 `Point<T>` 结构体，并为它实现了名为 `x` 的方法。

<Listing number="10-9" file-name="src/main.rs" caption="在 `Point<T>` 结构体上实现名为 `x` 的方法，返回对 `T` 类型字段 `x` 的引用">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

这里，我们在 `Point<T>` 上定义了名为 `x` 的方法，它返回对字段 `x` 中数据的引用。

请注意，必须紧跟 `impl` 之后声明 `T`，这样才能使用 `T` 指定我们要为 `Point<T>` 类型实现方法。通过在 `impl` 之后把 `T` 声明为泛型类型，Rust 可以识别出 `Point` 尖括号中的类型是泛型类型，而不是具体类型。这里可以为泛型形参选择与结构体定义中声明的泛型形参不同的名称，但按照惯例会使用相同名称。如果在声明了泛型类型的 `impl` 中编写方法，无论最终用什么具体类型替代泛型类型，该方法都会在该类型的所有实例上定义。

定义类型上的方法时，还可以为泛型类型指定约束。例如，可以只为 `Point<f32>` 实例实现方法，而不是为具有任意泛型类型的 `Point<T>` 实例实现方法。在示例 10-10 中，我们使用具体类型 `f32`，这意味着无需在 `impl` 之后声明任何类型。

<Listing number="10-10" file-name="src/main.rs" caption="仅适用于泛型类型形参 `T` 被某个特定具体类型替代的结构体的 `impl` 块">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

这段代码表示 `Point<f32>` 类型会有 `distance_from_origin` 方法，而 `T` 不是 `f32` 的其他 `Point<T>` 实例不会定义该方法。这个方法会测量点与坐标 (0.0, 0.0) 处的点之间的距离，并使用只适用于浮点类型的数学运算。

结构体定义中的泛型类型形参不一定与同一结构体的方法签名所使用的形参相同。示例 10-11 为 `Point` 结构体使用泛型类型 `X1` 和 `Y1`，为 `mixup` 方法签名使用 `X2` 和 `Y2`，以使例子更加清晰。该方法会创建一个新的 `Point` 实例，其 `x` 值来自 `self` 的 `Point`（类型为 `X1`），`y` 值来自传入的 `Point`（类型为 `Y2`）。

<Listing number="10-11" file-name="src/main.rs" caption="使用不同于结构体定义中泛型类型的方法">

```rust
{{#rustdoc_include ../../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

在 `main` 中，我们定义了一个 `Point`，其 `x` 是值为 `5` 的 `i32`，`y` 是值为 `10.4` 的 `f64`。变量 `p2` 是一个 `Point` 结构体，其 `x` 是值为 `"Hello"` 的字符串切片，`y` 是值为 `c` 的 `char`。以 `p2` 为实参对 `p1` 调用 `mixup` 会得到 `p3`；因为 `x` 来自 `p1`，所以 `p3` 的 `x` 是 `i32`；因为 `y` 来自 `p2`，所以 `p3` 的 `y` 是 `char`。`println!` 宏调用会打印 `p3.x = 5, p3.y = c`。

这个例子旨在展示一种情形：一些泛型形参随 `impl` 声明，另一些随方法定义声明。这里，泛型形参 `X1` 和 `Y1` 在 `impl` 之后声明，因为它们与结构体定义相关；泛型形参 `X2` 和 `Y2` 在 `fn mixup` 之后声明，因为它们只与该方法相关。

<a id="performance-of-code-using-generics"></a>

### 使用泛型的代码性能

你可能会想，使用泛型类型形参是否会产生运行时开销。好消息是，与使用具体类型相比，使用泛型类型不会让程序运行得更慢。

Rust 通过在编译时对使用泛型的代码执行<em>单态化(monomorphization)</em>来做到这一点。单态化是在编译时填入所使用的具体类型，把泛型代码转变为特定代码的过程。在这个过程中，编译器所做的事情与我们创建示例 10-5 中泛型函数时采取的步骤相反：编译器查看调用泛型代码的所有位置，并针对调用时使用的具体类型生成代码。

让我们使用标准库的泛型 `Option<T>` 枚举，看看这个过程如何运作：

```rust
let integer = Some(5);
let float = Some(5.0);
```

Rust 编译这段代码时会执行单态化。在此过程中，编译器读取 `Option<T>` 实例中使用的值，识别出两种 `Option<T>`：一种是 `i32`，另一种是 `f64`。于是，它会把 `Option<T>` 的泛型定义展开为分别针对 `i32` 和 `f64` 的两个定义，从而用具体定义取代泛型定义。

单态化后的代码版本与下面类似（为了说明问题，我们使用的名称与编译器实际使用的名称不同）：

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

泛型 `Option<T>` 被编译器创建的具体定义所替代。由于 Rust 会把泛型代码编译为在每个实例中都指定类型的代码，使用泛型无需付出运行时成本。代码运行时的行为，就像我们手动复制了每个定义一样。单态化过程让 Rust 的泛型在运行时极为高效。
