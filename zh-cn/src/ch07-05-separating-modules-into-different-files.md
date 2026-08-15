## 把模块拆分到不同文件

目前本章的示例都在一个文件中定义多个模块。模块变大后，可以把定义移入独立文件，
使代码更易浏览。

从包含多个餐厅模块的示例 7-17 开始。我们不再把所有模块定义在 crate 根文件中，
而是提取到不同文件。本例 crate 根是 <em>src/lib.rs</em>，但同样的方法也适用于以
<em>src/main.rs</em> 为根的二进制 crate。

首先把 `front_of_house` 提取到自己的文件。删除该模块花括号内的代码，只保留
`mod front_of_house;` 声明，使 <em>src/lib.rs</em> 包含示例 7-21 的代码。注意，
在示例 7-22 创建 <em>src/front_of_house.rs</em> 前，代码还无法编译。

<Listing number="7-21" file-name="src/lib.rs" caption="声明模块体位于 <em>src/front_of_house.rs</em> 的 `front_of_house` 模块">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

接着，把原来位于花括号中的代码放入新文件 <em>src/front_of_house.rs</em>，如示例
7-22 所示。编译器在 crate 根遇到名为 `front_of_house` 的模块声明，所以知道要
到这个文件查找。

<Listing number="7-22" file-name="src/front_of_house.rs" caption="<em>src/front_of_house.rs</em> 中 `front_of_house` 模块内部的定义">

```rust,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

模块树中只需用 `mod` 声明加载文件<em>一次</em>。编译器知道文件属于项目，并根据
`mod` 语句的位置知道代码在模块树中的位置后，其他文件应通过模块声明处的路径
引用其中代码，正如[“引用模块树中条目的路径”][paths]<!-- ignore -->一节所述。
换言之，`mod` <em>不是</em>其他编程语言中可能见过的“include”操作。

接下来把 `hosting` 提取到自己的文件。过程略有不同，因为它是 `front_of_house`
的子模块，而非根模块的子模块。它的文件应放在以模块树祖先命名的新目录中，本例
是 <em>src/front_of_house</em>。

先修改 <em>src/front_of_house.rs</em>，只保留 `hosting` 模块的声明：

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

然后创建 <em>src/front_of_house</em> 目录和 <em>hosting.rs</em>，保存 `hosting` 中的定义：

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

如果把 <em>hosting.rs</em> 放在 <em>src</em> 目录，编译器会认为其中代码属于在 crate 根
声明的 `hosting` 模块，而不是 `front_of_house` 的子模块。编译器查找模块代码的
规则，让目录和文件能够较为贴切地对应模块树。

<a id="alternate-file-paths"></a>

> ### 其他文件路径
>
> 前面介绍的是 Rust 编译器最惯用的文件路径，不过 Rust 也支持较旧的风格。对于
> crate 根中声明的 `front_of_house`，编译器会在以下位置查找：
>
> - <em>src/front_of_house.rs</em>（前面介绍的方式）；
> - <em>src/front_of_house/mod.rs</em>（仍受支持的旧式路径）。
>
> 对于 `front_of_house` 的子模块 `hosting`，编译器会查找：
>
> - <em>src/front_of_house/hosting.rs</em>（前面介绍的方式）；
> - <em>src/front_of_house/hosting/mod.rs</em>（仍受支持的旧式路径）。
>
> 同一模块同时使用两种风格会产生编译器错误。同一项目中的不同模块混用两种风格
> 是允许的，但可能让浏览项目的人感到困惑。
>
> 使用 <em>mod.rs</em> 风格的主要缺点，是项目最终可能出现许多同名文件；在编辑器中
> 同时打开时很容易混淆。

现在每个模块的代码都移入了独立文件，但模块树保持不变。即使定义位于不同文件，
`eat_at_restaurant` 中的函数调用也无须修改。这种技巧让模块变大后能迁移到新文件。

<em>src/lib.rs</em> 中的 `pub use crate::front_of_house::hosting` 同样没有改变，
`use` 也不影响哪些文件会作为 crate 的一部分编译。`mod` 关键字声明模块，Rust
会在与模块同名的文件中查找该模块的代码。

## 小结

Rust 可以把包拆分为多个 crate，再把 crate 拆分为模块，使一个模块能引用另一
模块中定义的条目。引用可以使用绝对或相对路径；用 `use` 把路径引入作用域后，
可以在该作用域多次使用较短路径。模块代码默认私有，但可以用 `pub` 公开定义。

下一章将介绍标准库提供的一些集合数据结构，让你在组织整齐的代码中使用。

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
