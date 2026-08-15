## 什么是所有权？

<em>所有权(ownership)</em>是一组管理 Rust 程序如何使用内存的规则。所有程序在运行时
都必须管理其使用计算机内存的方式。有些语言使用<em>垃圾回收(garbage collection)</em>，
在程序运行期间定期查找不再使用的内存；另一些语言则要求程序员显式分配和释放
内存。Rust 采用第三种方式：通过所有权系统管理内存，并由编译器检查一组规则。
如果违反任何规则，程序便无法编译。所有权的任何特性都不会拖慢程序运行。

对许多程序员来说，所有权是个新概念，确实需要一些时间适应。好消息是，随着你
越来越熟悉 Rust 和所有权系统的规则，自然而然地写出安全、高效的代码也会变得
越来越容易。坚持下去！

理解所有权后，你就为理解 Rust 的独特特性打下了坚实基础。本章将通过一种十分
常见的数据结构——字符串——的相关示例来学习所有权。

<a id="the-stack-and-the-heap"></a>

> ### 栈与堆
>
> 许多编程语言并不要求你经常考虑栈和堆。但在 Rust 这样的系统编程语言中，值
> 位于栈还是堆会影响语言的行为，也会影响你为何必须作出某些选择。本章后面会
> 结合栈和堆说明所有权的一些方面，因此先作简要介绍。
>
> <em>栈(stack)</em>和<em>堆(heap)</em>都是代码在运行时可使用的内存区域，但组织方式不同。
> 栈按取得值的顺序存储它们，并按相反顺序移除。这称为<em>后进先出(last in,
> first out, LIFO)</em>。可以想象一叠盘子：增加盘子时放在最上面，需要盘子时也从
> 最上面取走；从中间或底部增减盘子就不太方便。添加数据称为<em>压栈(pushing onto
> the stack)</em>，移除数据称为<em>出栈(popping off the stack)</em>。存储在栈上的所有
> 数据都必须具有已知且固定的大小；编译期大小未知或大小可能变化的数据必须改为
> 存储在堆上。
>
> 堆的组织更为松散：把数据放到堆上时，需要请求一定大小的空间。<em>内存分配器
> (memory allocator)</em>会在堆中找到一块足够大的空闲区域，将其标记为正在使用，
> 并返回一个<em>指针(pointer)</em>，即该位置的地址。这个过程称为<em>在堆上分配
> (allocating on the heap)</em>，有时简称<em>分配(allocating)</em>（把值压入栈不算
> 分配）。因为指针的大小已知且固定，所以可以把指针存放在栈上；但要取得实际
> 数据，就必须沿着指针访问。可以把它想成在餐厅入座：进门后说明人数，接待员
> 找到一张大小合适的空桌并带你过去；迟到的同伴可以询问你坐在哪里，从而找到你。
>
> 压栈比在堆上分配更快，因为分配器无须寻找存放新数据的位置；这个位置始终在
> 栈顶。相比之下，在堆上分配空间需要更多工作，因为分配器必须先找到足以容纳
> 数据的空间，再进行记录，为下一次分配作准备。
>
> 访问堆上的数据通常比访问栈上的数据慢，因为必须沿指针才能到达数据。现代处理器
> 在内存中跳转得越少，速度就越快。继续用餐厅类比：服务员为许多桌客人点餐时，
> 先收完一桌的全部订单，再前往下一桌最为高效；依次在 A 桌、B 桌、A 桌、B 桌
> 各点一份菜会慢得多。同理，处理器处理彼此相邻的数据（如栈上的数据），通常比
> 处理相距较远的数据（堆上的数据可能如此）更高效。
>
> 代码调用函数时，传给函数的值（可能包括指向堆数据的指针）和函数的局部变量会
> 被压入栈。函数结束后，这些值会被弹出栈。
>
> 跟踪代码的哪些部分正在使用堆上的哪些数据、尽量减少堆上的重复数据，以及清理
> 不再使用的堆数据以免耗尽空间，都是所有权要解决的问题。理解所有权以后，你不必
> 经常考虑栈和堆；但知道所有权的主要目的是管理堆数据，有助于理解它为何如此工作。

### 所有权规则

首先看看所有权规则。在分析说明这些规则的示例时，请始终牢记：

- Rust 中的每个值都有一个<em>所有者(owner)</em>。
- 同一时间只能有一个所有者。
- 所有者离开作用域时，该值会被丢弃。

### 变量作用域

我们已经学过 Rust 的基本语法，后续示例将不再完整写出 `fn main() {`。如果你在
跟着练习，请自行把示例放入 `main` 函数中。这样示例会更简洁，让我们专注于实际
细节而非样板代码。

所有权的第一个示例将考察一些变量的作用域。<em>作用域(scope)</em>是程序中某个条目
保持有效的范围。以下面这个变量为例：

```rust
let s = "hello";
```

变量 `s` 指向一个字符串字面量，字符串值被硬编码在程序文本中。从声明的位置
开始，直到当前作用域结束，变量一直有效。示例 4-1 通过注释标出变量 `s` 有效的
范围。

<Listing number="4-1" caption="变量及其有效作用域">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

</Listing>

换句话说，这里有两个重要时刻：

- `s` <em>进入(comes into)</em>作用域时，它开始有效。
- 它一直有效，直到<em>离开(goes out of)</em>作用域。

到这里，作用域与变量有效期之间的关系和其他编程语言类似。现在引入 `String`
类型，在这个认识的基础上继续深入。

### `String` 类型

为了说明所有权规则，我们需要一种比第 3 章[“数据类型”][data-types]<!-- ignore -->
一节介绍的类型更复杂的数据类型。此前介绍的类型大小已知，可以存放在栈上，并在
作用域结束时弹出栈；如果另一部分代码需要在不同作用域中使用同一个值，也可以
快速、轻易地复制一份，得到新的独立实例。但现在我们要考察存储在堆上的数据，
并探索 Rust 如何知道何时清理它们；`String` 类型正是很好的例子。

我们会重点介绍 `String` 中与所有权有关的部分。这些方面也适用于标准库提供或
你自己创建的其他复杂数据类型。[第 8 章][ch8]<!-- ignore -->将讨论 `String` 中与
所有权无关的部分。

前面已经见过<em>字符串字面量(string literal)</em>，也就是将字符串值硬编码进程序。
字符串字面量很方便，却不适用于所有需要文本的场景。原因之一是它不可变；另一个
原因是，并非所有字符串值都能在编写代码时得知。例如，要是我们想接收并保存用户
输入呢？Rust 为这些场景提供了 `String` 类型。它管理在堆上分配的数据，因此能
存储编译期大小未知的文本。可以使用 `from` 函数从字符串字面量创建 `String`：

```rust
let s = String::from("hello");
```

双冒号运算符 `::` 允许我们把这个 `from` 函数放在 `String` 类型的命名空间下，
而不是使用 `string_from` 之类的名称。第 5 章的[“方法”][methods]<!-- ignore -->
一节会进一步讨论这种语法；第 7 章的[“引用模块树中条目的路径”][paths-module-tree]
<!-- ignore -->一节会在介绍模块命名空间时再次讨论。

这种字符串<em>可以</em>修改：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

这里有什么不同？为什么 `String` 可以修改，而字面量不可以？区别在于这两种类型
处理内存的方式。

### 内存与分配

对于字符串字面量，其内容在编译期已知，因此文本会直接硬编码到最终的可执行文件
中。这正是字符串字面量快速而高效的原因。但这些特性只能来自它的不可变性。对于
每段编译期大小未知、运行时大小还可能变化的文本，我们无法预先在二进制文件中
放入一块内存。

为了让 `String` 支持可变、可增长的文本，需要在堆上分配一块编译期大小未知的
内存来保存内容。这意味着：

- 必须在运行时向内存分配器请求内存。
- `String` 使用完毕后，需要通过某种方式把内存归还给分配器。

第一件事由我们完成：调用 `String::from` 时，它的实现会请求所需内存。这在编程
语言中几乎是通用做法。

但第二件事有所不同。在具有<em>垃圾回收器(garbage collector, GC)</em>的语言中，GC
会跟踪并清理不再使用的内存，我们无须操心。在大多数没有 GC 的语言中，我们有
责任判断内存何时不再使用，并像请求内存时那样调用代码显式释放它。正确完成这件
事历来是个困难的编程问题：忘记释放会浪费内存；过早释放会使变量失效；释放两次
同样是错误。每次 `allocate` 必须恰好对应一次 `free`。

Rust 选择了另一条道路：拥有内存的变量离开作用域后，内存会自动归还。下面是
示例 4-1 作用域示例的另一个版本，它使用 `String` 而不是字符串字面量：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

在 `s` 离开作用域时，正好可以把 `String` 所需的内存归还给分配器。变量离开
作用域时，Rust 会替我们调用一个名为 `drop` 的特殊函数；`String` 的作者可以
在其中编写归还内存的代码。Rust 会在右花括号处自动调用 `drop`。

> 注意：在 C++ 中，这种在条目生命周期结束时释放资源的模式有时称为<em>资源获取
> 即初始化(Resource Acquisition Is Initialization, RAII)</em>。如果你使用过 RAII
> 模式，就会熟悉 Rust 的 `drop` 函数。

这种模式深刻影响着 Rust 代码的编写方式。现在看起来可能很简单，但在更复杂的
场景中，当我们希望多个变量使用堆上分配的数据时，代码行为可能出乎意料。下面
考察其中一些场景。

<!-- Old headings. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-move"></a>

<a id="variables-and-data-interacting-with-move"></a>

#### 变量与数据通过移动交互

在 Rust 中，多个变量可以用不同方式与同一份数据交互。示例 4-2 展示了使用整数
的例子。

<Listing number="4-2" caption="把变量 `x` 的整数值赋给 `y`">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

</Listing>

大概可以猜出它的作用：“把值 `5` 绑定到 `x`；然后复制 `x` 中的值并绑定到
`y`。”现在有两个变量 `x` 和 `y`，二者都等于 `5`。实际情况确实如此，因为
整数是大小已知且固定的简单值，这两个值 `5` 都会被压入栈。

再看看 `String` 版本：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

这看起来十分相似，因此可能会以为工作方式也相同：第二行复制 `s1` 中的值并
绑定到 `s2`。但实际情况并非如此。

图 4-1 展示了 `String` 在底层发生的事情。`String` 由左侧所示的三部分组成：
指向保存字符串内容的内存的指针、长度和容量。这组数据存储在栈上；右侧则是保存
内容的堆内存。

<img alt="两个表格：第一个表格表示栈上的 s1，包含长度 5、容量 5，以及指向第二个表格首个值的指针；第二个表格逐字节表示堆上的字符串数据。" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-1：绑定到 `s1`、保存值 `"hello"` 的 `String` 在内存中的表示</span>

长度表示 `String` 内容当前使用了多少字节内存。容量表示 `String` 从分配器取得
的内存总量（以字节为单位）。长度与容量之间的差异很重要，但与这里无关，目前
可以忽略容量。

把 `s1` 赋给 `s2` 时，会复制 `String` 数据，也就是复制栈上的指针、长度和
容量。指针所指向的堆数据并不会被复制。换句话说，内存中的数据表示如图 4-2。

<img alt="三个表格：分别表示栈上字符串 s1 和 s2 的两个表格都指向堆上的同一份字符串数据。" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-2：变量 `s2` 在内存中的表示，它复制了 `s1` 的指针、长度和容量</span>

它<em>不会</em>呈现为图 4-3，那是 Rust 同时复制堆数据时的内存布局。如果 Rust 这样
做，当堆数据很大时，`s2 = s1` 操作的运行时开销可能非常昂贵。

<img alt="四个表格：两个表格表示 s1 和 s2 的栈数据，每个表格分别指向堆上各自的一份字符串数据。" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-3：如果 Rust 同时复制堆数据，`s2 = s1` 可能产生的另一种结果</span>

前面说过，变量离开作用域时，Rust 会自动调用 `drop` 函数并清理该变量的堆内存。
但图 4-2 中的两个数据指针指向同一位置。这就产生了问题：`s2` 和 `s1` 离开
作用域时，二者都会尝试释放同一块内存。这称为<em>二次释放(double free)</em>错误，
是前面提到的内存安全缺陷之一。释放内存两次可能破坏内存，进而造成安全漏洞。

为保证内存安全，执行 `let s2 = s1;` 后，Rust 会认为 `s1` 不再有效。因此，
`s1` 离开作用域时，Rust 无须释放任何内容。尝试在创建 `s2` 后使用 `s1`，看看
会发生什么；这段代码无法工作：

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

Rust 会阻止你使用已经失效的引用，因此会得到类似下面的错误：

```console
{{#include ../../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

如果你在其他语言中听说过<em>浅拷贝(shallow copy)</em>和<em>深拷贝(deep copy)</em>，
只复制指针、长度和容量而不复制数据，听起来就像浅拷贝。但 Rust 还会使第一个
变量失效，因此这种操作不叫浅拷贝，而称为<em>移动(move)</em>。在这个示例中，我们
会说 `s1` 被<em>移动(moved)</em>到了 `s2`。实际发生的情况如图 4-4。

<img alt="三个表格：表示栈上字符串 s1 和 s2 的表格都指向堆上的同一份字符串数据。s1 表格呈灰色，因为 s1 已经失效；只有 s2 能访问堆数据。" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-4：`s1` 失效后在内存中的表示</span>

问题解决了！现在只有 `s2` 有效，它离开作用域时独自释放内存，事情就结束了。

此外，这里还隐含着一项设计选择：Rust 永远不会自动创建数据的“深”拷贝。因此，
可以认为任何<em>自动</em>复制在运行时的开销都很低。

#### 作用域与赋值

反过来，作用域、所有权以及通过 `drop` 函数释放内存之间的关系同样成立。给现有
变量赋予一个全新的值时，Rust 会立即调用 `drop` 并释放原值的内存。例如：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-04b-replacement-drop/src/main.rs:here}}
```

首先声明变量 `s`，并把它绑定到值为 `"hello"` 的 `String`。紧接着创建一个值为
`"ahoy"` 的新 `String` 并赋给 `s`。此时已经没有任何内容指向堆上的原值。图 4-5
展示了此时的栈和堆数据：

<img alt="一个表示栈上字符串值的表格指向堆上的第二段字符串数据 ahoy；原字符串数据 hello 呈灰色，因为已经无法访问。" src="img/trpl04-05.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-5：初始值被整体替换后在内存中的表示</span>

因此，原字符串会立即离开作用域。Rust 对它运行 `drop`，立刻释放其内存。最后
打印出的值会是 `"ahoy, world!"`。

<!-- Old headings. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-clone"></a>

<a id="variables-and-data-interacting-with-clone"></a>

#### 变量与数据通过克隆交互

如果确实希望深度复制 `String` 的堆数据，而不只是栈数据，可以使用常见的
`clone` 方法。第 5 章会讨论方法语法，不过方法是许多编程语言的常见特性，你
可能已经见过。

下面是使用 `clone` 方法的示例：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

这段代码可以正常工作，并明确产生图 4-3 所示的行为，其中的堆数据<em>确实</em>会被复制。

看到 `clone` 调用时，就知道这里会执行某些任意代码，而且开销可能很大。它直观
地表明正在发生某种不同的操作。

<a id="stack-only-data-copy"></a>

#### 只在栈上的数据：复制

还有一个细节尚未讨论。下面使用整数的代码（部分曾出现在示例 4-2 中）有效且能
正常工作：

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

但它似乎与刚学到的内容矛盾：代码没有调用 `clone`，`x` 却仍然有效，并未移动到
`y` 中。

原因是整数等编译期大小已知的类型完全存储在栈上，因此复制实际值的速度很快。
这意味着创建变量 `y` 后，没有理由阻止 `x` 继续有效。换句话说，这里的深拷贝
和浅拷贝没有区别；调用 `clone` 与通常的浅拷贝不会产生不同结果，所以可以省略。

Rust 提供了一个名为 `Copy` <em>特征(trait)</em>的特殊标注，可用于整数这类存储在
栈上的类型（[第 10 章][traits]<!-- ignore -->会进一步讨论特征）。如果一个类型实现了 `Copy` 特征，使用
该类型的变量就不会发生移动，而是被轻易复制，因此赋给另一个变量后仍然有效。

如果某个类型或它的任何组成部分实现了 `Drop` 特征，Rust 就不允许再为该类型
标注 `Copy`。如果类型需要在值离开作用域时执行特殊操作，而我们又为它添加
`Copy` 标注，就会得到编译期错误。要了解如何为自己的类型添加 `Copy` 标注以
实现该特征，请参阅附录 C 的[“可派生特征”][derivable-traits]<!-- ignore -->。

哪些类型实现了 `Copy` 特征？可以查阅相应类型的文档进行确认。一般而言，任何
一组简单标量值都可以实现 `Copy`；需要分配内存或本身是某种资源的类型则不能。
下面列出一些实现了 `Copy` 的类型：

- 所有整数类型，例如 `u32`。
- 布尔类型 `bool`，其值为 `true` 和 `false`。
- 所有浮点类型，例如 `f64`。
- 字符类型 `char`。
- 元组，但其中只能包含同样实现了 `Copy` 的类型。例如，`(i32, i32)` 实现了
  `Copy`，而 `(i32, String)` 没有。

### 所有权与函数

把值传给函数的机制与给变量赋值类似。向函数传递变量时，同赋值一样，会发生移动
或复制。示例 4-3 带有一些注释，说明变量何时进入和离开作用域。

<Listing number="4-3" file-name="src/main.rs" caption="标注了所有权和作用域的函数">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

</Listing>

如果在调用 `takes_ownership` 后尝试使用 `s`，Rust 会抛出编译期错误。这些静态
检查可以保护我们免于犯错。尝试在 `main` 中添加使用 `s` 和 `x` 的代码，看看
可以在哪里使用它们，所有权规则又会在哪里阻止你。

### 返回值与作用域

返回值也可以转移所有权。示例 4-4 展示了一个返回某个值的函数，并带有与示例
4-3 类似的注释。

<Listing number="4-4" file-name="src/main.rs" caption="转移返回值的所有权">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

</Listing>

变量的所有权每次都遵循同一模式：把值赋给另一个变量会移动它。包含堆数据的变量
离开作用域时，除非数据的所有权已经移动到另一个变量，否则 `drop` 会清理该值。

虽然这样可行，但每次调用函数都要取得所有权再返回所有权，未免有些繁琐。如果
只想让函数使用一个值而不取得所有权，该怎么办？传入的任何内容如果还想再次使用，
就必须传回来；而函数体产生的其他数据可能也要返回，这十分麻烦。

Rust 确实允许使用元组返回多个值，如示例 4-5 所示。

<Listing number="4-5" file-name="src/main.rs" caption="返回形参的所有权">

```rust
{{#rustdoc_include ../../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

</Listing>

但对于一个本应常见的概念，这套流程仪式感太强，工作量也很大。幸好 Rust 提供了
一种无需转移所有权便能使用值的特性：引用。

[data-types]: ch03-02-data-types.html#data-types
[ch8]: ch08-02-strings.html
[traits]: ch10-02-traits.html
[derivable-traits]: appendix-03-derivable-traits.html
[methods]: ch05-03-method-syntax.html#methods
[paths-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
