<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## 使用测试驱动开发添加功能

现在，搜索逻辑位于 <em>src/lib.rs</em> 中，与 `main` 函数分离，为代码核心功能编写测试容易多了。我们可以使用不同实参直接调用函数并检查返回值，而无需从命令行调用二进制程序。

本节会使用<em>测试驱动开发(test-driven development, TDD)</em>过程，通过以下步骤向 `minigrep` 程序添加搜索逻辑：

1. 编写一个失败的测试并运行它，确保它因预期原因失败。
2. 编写或修改刚好足以让新测试通过的代码。
3. 重构刚添加或修改的代码，确保测试继续通过。
4. 从第 1 步开始重复！

TDD 只是众多软件编写方式之一，但有助于推动代码设计。在编写使测试通过的代码之前先写测试，有助于在整个过程中保持较高的测试覆盖率。

我们会以测试驱动的方式实现一项功能：在文件内容中实际搜索查询字符串，并生成与查询匹配的行列表。该功能将添加到名为 `search` 的函数中。

### 编写失败的测试

与[第 11 章][ch11-anatomy]一样，我们会在 <em>src/lib.rs</em> 中添加包含测试函数的 `tests` 模块。测试函数指定我们希望 `search` 函数拥有的行为：它接收查询和要搜索的文本，只返回文本中包含查询的行。示例 12-15 展示了该测试。

<Listing number="12-15" file-name="src/lib.rs" caption="为尚未实现、但希望 `search` 函数拥有的功能创建失败测试">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

这个测试搜索字符串 `"duct"`。待搜索文本有三行，其中只有一行包含 `"duct"`（请注意，开头双引号之后的反斜杠告诉 Rust 不要在该字符串字面量的内容开头加入换行符）。我们断言 `search` 函数返回的值只包含预期的那一行。

如果现在运行测试，它会失败，因为 `unimplemented!` 宏会以信息“not implemented”发生 panic。按照 TDD 原则，我们只迈出一小步：添加刚好足以让测试调用函数时不 panic 的代码，把 `search` 函数定义为始终返回空向量，如示例 12-16 所示。随后测试应该能够编译，但会失败，因为空向量与包含 `"safe, fast, productive."` 一行的向量不匹配。

<Listing number="12-16" file-name="src/lib.rs" caption="定义刚好足以让调用不发生 panic 的 `search` 函数">

```rust,noplayground
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

现在来讨论为何需要在 `search` 签名中定义显式生命周期 `'a`，并把该生命周期用于 `contents` 实参和返回值。回想[第 10 章][ch10-lifetimes]：生命周期形参指定哪个实参的生命周期与返回值生命周期相关。这里，我们表示返回的向量应包含引用实参 `contents` 中切片的字符串切片（而不是引用实参 `query`）。

换句话说，我们告诉 Rust，`search` 函数返回的数据将与通过 `contents` 实参传给 `search` 函数的数据存活同样久。这一点很重要！切片<em>所引用的</em>数据需要保持有效，引用才能有效；如果编译器假定我们创建的是 `query` 而不是 `contents` 的字符串切片，安全检查就会出错。

如果忘记生命周期标注并尝试编译该函数，会得到以下错误：

```console
{{#include ../../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust 不知道输出需要两个形参中的哪一个，所以必须明确告诉它。请注意，帮助文本建议为所有形参和输出类型指定相同的生命周期形参，但这是不正确的！由于 `contents` 是包含全部文本的形参，而我们希望返回该文本中匹配的部分，因此知道只有 `contents` 形参应该通过生命周期语法与返回值相关联。

其他编程语言不要求你在签名中把实参与返回值连接起来，但随着时间推移，这种做法会变得更容易。可以把这个例子与第 10 章[“使用生命周期验证引用”][validating-references-with-lifetimes]一节中的示例比较。

### 编写使测试通过的代码

目前，测试因为我们始终返回空向量而失败。为了修复它并实现 `search`，程序需要执行以下步骤：

1. 遍历内容中的每一行。
2. 检查该行是否包含查询字符串。
3. 如果包含，把该行添加到要返回的值列表。
4. 如果不包含，什么也不做。
5. 返回匹配的结果列表。

让我们依次完成每一步，从遍历各行开始。

#### 使用 `lines` 方法遍历各行

Rust 提供了一个很有帮助、用于逐行遍历字符串的方法，名字恰如其分地叫作 `lines`，其用法如示例 12-17 所示。请注意，这段代码尚不能编译。

<Listing number="12-17" file-name="src/lib.rs" caption="遍历 `contents` 中的每一行">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

`lines` 方法返回迭代器。[第 13 章][ch13-iterators]会深入讨论迭代器。不过回想[示例 3-5][ch3-iter]，你已经见过这种迭代器用法：我们使用 `for` 循环和迭代器，对集合中的每一项运行一些代码。

#### 在每一行中搜索查询

接下来，检查当前行是否包含查询字符串。幸运的是，字符串有一个很有帮助、可以替我们完成这项工作的 `contains` 方法！按照示例 12-18，在 `search` 函数中添加对 `contains` 方法的调用。请注意，这段代码仍然无法编译。

<Listing number="12-18" file-name="src/lib.rs" caption="添加功能，检查行是否包含 `query` 中的字符串">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

目前，我们正在逐步构建功能。要让代码编译，还需要按照函数签名中的承诺，从函数体返回一个值。

#### 存储匹配的行

为了完成这个函数，需要一种存储想要返回的匹配行的方式。可以在 `for` 循环之前创建可变向量，并调用 `push` 方法把 `line` 存入向量。`for` 循环之后返回向量，如示例 12-19 所示。

<Listing number="12-19" file-name="src/lib.rs" caption="存储匹配的行，以便返回它们">

```rust,ignore
{{#rustdoc_include ../../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

现在，`search` 函数应该只返回包含 `query` 的行，测试应该通过。来运行测试：

```console
{{#include ../../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

测试通过了，所以我们知道它能正常工作！

此时，可以考虑重构搜索函数实现的机会，同时保持测试通过和功能不变。搜索函数的代码并不差，但没有利用迭代器的一些实用功能。[第 13 章][ch13-iterators]会详细探索迭代器，并回到这个例子，看看如何改进它。

现在，整个程序应该都能工作了！先使用一个应该恰好返回 Emily Dickinson 诗中一行的单词来试试：<em>frog</em>。

```console
{{#include ../../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

不错！再试一个会匹配多行的单词，比如 <em>body</em>：

```console
{{#include ../../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

最后，确保搜索诗中任何位置都没有出现的单词时，不会得到任何行，例如 <em>monomorphization</em>：

```console
{{#include ../../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

太棒了！我们构建了经典工具的迷你版本，并学到了许多有关应用程序结构的知识。我们还了解了一些文件输入输出、生命周期、测试和命令行解析的内容。

为了完成这个项目，我们会简要展示如何使用环境变量，以及如何打印到标准错误；编写命令行程序时，两者都很有用。

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
