## 使用向量存储值列表

首先介绍 `Vec<T>`，也称为向量。向量在单个数据结构中存储多个值，并让它们在
内存中彼此相邻。向量只能存储相同类型的值，适合保存条目列表，例如文件中的文本
行或购物车中的商品价格。

### 创建新向量

创建新的空向量时，调用 `Vec::new`，如示例 8-1 所示。

<Listing number="8-1" caption="创建保存 `i32` 值的新空向量">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

这里添加了类型标注。由于没有向量中插入任何值，Rust 不知道准备存储哪种元素。
这是重要的一点。向量使用泛型实现；第 10 章会介绍如何为自己的类型使用泛型。
目前只需知道，标准库的 `Vec<T>` 可以保存任意类型。创建保存特定类型的向量时，
可以在尖括号中指定类型。示例 8-1 告诉 Rust，`v` 中的 `Vec<T>` 将保存 `i32`。

更常见的做法是使用初始值创建 `Vec<T>`，Rust 会推断要存储的类型，因此很少需要
类型标注。Rust 提供方便的 `vec!` 宏，根据给出的值创建新向量。示例 8-2 创建
保存 `1`、`2`、`3` 的 `Vec<i32>`。整数类型为 `i32`，因为它是默认整数类型，
正如第 3 章[“数据类型”][data-types]<!-- ignore -->一节所述。

<Listing number="8-2" caption="创建包含值的新向量">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

给出初始 `i32` 值后，Rust 能推断 `v` 为 `Vec<i32>`，无须类型标注。下面看看
如何修改向量。

### 更新向量

创建向量后，可以用 `push` 添加元素，如示例 8-3 所示。

<Listing number="8-3" caption="使用 `push` 方法向向量添加值">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

与其他变量一样，要改变值，必须用第 3 章介绍的 `mut` 将其设为可变。放入的数字
类型都是 `i32`，Rust 可以从数据推断，因此不需要 `Vec<i32>` 标注。

### 读取向量元素

引用向量中存储的值有两种方式：索引或 `get` 方法。为使示例更清楚，下面标出了
这些操作返回值的类型。

示例 8-4 同时展示索引语法和 `get` 方法访问向量值。

<Listing number="8-4" caption="使用索引语法和 `get` 方法访问向量条目">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

注意几个细节。向量索引从零开始，所以用索引 `2` 取得第三个元素。`&` 和 `[]`
会给出索引位置元素的引用。把索引作为实参传给 `get`，会得到可与 `match` 配合
使用的 `Option<&T>`。

Rust 提供两种引用元素的方式，让你可以选择索引超出已有元素范围时的程序行为。
例如，示例 8-5 对含五个元素的向量分别用两种方式访问索引 100。

<Listing number="8-5" caption="尝试访问五元素向量中索引为 100 的元素">

```rust,should_panic,panics
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

运行代码时，第一种 `[]` 方法会因为引用不存在的元素而导致程序 panic。如果越界
访问时希望程序崩溃，就适合使用这种方式。

向 `get` 传入越界索引时，它返回 `None` 而不会 panic。如果正常情况下偶尔可能
越界，应使用这种方式；代码再像第 6 章所述，处理 `Some(&element)` 或 `None`。
例如，索引可能来自用户输入。对方不慎输入过大数字并得到 `None` 时，可以告知
当前向量有多少条目，再给一次输入有效值的机会；这比因输入错误让程序崩溃友好！

程序拥有有效引用时，<em>借用检查器(borrow checker)</em>会执行第 4 章的所有权和借用
规则，确保该引用和向量内容的其他引用保持有效。回想同一作用域不能同时存在可变
与不可变引用的规则。示例 8-6 保存向量首个元素的不可变引用，同时尝试在末尾添加
元素；如果随后还使用该引用，程序就无法工作。

<Listing number="8-6" caption="持有向量条目引用时尝试添加元素">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

编译会产生以下错误：

```console
{{#include ../../listings/ch08-common-collections/listing-08-06/output.txt}}
```

示例 8-6 看似应该工作：首个元素的引用为何关心末尾的变化？原因在于向量的工作
方式。它把值相邻放在内存中；如果当前位置没有足够空间容纳全部元素，在末尾添加
元素可能需要分配新内存，并把旧元素复制过去。此时首元素的引用会指向已释放内存。
借用规则会阻止程序陷入这种情况。

> 注意：`Vec<T>` 的更多实现细节参阅 [Rustonomicon][nomicon]。

### 遍历向量中的值

依次访问向量的每个元素时，应遍历全部元素，而不是逐个使用索引。示例 8-7 用
`for` 循环取得 `i32` 向量各元素的不可变引用并打印。

<Listing number="8-7" caption="使用 `for` 循环遍历并打印向量中的每个元素">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

也可以遍历可变向量中每个元素的可变引用，修改全部元素。示例 8-8 的 `for` 循环
为每个元素加 `50`。

<Listing number="8-8" caption="遍历向量元素的可变引用">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

修改可变引用指向的值之前，必须使用 `*` 解引用运算符取得 `i` 中的值，再用 `+=`。
第 15 章的[“沿指针访问值”][deref]<!-- ignore -->一节会进一步讨论解引用运算符。

无论不可变还是可变地遍历向量，借用检查器规则都会保证安全。如果在示例 8-7 或
8-8 的循环体中插入或删除条目，会得到类似示例 8-6 的编译器错误。`for` 循环持有
的向量引用会阻止同时修改整个向量。

### 使用枚举存储多种类型

向量只能存储相同类型的值，这有时并不方便，因为确实存在保存不同类型条目列表的
需求。幸好枚举的所有变体都属于同一枚举类型；需要一个类型表示不同类型的元素时，
可以定义并使用枚举！

例如，读取电子表格中的一行值，其中一些列是整数，一些是浮点数，还有一些是
字符串。可以定义枚举，让不同变体保存不同值类型，而全部变体都视为同一枚举类型；
再创建保存该枚举的向量，最终就能保存不同类型。示例 8-9 展示了这种做法。

<Listing number="8-9" caption="定义枚举，在一个向量中存储不同类型的值">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust 必须在编译期知道向量中有哪些类型，才能确定在堆上为每个元素分配多少内存。
我们还必须明确允许哪些类型。如果向量能保存任意类型，其中某些类型可能不支持对
向量元素执行的操作，从而导致错误。结合枚举与 `match`，Rust 会像第 6 章所述，
在编译期确保处理每种可能情况。

如果不知道程序运行时将收到的完整类型集合，枚举方式就不可行。可以改用第 18 章
将介绍的特征对象。

了解向量的常见用法后，别忘了查看 [API 文档][vec-api]<!-- ignore -->，其中列出
标准库为 `Vec<T>` 定义的众多实用方法。例如除了 `push`，`pop` 会删除并返回
最后一个元素。

### 丢弃向量也会丢弃其元素

与其他 `struct` 一样，向量离开作用域时会被释放，如示例 8-10 的注释所示。

<Listing number="8-10" caption="展示向量及其元素在何处被丢弃">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

丢弃向量时，其中所有内容也会被丢弃，保存的整数会得到清理。借用检查器确保对
向量内容的引用只在向量本身有效时使用。

下面继续介绍下一种集合：`String`！

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: https://doc.rust-lang.org/nomicon/vec/vec.html
[vec-api]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
