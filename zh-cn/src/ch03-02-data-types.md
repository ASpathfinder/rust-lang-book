<a id="data-types"></a>

## 数据类型

Rust 中的每个值都属于某种<em>数据类型(data type)</em>，它告诉 Rust 正在指定哪类
数据，以便 Rust 知道该如何处理。我们将介绍两类数据类型：标量和复合类型。

Rust 是<em>静态类型(statically typed)</em>语言，必须在编译期知道所有变量的类型。
编译器通常可以根据值及其用法推断所需类型。存在多种可能类型时，例如第 2 章
[“比较猜测与秘密数字”][comparing-the-guess-to-the-secret-number]<!-- ignore -->一节使用 `parse` 把 `String` 转成数值类型，就必须添加类型标注：

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

如果不添加 `: u32`，Rust 会显示下面的错误，表示编译器需要更多信息才能知道
我们想使用哪种类型：

```console
{{#include ../../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

其他数据类型会使用不同的类型标注。

### 标量类型

<em>标量(scalar)</em>类型表示单个值。Rust 有四种主要标量类型：整数、浮点数、布尔
值和字符。你可能在其他语言中见过它们，下面看看它们在 Rust 中如何工作。

<a id="integer-types"></a>

#### 整数类型

<em>整数(integer)</em>是没有小数部分的数。第 2 章使用过 `u32`：它表示值应为占用
32 位空间的无符号整数（有符号整数类型以 `i` 而不是 `u` 开头）。表 3-1
列出 Rust 内置整数类型，声明整数值时可以选择任意一种。

<span class="caption">表 3-1：Rust 中的整数类型</span>

| 长度 | 有符号 | 无符号 |
| --- | --- | --- |
| 8 位 | `i8` | `u8` |
| 16 位 | `i16` | `u16` |
| 32 位 | `i32` | `u32` |
| 64 位 | `i64` | `u64` |
| 128 位 | `i128` | `u128` |
| 取决于体系结构 | `isize` | `usize` |

每种类型都有明确大小，并分为有符号或无符号。<em>有符号(signed)</em>和<em>无符号
(unsigned)</em>表示数字能否为负，即是否需要携带符号。就像在纸上写数字：符号
有意义时会写正号或负号；可以确定为正时则不写。Rust 使用[二进制补码
(two's complement)][twos-complement]<!-- ignore -->存储有符号数。

使用 <em>n</em> 位的有符号类型可以存储 −(2<sup>n − 1</sup>) 到 2<sup>n − 1</sup> − 1
（含端点）的数。因此 `i8` 范围是 −(2<sup>7</sup>) 到 2<sup>7</sup> − 1，即
−128 到 127。无符号类型范围是 0 到 2<sup>n</sup> − 1，因此 `u8` 是 0 到 255。

`isize` 和 `usize` 取决于程序运行计算机的体系结构：64 位体系结构上为 64 位，
32 位体系结构上为 32 位。

整数文字值可以写成表 3-2 的任意形式。可能属于多种数值类型的文字值允许添加
`57u8` 这样的类型后缀。数字也可用 `_` 作视觉分隔，例如 `1_000` 与 `1000`
数值相同。

<span class="caption">表 3-2：Rust 中的整数文字值</span>

| 数字文字值 | 示例 |
| --- | --- |
| 十进制 | `98_222` |
| 十六进制 | `0xff` |
| 八进制 | `0o77` |
| 二进制 | `0b1111_0000` |
| 字节（仅限 `u8`） | `b'A'` |

不确定整数类型时，Rust 默认值通常是良好起点：整数默认为 `i32`。使用 `isize`
或 `usize` 的主要场景，是对某种集合进行索引。

> ##### 整数溢出
>
> 假设一个 `u8` 变量能保存 0 到 255。若尝试把它改成范围外的 256，就会发生
> <em>整数溢出(integer overflow)</em>，可能产生两种行为。调试模式编译时，Rust 会
> 加入溢出检查；发生溢出会让程序在运行时 <em>panic</em>，即带着错误退出。第 9 章
> [“使用 `panic!` 处理不可恢复错误”][unrecoverable-errors-with-panic]<!-- ignore -->一节会深入讨论 panic。
>
> 使用 `--release` 在发布模式编译时，Rust 不会加入导致 panic 的整数溢出
> 检查，而是执行<em>二进制补码回绕(two's complement wrapping)</em>。超过类型最大
> 值的数会“绕回”最小值：`u8` 中 256 变成 0，257 变成 1，依此类推。程序不
> 会 panic，但变量值很可能不符合预期。依赖这种回绕行为被视为错误。
>
> 要显式处理溢出可能性，可以使用标准库为原始数值类型提供的方法族：
>
> - `wrapping_*` 方法（如 `wrapping_add`）在所有编译模式中执行回绕。
> - `checked_*` 方法在溢出时返回 `None`。
> - `overflowing_*` 方法返回值以及表示是否溢出的布尔值。
> - `saturating_*` 方法把结果限定在类型的最小值或最大值。

#### 浮点类型

Rust 还有两个用于<em>浮点数(floating-point number)</em>的原始类型 `f32` 和 `f64`，
分别占 32 位和 64 位。默认是 `f64`，因为在现代 CPU 上，它与 `f32` 速度大致
相同，却具有更高精度。所有浮点类型都有符号。

浮点数示例：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

浮点数按照 IEEE-754 标准表示。

#### 数值运算

Rust 为所有数值类型支持常见的基本数学运算：加、减、乘、除和取余。整数除法
会向零截断到最近的整数。下面是在 `let` 语句中使用各项运算的方式：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

这些语句中的每个表达式都使用一个数学运算符，求值得到单个值，再绑定到变量。
[附录 B][appendix_b]<!-- ignore -->列出 Rust 提供的全部运算符。

#### 布尔类型

与多数编程语言一样，Rust 的<em>布尔(Boolean)</em>类型只有 `true` 和 `false` 两个值，
占一个字节，用 `bool` 指定：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

布尔值主要用于 `if` 表达式等条件判断。[“控制流”][control-flow]<!-- ignore -->一节会介绍 Rust 的 `if`。

#### 字符类型

Rust 的 `char` 是语言中最基本的字母类型。声明示例如下：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

`char` 文字值使用单引号，字符串文字值使用双引号。Rust 的 `char` 占 4 字节，
表示 Unicode 标量值，因此远不止能表示 ASCII：带重音字母、中日韩文字、emoji
和零宽空格都是有效的 `char`。Unicode 标量值范围包括 `U+0000` 到 `U+D7FF`
及 `U+E000` 到 `U+10FFFF`。不过，“字符”在 Unicode 中并不是一个真正的概念，
因此人们对字符的直觉可能与 Rust 的 `char` 不符。第 8 章[“使用字符串存储 UTF-8 编码文本”][strings]<!-- ignore -->一节会详细讨论这个主题。

### 复合类型

<em>复合类型(compound type)</em>可以把多个值组合成一个类型。Rust 有两种原始复合
类型：元组和数组。

<a id="the-tuple-type"></a>

#### 元组类型

<em>元组(tuple)</em>是把多个不同类型的值组合成一个复合类型的通用方式。元组长度
固定，声明后不能增长或缩短。

在圆括号中写下逗号分隔的值即可创建元组。每个位置都有类型，不同位置的类型
不必相同。下面添加了可选类型标注：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

`tup` 绑定整个元组，因为元组被视为单一复合元素。要取出各个值，可以用模式
匹配解构元组：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

程序先创建元组并绑定到 `tup`，再用带 `let` 的模式把它变成 `x`、`y`、`z`
三个变量。这称为<em>解构(destructuring)</em>，因为它把单个元组拆成三部分。最后打印
`y` 的值 `6.4`。

也可以用句点 `.` 加要访问值的索引，直接访问元组元素：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

程序创建元组 `x`，再用各自索引访问每个元素。与多数语言一样，首个索引是 0。

不含任何值的元组有一个特殊名称：<em>单元(unit)</em>。值和对应类型都写作 `()`，
表示空值或空返回类型。表达式没有返回其他值时，会隐式返回单元值。

#### 数组类型

另一种保存多个值的集合是<em>数组(array)</em>。与元组不同，数组中每个元素的类型
必须相同；与一些其他语言的数组不同，Rust 数组长度固定。

数组值写成方括号中的逗号分隔列表：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

需要让数据像此前类型一样分配在<em>栈(stack)</em>而不是<em>堆(heap)</em>上，或确保元素数量固定
时，数组很有用。[第 4 章][stack-and-heap]<!-- ignore -->会讨论栈和堆。不过，数组不如 vector 灵活。vector 是
标准库提供的类似集合，其内容位于堆上，所以可以增长或缩短。不确定使用数组还是
vector 时，通常应该选 vector；[第 8 章][vectors]<!-- ignore -->会详细介绍。

明确知道元素数量不变时，数组更合适。例如，程序中的月份名称始终有 12 个，
所以可能使用数组：

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

数组类型用方括号书写，其中依次为元素类型、分号和元素数量：

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

`i32` 是每个元素的类型，分号后的 `5` 表示数组含五个元素。

还可以在方括号中指定初始值、分号和长度，让所有元素拥有同一个初始值：

```rust
let a = [3; 5];
```

数组 `a` 含 `5` 个元素，初始值全为 `3`，等同于更冗长的
`let a = [3, 3, 3, 3, 3];`。

<!-- Old headings. Do not remove or links may break. -->
<a id="accessing-array-elements"></a>

#### 访问数组元素

数组是一块大小已知且固定、可分配在栈上的连续内存。可以用索引访问元素：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

`first` 获得 `1`，因为它位于索引 `[0]`；`second` 从 `[1]` 获得 `2`。

#### 无效的数组元素访问

看看访问超出数组末尾的元素会发生什么。运行下面类似第 2 章猜数字游戏的代码，
从用户处取得数组索引：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

代码可以编译。使用 `cargo run` 运行并输入 `0` 到 `4`，程序会打印对应元素。
如果输入超出数组末尾的数字，例如 `10`，会看到：

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

在索引操作中使用无效值时发生运行时错误，程序带着错误消息退出，没有执行最后的
`println!`。通过索引访问元素时，Rust 检查索引是否小于数组长度；大于或等于
长度便会 panic。这项检查必须在运行时进行，因为编译器不可能预知用户以后运行
代码时输入什么值。

这体现了 Rust 的内存安全原则。许多底层语言不会作这种检查，错误索引可能访问
无效内存。Rust 会立即退出，而不是允许无效内存访问后继续运行，从而保护程序。
第 9 章会进一步讨论错误处理，以及如何编写既不会 panic、也不允许无效内存
访问的清晰安全代码。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.html
