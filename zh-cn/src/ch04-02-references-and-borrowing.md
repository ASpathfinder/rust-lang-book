<a id="references-and-borrowing"></a>

## 引用与借用

示例 4-5 中元组代码的问题在于：`String` 被移动进 `calculate_length` 后，为了能
在调用结束后继续使用它，必须让 `calculate_length` 把它返回给调用函数。我们
可以改为提供 `String` 值的<em>引用(reference)</em>。引用类似于指针，它是一个地址，
我们可以沿着该地址访问其中存储的数据；这些数据归其他变量所有。与指针不同，
在引用的整个生命周期内，它保证指向某个特定类型的有效值。

下面定义并使用一个 `calculate_length` 函数：它以对象的引用作为形参，而不是
取得值的所有权。

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

首先，变量声明和函数返回值中的所有元组代码都消失了。其次，我们把 `&s1` 传入
`calculate_length`，并在函数定义中接收 `&String` 而非 `String`。这些与号表示
引用，让你可以引用某个值而不取得其所有权。图 4-6 展示了这个概念。

<img alt="三个表格：s 的表格只包含一个指向 s1 表格的指针；s1 的表格包含 s1 的栈数据，并指向堆上的字符串数据。" src="img/trpl04-06.svg" class="center" />

<span class="caption">图 4-6：`&String` 类型的 `s` 指向 `String` 类型的 `s1`</span>

> 注意：使用 `&` 取得引用的相反操作是<em>解引用(dereferencing)</em>，通过解引用
> 运算符 `*` 完成。第 8 章会看到解引用运算符的一些用法，第 15 章会详细讨论
> 解引用。

仔细看看这里的函数调用：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

`&s1` 语法创建一个<em>指向(refers to)</em> `s1` 值但不拥有它的引用。因为引用并不
拥有该值，所以引用停止使用时，它指向的值不会被丢弃。

同样，函数签名使用 `&` 表示形参 `s` 的类型是引用。下面添加一些说明性注释：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

变量 `s` 的有效作用域与其他函数形参相同，但 `s` 停止使用时，引用指向的值不会
被丢弃，因为 `s` 没有所有权。当函数以引用而非实际值作为形参时，我们无须为了
归还所有权而返回这些值，因为函数从未拥有它们。

创建引用的动作称为<em>借用(borrowing)</em>。就像现实生活中一样，如果某人拥有一件
东西，你可以向他借用；用完以后必须归还，因为你并不拥有它。

如果尝试修改借来的东西，会发生什么？试试示例 4-6 中的代码。提前透露一下：
它无法工作！

<Listing number="4-6" file-name="src/main.rs" caption="尝试修改借用的值">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

错误如下：

```console
{{#include ../../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

引用和变量一样默认不可变。我们不能修改引用所指向的内容。

### 可变引用

只需作几处小改动，改用<em>可变引用(mutable reference)</em>，就能修复示例 4-6 的
代码并允许修改借用的值：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

首先把 `s` 改为 `mut`。然后，在调用 `change` 函数的位置用 `&mut s` 创建可变
引用，并把函数签名改为通过 `some_string: &mut String` 接受可变引用。这清楚地
表明 `change` 函数将修改它所借用的值。

可变引用有一个重要限制：如果有一个值的可变引用，就不能再有该值的任何其他
引用。下面的代码试图为 `s` 创建两个可变引用，因此会失败：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

错误如下：

```console
{{#include ../../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

这个错误说明代码无效，因为我们不能同时多次以可变方式借用 `s`。第一次可变借用
位于 `r1`，而且必须持续到 `println!` 使用它为止；但在创建该可变引用与使用它
之间，代码又尝试在 `r2` 中创建另一个可变引用，借用与 `r1` 相同的数据。

禁止同时对同一数据创建多个可变引用的限制，允许程序以严格受控的方式修改数据。
Rust 新手经常为此感到不适应，因为大多数语言允许随时修改数据。这项限制的好处
是，Rust 能在编译期阻止<em>数据竞争(data race)</em>。数据竞争类似于竞态条件，会在
以下三种行为同时发生时出现：

- 两个或更多指针同时访问同一数据。
- 至少一个指针正在写入数据。
- 没有使用任何机制同步对数据的访问。

数据竞争会导致未定义行为，在运行时追踪时可能难以诊断和修复。Rust 通过拒绝
编译存在数据竞争的代码来阻止这个问题！

和往常一样，可以用花括号创建新作用域，从而拥有多个可变引用，只是不能<em>同时</em>
拥有：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust 对可变引用与不可变引用的组合执行类似规则。下面的代码会产生错误：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

错误如下：

```console
{{#include ../../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

呼！拥有指向同一值的不可变引用时，我们<em>同样</em>不能再拥有可变引用。

不可变引用的使用者不会预期值突然在眼前改变！但可以存在多个不可变引用，因为
只读取数据的人无法影响其他人读取数据。

请注意，引用的作用域从引入它的位置开始，一直持续到最后一次使用该引用的位置。
例如，下面的代码能够编译，因为不可变引用最后一次用于 `println!`，发生在引入
可变引用之前：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

不可变引用 `r1` 和 `r2` 的作用域在最后使用它们的 `println!` 后结束，早于创建
可变引用 `r3`。这些作用域没有重叠，因此允许这段代码：编译器可以判断引用在
作用域真正结束之前就已经不再使用。

尽管借用错误有时令人沮丧，但请记住：Rust 编译器是在尽早指出潜在错误（发生在
编译期而非运行时），并准确显示问题所在。这样就不必追查数据为何与预期不同。

<a id="dangling-references"></a>

### 悬垂引用

在使用指针的语言中，很容易错误地创建<em>悬垂指针(dangling pointer)</em>：释放一块
内存后仍然保留指向它的指针，而该内存可能已经分配给别人。相比之下，Rust 编译器
保证引用永远不会成为<em>悬垂引用(dangling reference)</em>：如果存在某些数据的引用，
编译器就会确保数据不会先于引用离开作用域。

下面尝试创建一个悬垂引用，看看 Rust 如何通过编译期错误阻止它：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

错误如下：

```console
{{#include ../../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

这条错误信息提到一个尚未介绍的特性：<em>生命周期(lifetime)</em>。第 10 章会详细
讨论生命周期。不过，即使忽略与生命周期有关的部分，错误信息仍然指出了这段代码
存在问题的关键：

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

仔细看看 `dangle` 代码每个阶段究竟发生了什么：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

因为 `s` 在 `dangle` 内部创建，所以 `dangle` 的代码结束后，`s` 会被释放。但
我们却尝试返回它的引用，这意味着该引用将指向一个无效的 `String`。这可不行！
Rust 不允许这样做。

解决办法是直接返回 `String`：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

这段代码能够正常工作。所有权被移出函数，没有任何内容被释放。

### 引用规则

回顾一下引用的规则：

- 在任意时刻，只能拥有<em>一个</em>可变引用，<em>或者</em>任意数量的不可变引用。
- 引用必须始终有效。

接下来考察另一种引用：切片。
