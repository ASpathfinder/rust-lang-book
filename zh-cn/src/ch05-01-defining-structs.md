## 定义并实例化结构体

结构体与[“元组类型”][tuples]<!-- ignore -->一节介绍的元组相似，二者都能保存多个
相关值。与元组一样，结构体各部分可以具有不同类型。不同之处在于，结构体会给
每项数据命名，从而明确各个值的含义。这些名称让结构体比元组更灵活：指定或访问
实例中的值时，不必依赖数据的顺序。

定义结构体时，输入关键字 `struct`，并为整个结构体命名。结构体名称应描述组合
在一起的各项数据所代表的意义。然后在花括号内定义每项数据的名称和类型，这些
数据项称为<em>字段(field)</em>。例如，示例 5-1 定义了一个存储用户账户信息的结构体。

<Listing number="5-1" file-name="src/main.rs" caption="定义 `User` 结构体">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

定义结构体后，要使用它，需要为每个字段指定具体值，创建该结构体的<em>实例
(instance)</em>。创建实例时，先写结构体名称，再写花括号；花括号中包含 `key:
value` 对，其中键是字段名，值是要存储在相应字段中的数据。字段顺序不必与结构体
声明中的顺序相同。换句话说，结构体定义就像这种类型的通用模板，实例则用具体
数据填充模板，创建该类型的值。示例 5-2 声明了一个具体用户。

<Listing number="5-2" file-name="src/main.rs" caption="创建 `User` 结构体的实例">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

要从结构体中取得特定值，可以使用点号表示法。例如，使用 `user1.email` 访问该
用户的电子邮件地址。如果实例可变，就能通过点号表示法给特定字段赋值来改变它。
示例 5-3 修改可变 `User` 实例的 `email` 字段。

<Listing number="5-3" file-name="src/main.rs" caption="修改 `User` 实例的 `email` 字段值">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

请注意，整个实例都必须可变；Rust 不允许只把某些字段标记为可变。与其他表达式
一样，可以把结构体的新实例作为函数体的最后一个表达式，从而隐式返回它。

示例 5-4 中的 `build_user` 函数接收电子邮件和用户名，并返回一个 `User` 实例。
`active` 字段取得值 `true`，`sign_in_count` 取得值 `1`。

<Listing number="5-4" file-name="src/main.rs" caption="接收电子邮件和用户名并返回 `User` 实例的 `build_user` 函数">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

函数形参与结构体字段使用相同名称很合理，但反复书写 `email` 和 `username` 的
字段名与变量名有些繁琐。结构体字段更多时，重复每个名称会更加恼人。幸好这里有
一种方便的简写！

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### 使用字段初始化简写

示例 5-4 中的形参名与结构体字段名完全相同，因此可以使用<em>字段初始化简写(field
init shorthand)</em>语法重写 `build_user`。如示例 5-5 所示，函数行为完全相同，
却不必重复 `username` 和 `email`。

<Listing number="5-5" file-name="src/main.rs" caption="形参 `username` 和 `email` 与结构体字段同名，因此 `build_user` 使用字段初始化简写">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

这里创建一个新的 `User` 实例，它有一个名为 `email` 的字段。我们希望把 `email`
字段设为 `build_user` 函数的 `email` 形参值。由于字段和形参同名，只需写
`email`，无须写 `email: email`。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### 使用结构体更新语法创建实例

创建结构体的新实例时，经常需要沿用同类型另一实例的大部分值，只修改其中一些。
这可以通过<em>结构体更新语法(struct update syntax)</em>完成。

示例 5-6 先展示不使用更新语法，以常规方式在 `user2` 中创建新 `User` 实例。
我们为 `email` 设置新值，其他字段则使用示例 5-2 中 `user1` 的相同值。

<Listing number="5-6" file-name="src/main.rs" caption="使用 `user1` 中除一个值之外的所有值创建新 `User` 实例">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

使用结构体更新语法，可以用更少代码达到相同效果，如示例 5-7 所示。`..` 表示
没有显式设置的其余字段应与给定实例中的字段具有相同值。

<Listing number="5-7" file-name="src/main.rs" caption="用结构体更新语法为 `User` 实例设置新的 `email`，其余值取自 `user1`">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

示例 5-7 同样在 `user2` 中创建一个实例：它的 `email` 值不同，但 `username`、
`active` 和 `sign_in_count` 与 `user1` 相同。`..user1` 必须放在最后，表示所有
剩余字段都从 `user1` 的对应字段取得值；但无论结构体定义中的字段顺序如何，我们
都可以按任意顺序为任意数量的字段指定值。

请注意，结构体更新语法像赋值一样使用 `=`，这是因为它会移动数据，正如[“变量与
数据通过移动交互”][move]<!-- ignore -->一节所述。在这个示例中，创建 `user2` 后
不能再使用 `user1`，因为 `user1.username` 中的 `String` 已移动到 `user2`。
如果为 `user2` 的 `email` 和 `username` 都提供新的 `String`，只使用 `user1`
中的 `active` 和 `sign_in_count`，那么创建 `user2` 后 `user1` 仍然有效。这两个
字段的类型都实现了 `Copy` 特征，因此适用[“只在栈上的数据：复制”][copy]
<!-- ignore -->一节介绍的行为。这个示例中仍然可以使用 `user1.email`，因为它的
值没有移出 `user1`。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

<a id="creating-different-types-with-tuple-structs"></a>

### 使用元组结构体创建不同类型

Rust 还支持一种看起来与元组相似的结构体，称为<em>元组结构体(tuple struct)</em>。
元组结构体通过结构体名称赋予整体额外含义，却不为字段命名，只指定字段类型。
当你希望给整个元组命名、让它成为不同于其他元组的类型，而像普通结构体那样为
每个字段命名又显得冗长或多余时，元组结构体很有用。

定义元组结构体时，先写 `struct` 关键字和结构体名称，再写元组中的类型。例如，
下面定义并使用名为 `Color` 和 `Point` 的两个元组结构体：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

`black` 和 `origin` 是不同类型的值，因为它们是不同元组结构体的实例。定义的
每个结构体都是独立类型，即使结构体内部字段的类型完全相同。例如，接收 `Color`
形参的函数不能接收 `Point` 实参，尽管两种类型都由三个 `i32` 值组成。在其他
方面，元组结构体实例与元组相似：可以解构为单独部分，也可以用 `.` 加索引访问
单个值。不同之处是，解构元组结构体时必须写出结构体类型名。例如，使用
`let Point(x, y, z) = origin;` 把点 `origin` 的值解构到变量 `x`、`y`、`z` 中。

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### 定义类单元结构体

还可以定义没有任何字段的结构体！这种结构体称为<em>类单元结构体(unit-like
struct)</em>，因为它的行为类似于[“元组类型”][tuples]<!-- ignore -->一节提到的
单元类型 `()`。当需要为某个类型实现特征，却没有任何数据需要存储在类型本身时，
类单元结构体很有用。第 10 章会讨论特征。下面声明并实例化名为 `AlwaysEqual`
的类单元结构体：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

定义 `AlwaysEqual` 时，只使用 `struct` 关键字、所需名称和分号，不需要花括号或
圆括号！随后可以用类似方式在变量 `subject` 中取得 `AlwaysEqual` 的实例：只写
定义好的名称，不加花括号或圆括号。设想以后为该类型实现一种行为，让
`AlwaysEqual` 的每个实例始终等于其他任何类型的每个实例，也许可以为测试提供
已知结果。实现这种行为不需要任何数据！第 10 章会介绍如何定义特征，并为包括
类单元结构体在内的任意类型实现特征。

> ### 结构体数据的所有权
>
> 示例 5-1 的 `User` 结构体定义使用了拥有所有权的 `String` 类型，而不是 `&str`
> 字符串切片类型。这是有意的选择，因为我们希望结构体的每个实例都拥有自己的
> 全部数据，并且只要整个结构体有效，这些数据就一直有效。
>
> 结构体也可以存储对其他对象所拥有数据的引用，但这需要使用<em>生命周期
> (lifetime)</em>，这是第 10 章将讨论的 Rust 特性。生命周期确保结构体引用的数据
> 在结构体有效期间一直有效。假设你尝试在结构体中存储引用，却没有指定生命周期，
> 如下面的 <em>src/main.rs</em> 所示；这无法工作：
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> 编译器会指出需要生命周期说明符：
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> 第 10 章会讨论如何修复这些错误，以便在结构体中存储引用；目前遇到此类错误时，
> 可以像这里一样使用 `String` 这类拥有所有权的类型，而不是 `&str` 这类引用。

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
