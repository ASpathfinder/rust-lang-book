## 不安全 Rust

到目前为止，我们讨论的所有代码都在编译时受到 Rust 内存安全保证的约束。不过，Rust 内部还隐藏着第二种语言，它不强制执行这些内存安全保证：这就是<em>不安全 Rust(unsafe Rust)</em>。它的工作方式与普通 Rust 相同，但赋予了我们一些额外的超能力。

不安全 Rust 之所以存在，是因为静态分析天生是保守的。当编译器尝试判断代码是否维护了各项保证时，拒绝一些有效程序总比接受一些无效程序更好。即使代码<em>可能</em>没有问题，只要 Rust 编译器掌握的信息不足以让它确信这一点，就会拒绝该代码。在这种情况下，你可以使用不安全代码告诉编译器：“相信我，我知道自己在做什么。”但请注意，使用不安全 Rust 的风险由你自行承担：如果错误地使用不安全代码，可能会因内存不安全而出现问题，例如解引用空指针。

Rust 拥有这样一个不安全的另一面还有一个原因：底层计算机硬件本身就是不安全的。如果 Rust 不允许执行不安全操作，有些任务便无法完成。Rust 需要允许你开展低级系统编程，例如直接与操作系统交互，甚至编写自己的操作系统。支持低级系统编程正是这门语言的目标之一。下面来探索不安全 Rust 能做什么，以及如何去做。

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

### 施展不安全超能力

要切换到不安全 Rust，请使用 `unsafe` 关键字并开启一个容纳不安全代码的新代码块。在不安全 Rust 中可以执行五种安全 Rust 无法执行的操作，我们把它们称为<em>不安全超能力(unsafe superpowers)</em>。这些超能力包括：

1. 解引用原始指针。
1. 调用不安全函数或方法。
1. 访问或修改可变静态变量。
1. 实现不安全 trait。
1. 访问 `union` 的字段。

需要明确的是，`unsafe` 并不会关闭借用检查器，也不会禁用 Rust 的其他安全检查：如果在不安全代码中使用引用，它仍会受到检查。`unsafe` 关键字只是让你能够使用这五种不由编译器检查其内存安全性的功能。即使在不安全块中，你仍会获得一定程度的安全保障。

此外，`unsafe` 并不表示块中的代码必然危险，或一定会导致内存安全问题：它表达的意图是，作为程序员，你将确保 `unsafe` 块中的代码以有效方式访问内存。

人难免会犯错，但要求这五种不安全操作必须放在标有 `unsafe` 的代码块中，意味着你知道任何与内存安全相关的错误必定位于某个 `unsafe` 块内。让 `unsafe` 块保持短小；日后排查内存错误时，你会庆幸这样做。

为了尽可能隔离不安全代码，最佳做法是将其封装在<em>安全抽象(safe abstraction)</em>中并提供安全 API。稍后讨论不安全函数和方法时，我们还会介绍这一点。标准库的一部分就是以经过审计的不安全代码为基础实现的安全抽象。把不安全代码包装在安全抽象中，可以防止 `unsafe` 的使用扩散到你或用户需要使用该功能的所有位置，因为使用安全抽象本身是安全的。

下面依次看看这五种不安全超能力。我们还会考察一些为不安全代码提供安全接口的抽象。

### 解引用原始指针

第 4 章的[“悬垂引用”][dangling-references]<!-- ignore -->一节提到，编译器会确保引用始终有效。不安全 Rust 提供了两种类似引用的新类型，称为<em>原始指针(raw pointer)</em>。与引用一样，原始指针可以不可变或可变，分别写作 `*const T` 和 `*mut T`。这里的星号不是解引用运算符，而是类型名称的一部分。在原始指针的语境中，<em>不可变</em>表示指针解引用后不能直接对其赋值。

与引用和智能指针不同，原始指针：

- 可以忽略借用规则，同时拥有指向同一位置的不可变指针和可变指针，或多个可变指针
- 不保证指向有效内存
- 可以为空
- 不实现任何自动清理

选择不让 Rust 强制执行这些保证，就可以用确定的安全性换取更高性能，或者与 Rust 保证不适用的其他语言或硬件交互。

示例 20-1 展示了如何创建一个不可变原始指针和一个可变原始指针。

<Listing number="20-1" caption="使用原始借用运算符创建原始指针">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

请注意，这段代码中没有使用 `unsafe` 关键字。我们可以在安全代码中创建原始指针，只是不能在 `unsafe` 块之外解引用原始指针，稍后你会看到这一点。

我们使用原始借用运算符创建了原始指针：`&raw const num` 创建一个 `*const i32` 类型的不可变原始指针，`&raw mut num` 创建一个 `*mut i32` 类型的可变原始指针。因为它们直接由局部变量创建，所以我们知道这两个特定原始指针是有效的；但不能对任意原始指针都作出这种假设。

为了说明这一点，接下来我们不使用原始借用运算符，而是用关键字 `as` 转换一个值，创建一个无法确定其有效性的原始指针。示例 20-2 展示了如何创建指向任意内存位置的原始指针。尝试使用任意内存会产生未定义行为：该地址可能有数据，也可能没有；编译器可能把代码优化到根本不访问内存；程序也可能因段错误而终止。通常没有充分理由编写这样的代码，尤其是在可以使用原始借用运算符的情况下，但这样做确实是可能的。

<Listing number="20-2" caption="创建指向任意内存地址的原始指针">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

回想一下，我们可以在安全代码中创建原始指针，却不能解引用原始指针来读取其指向的数据。在示例 20-3 中，我们对原始指针使用需要 `unsafe` 块的解引用运算符 `*`。

<Listing number="20-3" caption="在 `unsafe` 块中解引用原始指针">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

创建指针本身没有危害；只有在尝试访问它指向的值时，我们才可能遇到无效值。

还请注意，在示例 20-1 和 20-3 中，我们创建的 `*const i32` 和 `*mut i32` 原始指针都指向存储 `num` 的同一内存位置。如果改为尝试创建 `num` 的不可变引用和可变引用，代码将无法编译，因为 Rust 的所有权规则不允许可变引用与任何不可变引用同时存在。使用原始指针，我们可以创建指向同一位置的可变指针和不可变指针，并通过可变指针修改数据，这可能造成数据竞争。务必谨慎！

既然有这么多危险，为什么还要使用原始指针呢？一个主要用途是与 C 代码交互，下一节将对此进行介绍。另一个用途是构建借用检查器无法理解的安全抽象。我们先介绍不安全函数，然后考察一个使用不安全代码的安全抽象示例。

### 调用不安全函数或方法

可以在不安全块中执行的第二类操作是调用不安全函数。不安全函数和方法看起来与普通函数和方法完全相同，只是在定义的其余部分之前多了一个 `unsafe`。此处的 `unsafe` 关键字表示，该函数具有调用时必须维护的要求，因为 Rust 无法保证我们已经满足这些要求。在 `unsafe` 块中调用不安全函数，意味着我们声明自己已阅读该函数的文档、理解如何正确使用它，并负责维护函数的<em>契约(contract)</em>。

下面有一个名为 `dangerous`、函数体什么也不做的不安全函数：

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

我们必须在单独的 `unsafe` 块中调用 `dangerous` 函数。如果尝试在没有 `unsafe` 块的情况下调用 `dangerous`，会得到错误：

```console
{{#include ../../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

使用 `unsafe` 块，就是向 Rust 声明：我们已阅读函数文档，理解如何正确使用它，并确认自己正在履行该函数的契约。

即使在 `unsafe` 函数体内执行不安全操作，也仍然需要像普通函数中那样使用 `unsafe` 块；如果忘记，编译器会发出警告。这有助于我们尽可能缩小 `unsafe` 块，因为整个函数体中未必处处都需要不安全操作。

#### 在不安全代码之上创建安全抽象

函数包含不安全代码，并不意味着必须把整个函数标记为不安全。事实上，把不安全代码包装在安全函数中是一种常见抽象。例如，我们来研究标准库中需要一些不安全代码的 `split_at_mut` 函数，并探索如何实现它。这个安全方法定义在可变切片上：它接收一个切片，并在实参指定的索引处将其一分为二。示例 20-4 展示了如何使用 `split_at_mut`。

<Listing number="20-4" caption="使用安全的 `split_at_mut` 函数">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

我们无法只使用安全 Rust 来实现这个函数。一种尝试可能如示例 20-5 所示，但它无法编译。为简单起见，我们将 `split_at_mut` 实现为函数而不是方法，而且只处理 `i32` 值的切片，而不是泛型类型 `T`。

<Listing number="20-5" caption="尝试仅使用安全 Rust 实现 `split_at_mut`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

这个函数首先取得切片的总长度。然后通过检查作为形参传入的索引是否小于或等于长度，断言该索引位于切片范围内。该断言意味着，如果传入一个大于切片长度的分割索引，函数会在尝试使用该索引之前 panic。

接着，我们用元组返回两个可变切片：一个从原切片开头到 `mid` 索引，另一个从 `mid` 到末尾。

尝试编译示例 20-5 中的代码时，会得到错误：

```console
{{#include ../../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Rust 的借用检查器无法理解我们借用的是切片的不同部分；它只知道我们对同一个切片借用了两次。借用切片的不同部分从根本上说没有问题，因为两个切片并不重叠，但 Rust 还不够聪明，无法得知这一点。当我们知道代码没问题而 Rust 不知道时，就该使用不安全代码了。

示例 20-6 展示了如何使用 `unsafe` 块、原始指针和若干不安全函数调用，让 `split_at_mut` 的实现能够工作。

<Listing number="20-6" caption="在 `split_at_mut` 函数的实现中使用不安全代码">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

回想第 4 章[“切片类型”][the-slice-type]<!-- ignore -->一节，切片由指向某些数据的指针和切片长度组成。我们使用 `len` 方法获取切片长度，并使用 `as_mut_ptr` 方法访问切片的原始指针。在这里，因为我们拥有一个 `i32` 值的可变切片，所以 `as_mut_ptr` 返回 `*mut i32` 类型的原始指针，并将其存储在变量 `ptr` 中。

我们保留了确保 `mid` 索引位于切片范围内的断言。接下来是不安全代码：`slice::from_raw_parts_mut` 函数接收一个原始指针和一个长度，并创建切片。我们用它创建一个从 `ptr` 开始、长度为 `mid` 项的切片。然后以 `mid` 为实参调用 `ptr` 的 `add` 方法，取得从 `mid` 开始的原始指针，再以该指针和 `mid` 之后剩余项数作为长度创建另一个切片。

`slice::from_raw_parts_mut` 函数是不安全的，因为它接收原始指针，必须相信该指针有效。原始指针上的 `add` 方法同样不安全，因为它必须相信偏移后的位置也是有效指针。因此，我们必须用 `unsafe` 块包围对 `slice::from_raw_parts_mut` 和 `add` 的调用，才能调用它们。查看代码并加入 `mid` 必须小于或等于 `len` 的断言后，我们能够判断 `unsafe` 块中使用的所有原始指针都有效地指向切片内的数据。这是可以接受且恰当的 `unsafe` 用法。

请注意，我们不需要把最终的 `split_at_mut` 函数标记为 `unsafe`，而且可以从安全 Rust 中调用它。我们为不安全代码创建了安全抽象：函数实现以安全方式使用不安全代码，因为它只会从该函数能够访问的数据中创建有效指针。

相比之下，示例 20-7 中对 `slice::from_raw_parts_mut` 的使用很可能在使用切片时导致崩溃。这段代码取得一个任意内存位置，并创建长度为 10,000 项的切片。

<Listing number="20-7" caption="从任意内存位置创建切片">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

我们并不拥有这个任意位置的内存，也无法保证代码创建的切片包含有效的 `i32` 值。把 `values` 当作有效切片使用会导致<em>未定义行为(undefined behavior)</em>。

#### 使用 `extern` 函数调用外部代码

有时 Rust 代码可能需要与其他语言编写的代码交互。为此，Rust 提供了 `extern` 关键字，用来帮助创建和使用<em>外部函数接口(Foreign Function Interface, FFI)</em>；这是一种让编程语言定义函数并允许另一种（外部）编程语言调用这些函数的方式。

示例 20-8 展示了如何与 C 标准库中的 `abs` 函数建立集成。在 `extern` 块中声明的函数通常不能从 Rust 代码安全调用，因此 `extern` 块也必须标记为 `unsafe`。原因是其他语言并不强制执行 Rust 的规则和保证，Rust 也无法检查它们，所以确保安全的责任落在程序员身上。

<Listing number="20-8" file-name="src/main.rs" caption="声明并调用在另一种语言中定义的 `extern` 函数">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

在 `unsafe extern "C"` 块中，我们列出希望调用的、来自另一种语言的外部函数名称和签名。其中的 `"C"` 部分定义外部函数使用的<em>应用二进制接口(application binary interface, ABI)</em>：ABI 规定如何在汇编层面调用函数。`"C"` ABI 最为常见，它遵循 C 编程语言的 ABI。有关 Rust 支持的所有 ABI 的信息，请参阅 [Rust 参考手册][ABI]。

在 `unsafe extern` 块中声明的每个项都隐式为不安全。不过，有些 FFI 函数调用起来确实是安全的。例如，C 标准库的 `abs` 函数不涉及任何内存安全事项，而且我们知道可以用任意 `i32` 调用它。在这种情况下，可以使用 `safe` 关键字说明这个特定函数即使位于 `unsafe extern` 块中，也可以安全调用。做出这项更改后，调用它便不再需要 `unsafe` 块，如示例 20-9 所示。

<Listing number="20-9" file-name="src/main.rs" caption="在 `unsafe extern` 块中显式将函数标记为 `safe`，并安全地调用它">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

将函数标记为 `safe` 并不会天然让它变得安全！这更像是你向 Rust 作出的安全承诺。确保兑现这项承诺仍然是你的责任！

#### 从其他语言调用 Rust 函数

我们也可以用 `extern` 创建接口，让其他语言调用 Rust 函数。此时不创建完整的 `extern` 块，而是在相关函数的 `fn` 关键字之前加入 `extern` 关键字并指定所用 ABI。还需要添加 `#[unsafe(no_mangle)]` 注解，告诉 Rust 编译器不要混淆该函数的名称。<em>名称混淆(mangling)</em>是指编译器把我们为函数指定的名称改成另一个名称；新名称包含更多信息，可供编译过程的其他部分使用，但可读性较差。每种编程语言的编译器混淆名称的方式略有不同，因此若要让其他语言能够按名称找到 Rust 函数，必须禁用 Rust 编译器的名称混淆。这是不安全的，因为没有内置的名称混淆后，不同库之间可能发生名称冲突；所以我们有责任确保选定的名称可以安全地以未混淆形式导出。

下面的示例在把代码编译为共享库并从 C 中链接之后，使 C 代码能够访问 `call_from_c` 函数：

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

这种 `extern` 用法只要求在属性中使用 `unsafe`，不需要在 `extern` 块上使用。

### 访问或修改可变静态变量

本书还没有讨论过全局变量。Rust 确实支持全局变量，但它们可能与 Rust 的所有权规则产生问题。如果两个线程访问同一个可变全局变量，可能造成数据竞争。

在 Rust 中，全局变量称为<em>静态变量(static variable)</em>。示例 20-10 展示了一个声明并使用以字符串切片为值的静态变量的例子。

<Listing number="20-10" file-name="src/main.rs" caption="定义并使用不可变静态变量">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

静态变量类似于第 3 章[“声明常量”][constants]<!-- ignore -->一节讨论的常量。按照惯例，静态变量名称采用 `SCREAMING_SNAKE_CASE`。静态变量只能存储带 `'static` 生命周期的引用，这意味着 Rust 编译器可以推断出生命周期，而无需我们显式标注。访问不可变静态变量是安全的。

常量与不可变静态变量之间有一个细微区别：静态变量中的值在内存中具有固定地址。使用该值时总会访问同一份数据。另一方面，常量每次使用时都可以复制其数据。另一个区别是静态变量可以是可变的。访问和修改可变静态变量是<em>不安全的</em>。示例 20-11 展示了如何声明、访问和修改名为 `COUNTER` 的可变静态变量。

<Listing number="20-11" file-name="src/main.rs" caption="读取或写入可变静态变量是不安全的">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

与普通变量一样，我们用 `mut` 关键字指定可变性。任何读取或写入 `COUNTER` 的代码都必须位于 `unsafe` 块中。示例 20-11 的代码能够编译，并如预期打印 `COUNTER: 3`，因为它是单线程的。让多个线程访问 `COUNTER` 很可能造成数据竞争，因而属于未定义行为。因此，我们需要把整个函数标记为 `unsafe`，并记录安全限制，让调用函数的人知道哪些操作可以安全执行、哪些不能。

每当编写不安全函数时，惯例是写一条以 `SAFETY` 开头的注释，解释调用者需要怎样做才能安全地调用该函数。同样，每当执行不安全操作时，惯例是写一条以 `SAFETY` 开头的注释，解释安全规则是如何得到维护的。

此外，编译器默认会通过编译器 lint 拒绝任何创建可变静态变量引用的尝试。你必须添加 `#[allow(static_mut_refs)]` 注解来显式放弃该 lint 的保护，或者通过某个原始借用运算符创建的原始指针访问可变静态变量。这也包括隐式创建引用的情况，例如在本代码示例的 `println!` 中使用变量时。要求通过原始指针创建静态可变变量的引用，有助于让使用它们所需满足的安全要求更加明显。

对于全局可访问的可变数据，很难确保没有数据竞争，这就是 Rust 把可变静态变量视为不安全的原因。只要可能，最好使用第 16 章讨论的并发技术和线程安全的智能指针，让编译器检查不同线程是否安全地访问数据。

### 实现不安全 Trait

我们可以使用 `unsafe` 实现不安全 trait。当一个 trait 至少有一个方法包含编译器无法验证的<em>不变条件(invariant)</em>时，该 trait 就是不安全的。声明不安全 trait 时，在 `trait` 前添加 `unsafe` 关键字；实现它时也把实现标记为 `unsafe`，如示例 20-12 所示。

<Listing number="20-12" caption="定义并实现不安全 trait">

```rust
{{#rustdoc_include ../../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

使用 `unsafe impl`，就是承诺我们会维护编译器无法验证的不变条件。

例如，回想第 16 章[“使用 `Send` 和 `Sync` 实现可扩展并发”][send-and-sync]<!-- ignore -->一节讨论的 `Send` 和 `Sync` 标记 trait：如果我们的类型完全由实现了 `Send` 和 `Sync` 的其他类型组成，编译器会自动为其实现这些 trait。如果我们实现的类型包含未实现 `Send` 或 `Sync` 的类型（例如原始指针），但又想把该类型标记为 `Send` 或 `Sync`，就必须使用 `unsafe`。Rust 无法验证我们的类型是否维护了能够安全地在线程间发送或由多个线程访问的保证；因此，我们需要手动执行这些检查，并用 `unsafe` 表明这一点。

### 访问联合体的字段

最后一种只有使用 `unsafe` 才能执行的操作，是访问联合体的字段。<em>联合体(union)</em>与 `struct` 类似，但每个特定实例在同一时刻只使用一个已声明字段。联合体主要用于与 C 代码中的联合体交互。访问联合体字段是不安全的，因为 Rust 无法保证联合体实例当前存储的数据属于哪种类型。有关联合体的更多信息，请参阅 [Rust 参考手册][unions]。

### 使用 Miri 检查不安全代码

编写不安全代码时，你可能希望检查所写代码实际上是否安全、正确。完成这项工作的最佳方式之一是使用 Miri——一个用于检测未定义行为的 Rust 官方工具。借用检查器是在编译时工作的<em>静态(static)</em>工具，而 Miri 是在运行时工作的<em>动态(dynamic)</em>工具。它通过运行你的程序或测试套件来检查代码，并检测何时违反了它所理解的 Rust 工作规则。

使用 Miri 需要 Rust 的 nightly 构建版本（[附录 G：Rust 是如何开发的与“Nightly Rust”][nightly]<!-- ignore -->会进一步讨论）。输入 `rustup +nightly component add miri` 可以同时安装 Rust 的 nightly 版本和 Miri 工具。这不会改变项目使用的 Rust 版本；它只是把工具添加到系统中，以便在需要时使用。输入 `cargo +nightly miri run` 或 `cargo +nightly miri test` 可以在项目上运行 Miri。

为了说明它有多大帮助，看看对示例 20-7 运行 Miri 时会发生什么。

```console
{{#include ../../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri 正确地警告我们正在把整数转换为指针，这可能存在问题；但它无法确定问题是否存在，因为不知道该指针的来源。随后，Miri 在示例 20-7 出现未定义行为的位置返回错误，因为那里存在悬垂指针。借助 Miri，我们现在知道存在未定义行为的风险，可以思考如何让代码变得安全。在某些情况下，Miri 甚至能提出修复错误的建议。

Miri 无法捕获编写不安全代码时可能犯下的所有错误。Miri 是动态分析工具，因此只能捕获实际运行的代码中的问题。这意味着你需要把它与良好的测试方法结合使用，才能增强对所写不安全代码的信心。Miri 也没有覆盖代码可能不健全的每一种方式。

换句话说：如果 Miri <em>确实</em>捕获了问题，你就知道存在错误；但 Miri <em>没有</em>捕获错误，并不表示没有问题。尽管如此，它仍能捕获很多问题。试着对本章其他不安全代码示例运行 Miri，看看它会给出什么结果！

你可以在 [Miri 的 GitHub 仓库][miri]了解更多信息。

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

### 正确使用不安全代码

使用 `unsafe` 施展刚刚讨论的五种超能力之一并不是错误，也不会遭到反对；只是把 `unsafe` 代码写对更加困难，因为编译器无法帮助维护内存安全。当有理由使用 `unsafe` 代码时，你可以这样做；显式的 `unsafe` 标注会让问题发生时更容易追踪其来源。每当编写不安全代码时，都可以使用 Miri，帮助自己更有信心地确认所写代码维护了 Rust 的规则。

如需更深入地了解如何有效使用不安全 Rust，请阅读 Rust 的官方 `unsafe` 指南 [The Rustonomicon][nomicon]。

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: https://doc.rust-lang.org/reference/items/external-blocks.html#abi
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: https://doc.rust-lang.org/reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
