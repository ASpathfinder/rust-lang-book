<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>
<a id="closures-anonymous-functions-that-capture-their-environment"></a>

## 闭包

Rust 的<em>闭包(closure)</em>是可以保存到变量中或作为实参传给其他函数的匿名函数。你可以在一个位置创建闭包，再到其他地方调用闭包，在不同上下文中对其求值。与函数不同，闭包可以捕获其定义所在作用域中的值。我们会展示这些闭包功能如何实现代码复用和行为定制。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>
<a id="capturing-the-environment-with-closures"></a>

### 捕获环境

首先考察如何使用闭包捕获其定义环境中的值，供稍后使用。场景如下：我们的 T 恤公司偶尔会向邮件列表中的某个人赠送一件独家限量版 T 恤作为促销。邮件列表中的用户可以选择把自己最喜欢的颜色添加到个人资料。如果被选中获赠 T 恤的人设置了最喜欢的颜色，就会得到该颜色的 T 恤；如果没有指定，就会得到公司目前库存最多的颜色。

这项功能有许多实现方式。本例会使用名为 `ShirtColor` 的枚举，它包含 `Red` 和 `Blue` 两个变体（为简单起见，限制可用颜色数量）。我们使用 `Inventory` 结构体表示公司库存；该结构体有一个名为 `shirts` 的字段，其中包含 `Vec<ShirtColor>`，代表当前有货的 T 恤颜色。在 `Inventory` 上定义的 `giveaway` 方法会取得赠品获奖者可选的 T 恤颜色偏好，并返回该用户将得到的颜色。示例 13-1 展示了这项设置。

<Listing number="13-1" file-name="src/main.rs" caption="T 恤公司赠品场景">

```rust,noplayground
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`main` 中定义的 `store` 还剩两件蓝色和一件红色 T 恤，用于本次限量版促销。我们分别为偏爱红色 T 恤的用户和没有任何偏好的用户调用 `giveaway` 方法。

再次说明，这段代码可以有许多实现方式；为了专注于闭包，除了使用闭包的 `giveaway` 方法函数体，我们只采用已经学过的概念。在 `giveaway` 方法中，我们以 `Option<ShirtColor>` 类型的形参取得用户偏好，并对 `user_preference` 调用 `unwrap_or_else` 方法。标准库定义了 [`Option<T>` 上的 `unwrap_or_else` 方法][unwrap-or-else]。它接收一个实参：不带任何实参、返回 `T` 值的闭包（`T` 与 `Option<T>` 的 `Some` 变体中存储的类型相同，这里是 `ShirtColor`）。如果 `Option<T>` 是 `Some` 变体，`unwrap_or_else` 会返回 `Some` 中的值；如果是 `None` 变体，`unwrap_or_else` 会调用闭包，并返回闭包的返回值。

我们把闭包表达式 `|| self.most_stocked()` 指定为 `unwrap_or_else` 的实参。这个闭包本身不接收形参（如果闭包有形参，它们会出现在两条竖线之间）。闭包的函数体调用 `self.most_stocked()`。我们在这里定义闭包，而 `unwrap_or_else` 的实现稍后会在需要结果时对闭包求值。

运行这段代码会打印：

```console
{{#include ../../listings/ch13-functional-features/listing-13-01/output.txt}}
```

这里有趣的一点是，我们传入了一个会对当前 `Inventory` 实例调用 `self.most_stocked()` 的闭包。标准库无需了解我们定义的 `Inventory` 或 `ShirtColor` 类型，也无需了解这个场景中想使用的逻辑。闭包捕获对 `self` 的 `Inventory` 实例的不可变引用，并将它与指定代码一起传给 `unwrap_or_else` 方法。相比之下，函数无法以这种方式捕获其环境。

<!-- Old headings. Do not remove or links may break. -->

<a id="closure-type-inference-and-annotation"></a>

### 推断和标注闭包类型

函数与闭包还有更多区别。闭包通常不要求像 `fn` 函数那样标注形参或返回值的类型。函数需要类型标注，是因为类型属于向用户公开的显式接口。严格定义该接口对于确保所有人就函数使用和返回的值类型达成一致非常重要。而闭包不会用于这种公开接口：它们存储在变量中，使用时没有名称，也不会向库的用户公开。

闭包通常很短，只与狭窄的上下文有关，而不是适用于任意场景。在这些有限上下文中，编译器可以推断形参和返回类型，就像它能够推断大多数变量的类型一样（极少数情况下，编译器也需要闭包类型标注）。

与变量一样，如果愿意，可以添加类型标注来提高明确性和清晰度，代价是比严格意义上所需的写法更加冗长。为闭包标注类型的写法如示例 13-2 中的定义所示。本例把闭包定义并存储在变量中，而不是像示例 13-1 那样在把闭包作为实参传入的位置进行定义。

<Listing number="13-2" file-name="src/main.rs" caption="为闭包的形参和返回值类型添加可选的类型标注">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

添加类型标注后，闭包语法看起来与函数语法更加相似。这里，为了比较，我们定义了一个把形参加 1 的函数，以及一个行为相同的闭包。我们添加了一些空格，让相关部分对齐。这说明闭包语法与函数语法类似，区别在于使用竖线，以及许多语法都可以省略：

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

第一行展示函数定义，第二行展示完整标注的闭包定义。第三行删除了闭包定义中的类型标注。第四行删除了花括号；因为闭包函数体只有一个表达式，花括号可选。这些都是有效定义，调用时会产生相同行为。`add_one_v3` 和 `add_one_v4` 行中的闭包必须经过求值才能编译，因为其类型会从用法中推断。这与 `let v = Vec::new();` 需要类型标注，或需要向 `Vec` 插入某种类型的值以便 Rust 推断类型类似。

对于闭包定义，编译器会为其每个形参和返回值各推断一个具体类型。例如，示例 13-3 定义了一个只返回所接收形参值的短闭包。除了用于这个例子，它并没有太大用处。请注意，我们没有为定义添加任何类型标注。由于没有类型标注，看起来可以使用任意类型调用闭包；这里第一次以 `String` 调用了它。但随后如果尝试使用整数调用 `example_closure`，就会出错。

<Listing number="13-3" file-name="src/main.rs" caption="尝试使用两种不同类型调用类型经过推断的闭包">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

编译器会给出以下错误：

```console
{{#include ../../listings/ch13-functional-features/listing-13-03/output.txt}}
```

第一次使用 `String` 值调用 `example_closure` 时，编译器推断 `x` 的类型和闭包的返回类型都是 `String`。这些类型随后会锁定到 `example_closure` 中的闭包；下一次尝试对同一闭包使用不同类型时，就会出现类型错误。

<a id="capturing-references-or-moving-ownership"></a>

### 捕获引用或移动所有权

闭包可以通过三种方式从环境捕获值，它们与函数接收形参的三种方式直接对应：不可变借用、可变借用和取得所有权。闭包会根据函数体如何使用所捕获的值，决定采用哪种方式。

在示例 13-4 中，我们定义了一个捕获对名为 `list` 的向量的不可变引用的闭包，因为打印值只需要不可变引用。

<Listing number="13-4" file-name="src/main.rs" caption="定义并调用捕获不可变引用的闭包">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

这个例子还说明，变量可以绑定到闭包定义；之后，我们可以使用变量名和圆括号调用闭包，就像变量名是函数名一样。

由于可以同时拥有多个对 `list` 的不可变引用，因此在闭包定义之前、定义之后但调用之前，以及调用之后的代码中，仍然可以访问 `list`。这段代码可以编译、运行，并打印：

```console
{{#include ../../listings/ch13-functional-features/listing-13-04/output.txt}}
```

接着，在示例 13-5 中，我们修改闭包函数体，让它向 `list` 向量添加元素。现在，闭包会捕获可变引用。

<Listing number="13-5" file-name="src/main.rs" caption="定义并调用捕获可变引用的闭包">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

这段代码可以编译、运行，并打印：

```console
{{#include ../../listings/ch13-functional-features/listing-13-05/output.txt}}
```

请注意，`borrows_mutably` 闭包的定义与调用之间不再有 `println!`：定义 `borrows_mutably` 时，它会捕获对 `list` 的可变引用。闭包调用之后不再使用该闭包，所以可变借用结束。在闭包定义与调用之间，不允许为了打印而进行不可变借用，因为存在可变借用时不允许其他借用。请尝试在那里添加 `println!`，看看会得到什么错误信息！

如果想强制闭包取得它在环境中使用的值的所有权，即使闭包函数体并不严格需要所有权，也可以在形参列表之前使用 `move` 关键字。

这种技巧主要用于把闭包传给新线程时移动数据，让新线程拥有它。第 16 章讨论并发时，会详细介绍线程以及为什么要使用它们；现在，先简要探索如何使用需要 `move` 关键字的闭包生成新线程。示例 13-6 修改了示例 13-4，让向量在新线程而不是主线程中打印。

<Listing number="13-6" file-name="src/main.rs" caption="使用 `move` 强制线程闭包取得 `list` 的所有权">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

我们生成新线程，并把要运行的闭包作为实参交给线程。闭包函数体打印列表。在示例 13-4 中，闭包只使用不可变引用捕获 `list`，因为这是打印它所需的最低访问权限。在这个例子中，虽然闭包函数体仍然只需要不可变引用，但需要在闭包定义开头放置 `move` 关键字，指定把 `list` 移入闭包。如果主线程在对新线程调用 `join` 之前执行更多操作，新线程可能会先于主线程其余部分结束，也可能是主线程先结束。如果主线程保持 `list` 的所有权，却先于新线程结束并丢弃 `list`，线程中的不可变引用就会失效。因此，编译器要求把 `list` 移入传给新线程的闭包，确保引用有效。请尝试删除 `move` 关键字，或在定义闭包后于主线程中使用 `list`，看看会得到什么编译器错误！

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>
<a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

<a id="moving-captured-values-out-of-closures"></a>

### 把捕获的值移出闭包

闭包从定义它的环境中捕获引用或捕获值的所有权后（从而影响哪些内容被移<em>入</em>闭包），闭包函数体中的代码会定义稍后对闭包求值时引用或值会发生什么（从而影响哪些内容被移<em>出</em>闭包）。

闭包函数体可以执行以下任一种操作：把捕获的值移出闭包、修改捕获的值、既不移动也不修改该值，或者一开始就不从环境捕获任何内容。

闭包从环境捕获和处理值的方式，会影响闭包实现哪些特征；函数和结构体正是通过特征指定可使用哪些种类的闭包。根据闭包函数体处理值的方式，闭包会以累加方式自动实现以下一个、两个或全部三个 `Fn` 特征：

* `FnOnce` 适用于可以调用一次的闭包。所有闭包至少都会实现该特征，因为所有闭包都能被调用。把捕获的值移出函数体的闭包只会实现 `FnOnce`，而不实现其他 `Fn` 特征，因为它只能调用一次。
* `FnMut` 适用于不会把捕获的值移出函数体、但可能修改捕获值的闭包。这些闭包可以调用多次。
* `Fn` 适用于既不把捕获的值移出函数体、也不修改捕获值的闭包，以及不从环境捕获任何内容的闭包。这些闭包可以调用多次而不修改环境；在并发多次调用闭包等情况下，这一点很重要。

让我们看看示例 13-1 中使用的 `Option<T>` 上 `unwrap_or_else` 方法的定义：

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

回想一下，`T` 是表示 `Option` 的 `Some` 变体中值类型的泛型类型。类型 `T` 也是 `unwrap_or_else` 函数的返回类型：例如，对 `Option<String>` 调用 `unwrap_or_else` 的代码会得到 `String`。

接着，请注意 `unwrap_or_else` 函数还有额外的泛型类型形参 `F`。`F` 类型是名为 `f` 的形参类型，也就是调用 `unwrap_or_else` 时提供的闭包。

在泛型类型 `F` 上指定的特征约束是 `FnOnce() -> T`，表示 `F` 必须能够被调用一次、不接收实参，并返回 `T`。在特征约束中使用 `FnOnce`，表达了 `unwrap_or_else` 不会多次调用 `f` 的约束。在 `unwrap_or_else` 函数体中可以看到，如果 `Option` 是 `Some`，不会调用 `f`；如果 `Option` 是 `None`，会调用 `f` 一次。由于所有闭包都实现 `FnOnce`，`unwrap_or_else` 接受全部三种闭包，达到了最大灵活性。

> 注意：如果想做的事情不需要从环境捕获值，可以在需要实现某个 `Fn` 特征的内容时使用函数名，而不是闭包。例如，对于 `Option<Vec<T>>` 值，可以调用 `unwrap_or_else(Vec::new)`，在值为 `None` 时取得一个新的空向量。编译器会自动为函数定义实现适用的 `Fn` 特征。

现在看看在切片上定义的标准库方法 `sort_by_key`，了解它与 `unwrap_or_else` 有何不同，以及为什么 `sort_by_key` 的特征约束使用 `FnMut` 而不是 `FnOnce`。闭包接收一个实参，即对当前正在考察的切片项的引用，并返回可以排序的 `K` 类型值。如果想按每一项的某个特定属性对切片排序，这个函数很有用。在示例 13-7 中，我们有一个 `Rectangle` 实例列表，并使用 `sort_by_key` 按 `width` 属性从低到高排列它们。

<Listing number="13-7" file-name="src/main.rs" caption="使用 `sort_by_key` 按宽度排列矩形">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

这段代码会打印：

```console
{{#include ../../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key` 被定义为接收 `FnMut` 闭包，是因为它会多次调用闭包：切片中的每一项调用一次。闭包 `|r| r.width` 不从环境捕获、修改或移出任何内容，因此满足特征约束要求。

相比之下，示例 13-8 展示了只实现 `FnOnce` 特征的闭包，因为它会把一个值移出环境。编译器不允许将这个闭包用于 `sort_by_key`。

<Listing number="13-8" file-name="src/main.rs" caption="尝试对 `sort_by_key` 使用 `FnOnce` 闭包">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

这是一种刻意构造、十分复杂（而且无法工作）的方式，试图计算对 `list` 排序时 `sort_by_key` 调用闭包的次数。这段代码尝试通过把闭包环境中的 `String` 值 `value` 推入 `sort_operations` 向量来计数。闭包捕获 `value`，随后通过把 `value` 的所有权转移给 `sort_operations` 向量，将其移出闭包。这个闭包可以调用一次；尝试第二次调用会失败，因为环境中不再有 `value` 可以再次推入 `sort_operations`！因此，这个闭包只实现 `FnOnce`。尝试编译代码时，会得到错误，指出无法把 `value` 移出闭包，因为闭包必须实现 `FnMut`：

```console
{{#include ../../listings/ch13-functional-features/listing-13-08/output.txt}}
```

错误指向闭包函数体中把 `value` 移出环境的那一行。为了修复，需要修改闭包函数体，使其不再把值移出环境。在环境中维护一个计数器，并在闭包函数体中递增其值，是计算闭包调用次数更直接的方式。示例 13-9 中的闭包可以与 `sort_by_key` 配合使用，因为它只捕获对 `num_sort_operations` 计数器的可变引用，因此可以调用多次。

<Listing number="13-9" file-name="src/main.rs" caption="允许对 `sort_by_key` 使用 `FnMut` 闭包">

```rust
{{#rustdoc_include ../../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

定义或使用会利用闭包的函数或类型时，`Fn` 特征非常重要。下一节会讨论迭代器。许多迭代器方法都接收闭包实参，请在继续阅读时牢记这些闭包细节！

[unwrap-or-else]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else
