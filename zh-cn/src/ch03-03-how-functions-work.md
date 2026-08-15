## 函数

函数在 Rust 代码中随处可见。你已经见过语言中最重要的函数之一：作为许多程序
入口点的 `main`。你也见过用于声明新函数的 `fn` 关键字。

Rust 函数名和变量名的惯用风格是<em>蛇形命名法(snake case)</em>：所有字母小写，
单词间用下划线分隔。下面的程序包含一个函数定义：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Rust 函数定义以 `fn` 开头，后接函数名和一对圆括号；花括号告诉编译器函数体的
起止位置。

输入已定义函数的名称和一对圆括号即可调用它。程序定义了 `another_function`，
所以能在 `main` 中调用。这里把它定义在 `main` <em>之后</em>，放在之前也可以。Rust
不关心函数定义在何处，只要求它位于调用方可见的某个作用域中。

新建二进制项目 <em>functions</em>，把示例放进 <em>src/main.rs</em> 并运行，应该看到：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

代码行按照它们在 `main` 中出现的顺序执行：先打印“Hello, world!”，再调用
`another_function` 并打印它的消息。

### 形参

函数可以定义<em>形参(parameter)</em>，即<em>函数签名(signature)</em>中的特殊变量。函数有
形参时，可以为它们提供具体值。严格来说，这些具体值称为<em>实参(argument)</em>；
不过日常交流中，人们常混用 parameter 和 argument，既指函数定义中的变量，
也指调用函数时传入的具体值。

下面给 `another_function` 添加一个形参：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

运行得到：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

`another_function` 声明一个名为 `x`、类型为 `i32` 的形参。传入 `5` 时，
`println!` 会把 `5` 放到格式字符串中包含 `x` 的花括号位置。

函数签名中<em>必须</em>声明每个形参的类型。这是 Rust 的刻意设计：要求在函数定义中
标注类型，使编译器几乎不需要你在代码其他地方帮助判断类型；知道函数期望哪些
类型后，编译器也能给出更有用的错误信息。

定义多个形参时，用逗号分隔：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

`print_labeled_measurement` 有两个形参：`value` 是 `i32`，`unit_label` 是
`char`。函数打印同时包含二者的文本。替换 <em>functions</em> 项目的代码并运行：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

调用时分别传入 `5` 和 `'h'`，所以输出中包含这些值。

### 语句与表达式

函数体由一系列语句组成，末尾可以有一个表达式。此前的函数没有末尾表达式，但
你已经在语句中见过表达式。Rust 是基于表达式的语言，理解二者差异非常重要：

- <em>语句(statement)</em>是执行某项操作但不返回值的指令。
- <em>表达式(expression)</em>求值得到一个结果值。

用 `let` 创建变量并赋值就是语句。示例 3-1 的 `let y = 6;` 是一条语句。

<Listing number="3-1" file-name="src/main.rs" caption="包含一条语句的 `main` 函数声明">

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

函数定义也是语句，所以上面的整个例子本身也是语句（不过稍后会看到，函数调用
不是语句）。

语句不返回值，所以不能像下面这样把 `let` 语句赋给另一个变量：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

运行会得到：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

`let y = 6` 不返回值，因此没有可供 `x` 绑定的内容。这与 C、Ruby 等赋值操作
会返回赋值结果的语言不同；那些语言可写 `x = y = 6`，Rust 不行。

表达式求值得到值，并组成其余大部分 Rust 代码。`5 + 6` 是求值得到 `11` 的
表达式。表达式也能成为语句的一部分：示例 3-1 中 `let y = 6;` 里的 `6` 就是
表达式。函数调用、宏调用，以及花括号创建的新作用域块也都是表达式：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

其中：

```rust,ignore
{
    let x = 3;
    x + 1
}
```

是求值得到 `4` 的块，这个值在 `let` 中绑定到 `y`。请注意 `x + 1` 末尾没有
分号。表达式不以分号结尾；加上分号会把表达式变成语句，因而不再返回值。

### 带返回值的函数

函数可以把值返回给调用代码。返回值没有名称，但必须在箭头 `->` 后声明类型。
Rust 函数的返回值等同于函数体块中最后一个表达式的值。可以用 `return` 和一个
值提前返回，但多数函数会隐式返回最后的表达式：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

`five` 中没有函数调用、宏甚至 `let`，只有数字 `5`，这在 Rust 中完全有效。
返回类型指定为 `-> i32`。运行得到：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`five` 中的 `5` 是返回值，所以类型是 `i32`。`let x = five();` 使用函数返回值
初始化变量，等同于：

```rust
let x = 5;
```

`five` 没有形参，函数体中的 `5` 不带分号，因为它是要返回其值的表达式。

再看一个例子：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

运行会打印 `The value of x is: 6`。如果在 `x + 1` 后添加分号，把表达式变成
语句呢？

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

编译会报错：

```console
{{#include ../../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

核心错误 `mismatched types` 揭示了问题：`plus_one` 声明返回 `i32`，但语句不
产生值，用单元类型 `()` 表示。因此实际没有返回任何内容，与函数定义矛盾。
Rust 还建议删除分号，这正好可以修复错误。
