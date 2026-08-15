<a id="storing-utf-8-encoded-text-with-strings"></a>

## 使用字符串存储 UTF-8 编码文本

第 4 章已经讨论过字符串，现在进一步深入。Rust 新手常在字符串上遇到困难，原因
通常有三点：Rust 倾向于暴露潜在错误；字符串是比许多程序员以为的更复杂的数据
结构；以及 UTF-8。来自其他语言时，这些因素的组合可能显得棘手。

这里把字符串作为集合讨论，因为它由字节集合加上一些方法实现；这些方法在把字节
解释为文本时提供实用功能。本节会介绍所有集合都具备的 `String` 操作，例如创建、
更新和读取；也会说明它与其他集合的区别，尤其是人和计算机对 `String` 数据的
理解不同，使索引字符串变得复杂。

<!-- Old headings. Do not remove or links may break. -->

<a id="what-is-a-string"></a>

### 定义字符串

先明确“字符串”的含义。Rust 核心语言只有一种字符串类型：字符串切片 `str`，
通常以借用形式 `&str` 出现。第 4 章介绍过字符串切片，它引用存储在其他位置的
UTF-8 编码字符串数据。例如，字符串字面量存储在程序二进制文件中，所以也是
字符串切片。

`String` 由标准库提供，而不是内置在核心语言中；它是一种可增长、可变、拥有
所有权且采用 UTF-8 编码的字符串类型。Rust 程序员说“字符串”时，可能指 `String`
或 `&str` 字符串切片中的任一种。本节主要讨论 `String`，但两种类型都在标准库中
广泛使用，也都采用 UTF-8 编码。

### 创建新字符串

`String` 实际是字节向量的封装，并附带额外保证、限制和能力，因此许多 `Vec<T>`
操作同样适用于它。例如两者都用 `new` 创建实例，如示例 8-11 所示。

<Listing number="8-11" caption="创建新的空 `String`">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

这行创建名为 `s` 的空字符串，之后可以装入数据。通常会有一些初始数据，此时可用
`to_string` 方法。任何实现 `Display` 特征的类型（包括字符串字面量）都有这个
方法。示例 8-12 给出两个例子。

<Listing number="8-12" caption="使用 `to_string` 从字符串字面量创建 `String`">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

代码创建包含 `initial contents` 的字符串。

也可以用 `String::from` 从字面量创建 `String`。示例 8-13 与示例 8-12 中使用
`to_string` 的代码等价。

<Listing number="8-13" caption="使用 `String::from` 从字符串字面量创建 `String`">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

字符串用途广泛，因此可以使用许多不同的泛型 API，选择很多。有些看似重复，但
各有用途！这里 `String::from` 与 `to_string` 完成相同工作，选择哪一个取决于
风格和可读性。

字符串采用 UTF-8 编码，所以可以包含任何正确编码的数据，如示例 8-14 所示。

<Listing number="8-14" caption="在字符串中存储不同语言的问候语">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

这些都是有效的 `String` 值。

### 更新字符串

与 `Vec<T>` 一样，向 `String` 推入更多数据后，它的大小可以增长，内容可以改变。
还可以方便地用 `+` 运算符或 `format!` 宏连接 `String` 值。

<!-- Old headings. Do not remove or links may break. -->

<a id="appending-to-a-string-with-push_str-and-push"></a>

#### 使用 `push_str` 或 `push` 追加

使用 `push_str` 追加字符串切片，可以增长 `String`，如示例 8-15 所示。

<Listing number="8-15" caption="使用 `push_str` 向 `String` 追加字符串切片">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

执行后 `s` 包含 `foobar`。`push_str` 接收字符串切片，因为我们不一定想取得形参
的所有权。例如示例 8-16 在把 `s2` 的内容追加到 `s1` 后仍要使用 `s2`。

<Listing number="8-16" caption="把字符串切片内容追加到 `String` 后继续使用它">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

如果 `push_str` 取得 `s2` 的所有权，最后一行就不能打印它。但这段代码能按预期
工作！

`push` 接收单个字符并加入 `String`。示例 8-17 用它向字符串添加字母 <em>l</em>。

<Listing number="8-17" caption="使用 `push` 向 `String` 添加一个字符">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

结果 `s` 包含 `lol`。

<!-- Old headings. Do not remove or links may break. -->

<a id="concatenation-with-the--operator-or-the-format-macro"></a>

<a id="concatenating-with--or-format"></a>

#### 使用 `+` 或 `format!` 连接

经常需要组合两个已有字符串。一种方式是使用 `+`，如示例 8-18。

<Listing number="8-18" caption="使用 `+` 把两个 `String` 组合成新的 `String`">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

`s3` 包含 `Hello, world!`。相加后 `s1` 为何失效、为何使用 `s2` 的引用，与 `+`
调用的方法签名有关。`+` 使用 `add`，其签名大致如下：

```rust,ignore
fn add(self, s: &str) -> String {
```

标准库使用泛型和关联类型定义 `add`，这里换成调用时实际使用的具体类型。第 10 章
会介绍泛型。这个签名提供了理解 `+` 细节的线索。

首先，`s2` 前有 `&`，表示把第二个字符串的引用加到第一个字符串上。这由 `add`
的 `s` 形参决定：只能把字符串切片加到 `String`，不能直接相加两个 `String`。
但 `&s2` 是 `&String`，不是签名要求的 `&str`，示例为何能编译？

原因是编译器能把 `&String` 强制转换为 `&str`。调用 `add` 时，Rust 使用解引用
强制转换，在这里把 `&s2` 变为 `&s2[..]`。第 15 章会深入讨论。由于 `add` 不取得
`s` 的所有权，操作后 `s2` 仍然有效。

其次，签名中的 `self` 没有 `&`，所以 `add` 取得其所有权。示例 8-18 的 `s1`
会移入调用，之后不再有效。因此，`let s3 = s1 + &s2;` 虽然看似复制两个字符串
并创建新字符串，实际却是取得 `s1` 的所有权，追加 `s2` 内容的副本，再返回结果
的所有权。看似产生许多复制，实际实现更加高效。

连接多个字符串时，`+` 的写法会变得笨重：

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

此时 `s` 为 `tic-tac-toe`。大量 `+` 和 `"` 让代码难以理解。更复杂的连接可以
改用 `format!`：

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

它同样把 `s` 设为 `tic-tac-toe`。`format!` 与 `println!` 工作方式相似，但不向
屏幕打印，而是返回包含结果的 `String`。这种写法更易读，而且生成的代码使用引用，
不会取得任何形参的所有权。

### 索引字符串

许多语言可以用索引访问字符串中的单个字符，这是一种常见操作。但在 Rust 中尝试
以索引语法访问 `String` 的部分会报错，如示例 8-19。

<Listing number="8-19" caption="尝试对 `String` 使用索引语法">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

代码会产生以下错误：

```console
{{#include ../../listings/ch08-common-collections/listing-08-19/output.txt}}
```

错误说明 Rust 字符串不支持索引。为什么？需要先了解 Rust 如何在内存中存储字符串。

#### 内部表示

`String` 是 `Vec<u8>` 的封装。看看示例 8-14 中正确编码的 UTF-8 字符串，先看：

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

这里 `len` 为 `4`，意味着保存 `"Hola"` 的向量长度为 4 字节；每个字母的 UTF-8
编码占 1 字节。但下面一行可能令人意外（字符串以西里尔大写字母 <em>Ze</em> 开头，
不是数字 3）：

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

你可能认为长度是 12，Rust 的答案却是 24：UTF-8 编码“Здравствуйте”需要 24
字节，其中每个 Unicode 标量值占 2 字节。因此，字符串的字节索引并不总是对应
有效的 Unicode 标量值。看看这段无效代码：

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

`answer` 不会是首字母 `З`。它的 UTF-8 首字节为 `208`，第二字节为 `151`，所以
看起来 `answer` 应为 `208`；但 `208` 本身不是有效字符。请求首字母时，用户大概
不想得到 `208`，可字节索引 0 处只有这些数据。即使只含拉丁字母，用户通常也不想
得到字节值：若 `&"hi"[0]` 有效，它会返回 `104`，而不是 `h`。

为了避免返回意外值并造成难以及时发现的错误，Rust 完全不编译这种代码，在开发
早期阻止误解。

<!-- Old headings. Do not remove or links may break. -->

<a id="bytes-and-scalar-values-and-grapheme-clusters-oh-my"></a>

#### 字节、标量值与字形簇

从 Rust 的角度，UTF-8 字符串有三种相关视角：字节、标量值和<em>字形簇(grapheme
cluster)</em>；最后一种最接近人们所说的“字母”。

用天城文书写的印地语单词“नमस्ते”，存储为以下 `u8` 向量：

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

共 18 字节，这是计算机最终存储数据的方式。如果视为 Rust `char` 类型所表示的
Unicode 标量值，这些字节是：

```text
['न', 'म', 'स', '्', 'त', 'े']
```

这里有六个 `char`，但第四和第六个并非字母，而是无法单独表达含义的变音符号。
最后，如果视为字形簇，会得到人们认为组成该单词的四个字母：

```text
["न", "म", "स्", "ते"]
```

Rust 提供不同方式解释计算机存储的原始字符串数据，让每个程序都能选择所需方式，
无论数据使用哪种人类语言。

Rust 不允许索引 `String` 取得字符还有一个原因：索引操作通常应保持常数时间
O(1)，但 `String` 无法保证这种性能，因为必须从头遍历到索引处，才能确定经过了
多少有效字符。

### 字符串切片

索引字符串往往不是好主意，因为返回类型并不明确：字节值、字符、字形簇还是
字符串切片？如果确实需要用索引创建切片，Rust 要求写得更具体。

不要用单个数字的 `[]`，而是用范围创建包含特定字节的字符串切片：

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

`s` 是包含前 4 字节的 `&str`。每个字符占 2 字节，所以 `s` 为 `Зд`。

如果用 `&hello[0..1]` 只切取字符的部分字节，Rust 会在运行时 panic，就像访问
向量中的无效索引：

```console
{{#include ../../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

用范围创建字符串切片时务必小心，因为可能导致程序崩溃。

<!-- Old headings. Do not remove or links may break. -->

<a id="methods-for-iterating-over-strings"></a>

### 遍历字符串

操作字符串各部分的最佳方式，是明确需要字符还是字节。要取得各个 Unicode 标量
值，使用 `chars`。在“Зд”上调用它会分离并返回两个 `char`，可以遍历访问：

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

输出：

```text
З
д
```

也可以用 `bytes` 返回每个原始字节，这可能更适合某些问题领域：

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

输出组成字符串的 4 个字节：

```text
208
151
208
180
```

请记住，有效 Unicode 标量值可能由多个字节组成。

取得天城文字符串的字形簇很复杂，标准库没有提供这项功能；如有需要，可在
[crates.io](https://crates.io/)<!-- ignore -->找到相应 crate。

<!-- Old headings. Do not remove or links may break. -->

<a id="strings-are-not-so-simple"></a>

### 处理字符串的复杂性

总而言之，字符串很复杂。不同语言对如何向程序员呈现这种复杂性作出了不同选择。
Rust 选择让正确处理 `String` 成为所有程序的默认行为，这意味着程序员需要预先
投入更多思考处理 UTF-8。这种取舍暴露的复杂性比其他语言更多，却避免在开发后期
处理非 ASCII 字符相关错误。

好消息是，标准库基于 `String` 和 `&str` 提供了许多功能，帮助正确处理复杂场景。
请查阅文档中的实用方法，例如搜索字符串的 `contains`，以及用另一个字符串替换
部分内容的 `replace`。

下面转向稍微简单一点的内容：哈希表！
