<a id="the-slice-type"></a>

## 切片类型

<em>切片(slice)</em>允许你引用[<em>集合(collection)</em>](ch08-00-common-collections.html)<!-- ignore -->
中一段连续的元素序列。切片是一种引用，因此没有所有权。

来看一个小编程问题：编写一个函数，接收由空格分隔的单词字符串，并返回在该
字符串中找到的第一个单词。如果函数没有找到空格，说明整个字符串就是一个单词，
因此应返回完整字符串。

> 注意：为了介绍切片，本节只考虑 ASCII。第 8 章的[“用字符串存储 UTF-8 编码的
> 文本”][strings]<!-- ignore -->一节会更全面地讨论 UTF-8 处理。

为了理解切片要解决的问题，先看看不使用切片时该如何编写这个函数的签名：

```rust,ignore
fn first_word(s: &String) -> ?
```

`first_word` 函数有一个 `&String` 类型的形参。我们不需要所有权，所以这样很好。
（在惯用 Rust 中，除非确有需要，否则函数不会取得实参的所有权；随着学习深入，
其中的原因会越来越清楚。）但应该返回什么？我们其实没有办法表达字符串的
<em>一部分</em>。不过，可以返回表示单词结尾空格位置的索引。示例 4-7 就这样做。

<Listing number="4-7" file-name="src/main.rs" caption="`first_word` 函数返回 `String` 形参中的字节索引值">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

因为需要逐个检查 `String` 中的元素是否为空格，所以先使用 `as_bytes` 方法把
`String` 转换为字节数组。

```rust,ignore
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

接着，使用 `iter` 方法为字节数组创建一个<em>迭代器(iterator)</em>：

```rust,ignore
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

[第 13 章][ch13]<!-- ignore -->会详细讨论迭代器。目前只需知道，`iter` 方法返回集合中的每个元素；
`enumerate` 则封装 `iter` 的结果，把每个元素作为元组的一部分返回。`enumerate`
返回的元组中，第一个元素是索引，第二个元素是元素的引用。这比自行计算索引方便
一些。

因为 `enumerate` 方法返回元组，所以可以使用模式解构该元组。[第 6 章][ch6]
<!-- ignore -->会进一步讨论模式。在 `for` 循环中，我们指定一个模式：`i` 表示元组中的索引，`&item`
表示元组中的单个字节。由于 `.iter().enumerate()` 返回元素的引用，因此模式中
使用 `&`。

在 `for` 循环内部，使用字节字面量语法寻找表示空格的字节。如果找到空格，就
返回其位置；否则使用 `s.len()` 返回字符串长度。

```rust,ignore
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

现在可以找到字符串中第一个单词结尾的索引，但仍有问题。我们单独返回一个
`usize`，而这个数字只有在 `&String` 的上下文中才有意义。换句话说，因为它是
一个与 `String` 分离的值，所以无法保证将来仍然有效。看看示例 4-8，它使用了
示例 4-7 中的 `first_word` 函数。

<Listing number="4-8" file-name="src/main.rs" caption="保存 `first_word` 函数的调用结果，然后改变 `String` 的内容">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

这段程序可以无错编译；即使在调用 `s.clear()` 后使用 `word`，也仍然能编译。
因为 `word` 与 `s` 的状态毫无关联，所以 `word` 依旧包含值 `5`。我们可能会
尝试用这个值 `5` 从变量 `s` 中提取第一个单词，但这是个错误，因为把 `5` 保存
进 `word` 后，`s` 的内容已经改变。

必须担心 `word` 中的索引与 `s` 中的数据失去同步，既麻烦又容易出错！如果再
编写一个 `second_word` 函数，管理这些索引会更加脆弱。它的签名必须像这样：

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

现在要同时跟踪起始索引和结束索引；这些值根据数据在某个特定状态下计算得出，却
与该状态毫无关联。三个互不相关的变量四处游离，还必须保持同步。

幸好 Rust 为这个问题提供了解决方案：字符串切片。

<a id="string-slices"></a>

### 字符串切片

<em>字符串切片(string slice)</em>是对 `String` 中一段连续元素的引用，如下所示：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

`hello` 并非整个 `String` 的引用，而是由额外的 `[0..5]` 指定的 `String` 部分
的引用。我们在方括号内使用范围 `[starting_index..ending_index]` 创建切片：
<code>starting_index</code> 是切片中的第一个位置，<code>ending_index</code> 是切片
最后位置的下一个位置。在内部，切片数据结构保存起始位置和切片长度，后者等于
<code>ending_index</code> 减去 <code>starting_index</code>。因此，对于
`let world = &s[6..11];`，`world` 切片包含一个指向 `s` 中索引 6 处字节的
指针，长度值为 `5`。

图 4-7 展示了这一布局。

<img alt="三个表格：表示 s 的栈数据的表格指向堆上字符串数据 hello world 的索引 0 字节；第三个表格表示切片 world 的栈数据，其长度为 5，并指向堆数据表的字节 6。" src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-7：引用 `String` 一部分的字符串切片</span>

使用 Rust 的 `..` 范围语法时，如果希望从索引 0 开始，可以省略两个点之前的值。
也就是说，以下写法相同：

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

同理，如果切片包含 `String` 的最后一个字节，可以省略末尾的数字。因此，以下
写法相同：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

还可以同时省略两个值，取得整个字符串的切片。因此，以下写法相同：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 注意：字符串切片的范围索引必须位于有效的 UTF-8 字符边界上。如果尝试从一个
> 多字节字符的中间位置创建字符串切片，程序会出错并退出。

了解这些信息后，重写 `first_word` 使其返回切片。“字符串切片”类型写作 `&str`：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

和示例 4-7 一样，通过寻找第一次出现的空格来取得单词结尾的索引。找到空格时，
以字符串开头和空格索引作为起止索引，返回一个字符串切片。

现在调用 `first_word` 时，得到的是一个与底层数据相关联的值。这个值由指向切片
起点的引用以及切片中的元素数量组成。

返回切片的方式同样适用于 `second_word` 函数：

```rust,ignore
fn second_word(s: &String) -> &str {
```

现在得到一个简洁得多、也更不容易误用的 API，因为编译器会确保指向 `String`
内部的引用保持有效。还记得示例 4-8 中的错误吗？取得第一个单词结尾的索引后，
我们清空了字符串，使索引失效。那段代码在逻辑上错误，却没有立即报错；如果继续
用第一个单词的索引访问已清空的字符串，问题才会显现。切片让这种错误无法发生，
并能更早提醒我们代码有问题。使用切片版本的 `first_word` 会产生编译期错误：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

编译器错误如下：

```console
{{#include ../../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

回顾借用规则：如果有某个值的不可变引用，就不能同时取得它的可变引用。`clear`
需要截短 `String`，因此必须取得可变引用。调用 `clear` 后的 `println!` 会使用
`word` 中的引用，所以不可变引用在此处仍然有效。Rust 不允许 `clear` 中的可变
引用与 `word` 中的不可变引用同时存在，因此编译失败。Rust 不仅让 API 更容易
使用，还在编译期消除了一整类错误！

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### 字符串字面量就是切片

前面提到过，字符串字面量存储在二进制文件中。了解切片后，现在可以准确理解
字符串字面量：

```rust
let s = "Hello, world!";
```

这里 `s` 的类型是 `&str`：它是指向二进制文件中特定位置的切片。这也解释了
字符串字面量为何不可变；`&str` 是不可变引用。

<a id="string-slices-as-parameters"></a>

#### 以字符串切片作为形参

知道可以取得字面量和 `String` 值的切片后，还能进一步改进 `first_word` 的签名：

```rust,ignore
fn first_word(s: &String) -> &str {
```

经验更丰富的 Rust 程序员会改为编写示例 4-9 所示的签名，因为这样同一个函数既
能用于 `&String` 值，也能用于 `&str` 值。

<Listing number="4-9" caption="使用字符串切片作为形参 `s` 的类型，改进 `first_word` 函数">

```rust,ignore
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

如果有字符串切片，可以直接传入；如果有 `String`，可以传入 `String` 的切片或
`String` 的引用。这种灵活性利用了<em>解引用强制转换(deref coercion)</em>，第 15 章
的[“在函数和方法中使用解引用强制转换”][deref-coercions]<!-- ignore -->一节会介绍
这一特性。

让函数接收字符串切片而非 `String` 的引用，可以在不损失任何功能的情况下，使
API 更通用、更实用：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### 其他切片

顾名思义，字符串切片专用于字符串。但还有一种更通用的切片类型。看看这个数组：

```rust
let a = [1, 2, 3, 4, 5];
```

如同可能需要引用字符串的一部分，我们也可能需要引用数组的一部分，可以这样写：

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

这个切片的类型是 `&[i32]`。它的工作方式与字符串切片相同，存储指向第一个元素
的引用和一个长度。其他各种集合都会用到这类切片。第 8 章讨论向量时会详细介绍
这些集合。

## 小结

所有权、借用和切片这些概念在编译期保证 Rust 程序的内存安全。Rust 语言让你像
使用其他系统编程语言一样控制内存使用；但数据所有者离开作用域时会自动清理数据，
这意味着不必编写和调试额外代码便能获得这种控制能力。

所有权影响 Rust 许多其他部分的工作方式，因此本书后续还会继续讨论这些概念。
接下来进入第 5 章，看看如何使用 `struct` 把多项数据组合在一起。

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods
