## 附录 B：运算符与符号

本附录是 Rust 语法的词汇表，包含单独出现或出现在路径、泛型、trait bound、宏、属性、注释、元组和括号语境中的运算符及其他符号。

### 运算符

表 B-1 列出了 Rust 中的运算符、运算符在上下文中的示例、简短说明，以及该运算符能否重载。如果运算符可以重载，表中还列出了用于重载它的相应 trait。

<span class="caption">表 B-1：运算符</span>

| 运算符 | 示例 | 说明 | 可重载？ |
| --- | --- | --- | --- |
| `!` | `ident!(...)`、`ident!{...}`、`ident![...]` | 宏展开 | |
| `!` | `!expr` | 按位补或逻辑非 | `Not` |
| `!=` | `expr != expr` | 不等比较 | `PartialEq` |
| `%` | `expr % expr` | 算术取余 | `Rem` |
| `%=` | `var %= expr` | 算术取余并赋值 | `RemAssign` |
| `&` | `&expr`、`&mut expr` | 借用 | |
| `&` | `&type`、`&mut type`、`&'a type`、`&'a mut type` | 借用指针类型 | |
| `&` | `expr & expr` | 按位与 | `BitAnd` |
| `&=` | `var &= expr` | 按位与并赋值 | `BitAndAssign` |
| `&&` | `expr && expr` | 短路逻辑与 | |
| `*` | `expr * expr` | 算术乘法 | `Mul` |
| `*=` | `var *= expr` | 算术乘法并赋值 | `MulAssign` |
| `*` | `*expr` | 解引用 | `Deref` |
| `*` | `*const type`、`*mut type` | 原始指针 | |
| `+` | `trait + trait`、`'a + trait` | 复合类型约束 | |
| `+` | `expr + expr` | 算术加法 | `Add` |
| `+=` | `var += expr` | 算术加法并赋值 | `AddAssign` |
| `,` | `expr, expr` | 实参与元素分隔符 | |
| `-` | `- expr` | 算术取负 | `Neg` |
| `-` | `expr - expr` | 算术减法 | `Sub` |
| `-=` | `var -= expr` | 算术减法并赋值 | `SubAssign` |
| `->` | `fn(...) -> type`、<code>&vert;...&vert; -> type</code> | 函数与闭包的返回类型 | |
| `.` | `expr.ident` | 字段访问 | |
| `.` | `expr.ident(expr, ...)` | 方法调用 | |
| `.` | `expr.0`、`expr.1` 等 | 元组索引 | |
| `..` | `..`、`expr..`、`..expr`、`expr..expr` | 右侧不包含的范围字面量 | `PartialOrd` |
| `..=` | `..=expr`、`expr..=expr` | 右侧包含的范围字面量 | `PartialOrd` |
| `..` | `..expr` | 结构体字面量更新语法 | |
| `..` | `variant(x, ..)`、`struct_type { x, .. }` | “以及其余部分”模式绑定 | |
| `...` | `expr...expr` | （已弃用，请改用 `..=`）在模式中：包含端点的范围模式 | |
| `/` | `expr / expr` | 算术除法 | `Div` |
| `/=` | `var /= expr` | 算术除法并赋值 | `DivAssign` |
| `:` | `pat: type`、`ident: type` | 约束 | |
| `:` | `ident: expr` | 结构体字段初始化器 | |
| `:` | `'a: loop {...}` | 循环标签 | |
| `;` | `expr;` | 语句和项的终止符 | |
| `;` | `[...; len]` | 固定大小数组语法的一部分 | |
| `<<` | `expr << expr` | 左移 | `Shl` |
| `<<=` | `var <<= expr` | 左移并赋值 | `ShlAssign` |
| `<` | `expr < expr` | 小于比较 | `PartialOrd` |
| `<=` | `expr <= expr` | 小于或等于比较 | `PartialOrd` |
| `=` | `var = expr`、`ident = type` | 赋值/等价 | |
| `==` | `expr == expr` | 相等比较 | `PartialEq` |
| `=>` | `pat => expr` | 匹配分支语法的一部分 | |
| `>` | `expr > expr` | 大于比较 | `PartialOrd` |
| `>=` | `expr >= expr` | 大于或等于比较 | `PartialOrd` |
| `>>` | `expr >> expr` | 右移 | `Shr` |
| `>>=` | `var >>= expr` | 右移并赋值 | `ShrAssign` |
| `@` | `ident @ pat` | 模式绑定 | |
| `^` | `expr ^ expr` | 按位异或 | `BitXor` |
| `^=` | `var ^= expr` | 按位异或并赋值 | `BitXorAssign` |
| <code>&vert;</code> | <code>pat &vert; pat</code> | 模式备选项 | |
| <code>&vert;</code> | <code>expr &vert; expr</code> | 按位或 | `BitOr` |
| <code>&vert;=</code> | <code>var &vert;= expr</code> | 按位或并赋值 | `BitOrAssign` |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code> | 短路逻辑或 | |
| `?` | `expr?` | 错误传播 | |

### 非运算符符号

以下表格包含所有不作为运算符使用的符号；也就是说，它们的行为不像函数或方法调用。

表 B-2 展示单独出现、可以在多种位置使用的符号。

<span class="caption">表 B-2：独立语法</span>

| 符号 | 说明 |
| --- | --- |
| `'ident` | 具名生命周期或循环标签 |
| 数字后紧跟 `u8`、`i32`、`f64`、`usize` 等 | 特定类型的数值字面量 |
| `"..."` | 字符串字面量 |
| `r"..."`、`r#"..."#`、`r##"..."##` 等 | 原始字符串字面量；不处理转义字符 |
| `b"..."` | 字节字符串字面量；构造字节数组而不是字符串 |
| `br"..."`、`br#"..."#`、`br##"..."##` 等 | 原始字节字符串字面量；原始字符串字面量与字节字符串字面量的组合 |
| `'...'` | 字符字面量 |
| `b'...'` | ASCII 字节字面量 |
| <code>&vert;...&vert; expr</code> | 闭包 |
| `!` | 用于发散函数、始终为空的底类型 |
| `_` | “忽略”模式绑定；也用于提高整数字面量的可读性 |

表 B-3 展示出现在通过模块层次结构访问某个项的路径语境中的符号。

<span class="caption">表 B-3：路径相关语法</span>

| 符号 | 说明 |
| --- | --- |
| `ident::ident` | 命名空间路径 |
| `::path` | 相对于 crate 根的路径（即显式绝对路径） |
| `self::path` | 相对于当前模块的路径（即显式相对路径） |
| `super::path` | 相对于当前模块之父模块的路径 |
| `type::ident`、`<type as trait>::ident` | 关联常量、函数与类型 |
| `<type>::...` | 无法直接命名之类型的关联项（例如 `<&T>::...`、`<[T]>::...` 等） |
| `trait::method(...)` | 通过命名定义方法的 trait 来消除方法调用歧义 |
| `type::method(...)` | 通过命名定义方法的类型来消除方法调用歧义 |
| `<type as trait>::method(...)` | 通过命名 trait 和类型来消除方法调用歧义 |

表 B-4 展示出现在使用泛型类型参数之语境中的符号。

<span class="caption">表 B-4：泛型</span>

| 符号 | 说明 |
| --- | --- |
| `path<...>` | 在类型中为泛型类型指定参数（例如 `Vec<u8>`） |
| `path::<...>`、`method::<...>` | 在表达式中为泛型类型、函数或方法指定参数；通常称为<em>涡轮鱼(turbofish)</em>（例如 `"42".parse::<i32>()`） |
| `fn ident<...> ...` | 定义泛型函数 |
| `struct ident<...> ...` | 定义泛型结构体 |
| `enum ident<...> ...` | 定义泛型枚举 |
| `impl<...> ...` | 定义泛型实现 |
| `for<...> type` | 高阶生命周期 bound |
| `type<ident=type>` | 一个或多个关联类型具有特定赋值的泛型类型（例如 `Iterator<Item=T>`） |

表 B-5 展示出现在使用 trait bound 约束泛型类型参数之语境中的符号。

<span class="caption">表 B-5：Trait Bound 约束</span>

| 符号 | 说明 |
| --- | --- |
| `T: U` | 把泛型参数 `T` 约束为实现 `U` 的类型 |
| `T: 'a` | 泛型类型 `T` 的存续时间必须长于生命周期 `'a`（即该类型不能间接包含生命周期短于 `'a` 的任何引用） |
| `T: 'static` | 泛型类型 `T` 除了 `'static` 引用外，不包含任何借用引用 |
| `'b: 'a` | 泛型生命周期 `'b` 的存续时间必须长于生命周期 `'a` |
| `T: ?Sized` | 允许泛型类型参数为动态大小类型 |
| `'a + trait`、`trait + trait` | 复合类型约束 |

表 B-6 展示出现在调用或定义宏，以及为项指定属性之语境中的符号。

<span class="caption">表 B-6：宏与属性</span>

| 符号 | 说明 |
| --- | --- |
| `#[meta]` | 外部属性 |
| `#![meta]` | 内部属性 |
| `$ident` | 宏替换 |
| `$ident:kind` | 宏元变量 |
| `$(...)...` | 宏重复 |
| `ident!(...)`、`ident!{...}`、`ident![...]` | 宏调用 |

表 B-7 展示创建注释的符号。

<span class="caption">表 B-7：注释</span>

| 符号 | 说明 |
| --- | --- |
| `//` | 行注释 |
| `//!` | 内部行文档注释 |
| `///` | 外部行文档注释 |
| `/*...*/` | 块注释 |
| `/*!...*/` | 内部块文档注释 |
| `/**...*/` | 外部块文档注释 |

表 B-8 展示圆括号的使用语境。

<span class="caption">表 B-8：圆括号</span>

| 符号 | 说明 |
| --- | --- |
| `()` | 空元组（亦称单元），既可作字面量，也可作类型 |
| `(expr)` | 带圆括号的表达式 |
| `(expr,)` | 单元素元组表达式 |
| `(type,)` | 单元素元组类型 |
| `(expr, ...)` | 元组表达式 |
| `(type, ...)` | 元组类型 |
| `expr(expr, ...)` | 函数调用表达式；也用于初始化元组 `struct` 和元组 `enum` 变体 |

表 B-9 展示花括号的使用语境。

<span class="caption">表 B-9：花括号</span>

| 语境 | 说明 |
| --- | --- |
| `{...}` | 块表达式 |
| `Type {...}` | 结构体字面量 |

表 B-10 展示方括号的使用语境。

<span class="caption">表 B-10：方括号</span>

| 语境 | 说明 |
| --- | --- |
| `[...]` | 数组字面量 |
| `[expr; len]` | 包含 `len` 份 `expr` 的数组字面量 |
| `[type; len]` | 包含 `len` 个 `type` 实例的数组类型 |
| `expr[expr]` | 集合索引；可重载（`Index`、`IndexMut`） |
| `expr[..]`、`expr[a..]`、`expr[..b]`、`expr[a..b]` | 伪装成集合切片的集合索引，使用 `Range`、`RangeFrom`、`RangeTo` 或 `RangeFull` 作为“索引” |
