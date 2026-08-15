## 附录 A：关键字

以下列表包含 Rust 语言保留供当前或未来使用的关键字。因此，它们不能用作标识符（原始标识符除外，我们会在[“原始标识符”][raw-identifiers]<!-- ignore -->一节讨论）。<em>标识符(identifier)</em>是函数、变量、形参、结构体字段、模块、crate、常量、宏、静态值、属性、类型、trait 或生命周期的名称。

[raw-identifiers]: #raw-identifiers

### 当前使用的关键字

下面列出当前使用的关键字及其功能。

- <strong>`as`</strong>：执行基本类型转换；消除歧义，指定包含某个项的具体 trait；或者在 `use` 语句中重命名项。
- <strong>`async`</strong>：返回 `Future`，而不是阻塞当前线程。
- <strong>`await`</strong>：暂停执行，直到 `Future` 的结果就绪。
- <strong>`break`</strong>：立即退出循环。
- <strong>`const`</strong>：定义常量项或常量原始指针。
- <strong>`continue`</strong>：继续下一次循环迭代。
- <strong>`crate`</strong>：在模块路径中引用 crate 根。
- <strong>`dyn`</strong>：动态分派到 trait 对象。
- <strong>`else`</strong>：`if` 和 `if let` 控制流结构的后备分支。
- <strong>`enum`</strong>：定义枚举。
- <strong>`extern`</strong>：链接外部函数或变量。
- <strong>`false`</strong>：布尔值 false 字面量。
- <strong>`fn`</strong>：定义函数或函数指针类型。
- <strong>`for`</strong>：循环遍历迭代器中的项、实现 trait，或指定高阶生命周期。
- <strong>`if`</strong>：根据条件表达式的结果进行分支。
- <strong>`impl`</strong>：实现固有功能或 trait 功能。
- <strong>`in`</strong>：`for` 循环语法的一部分。
- <strong>`let`</strong>：绑定变量。
- <strong>`loop`</strong>：无条件循环。
- <strong>`match`</strong>：将值与模式匹配。
- <strong>`mod`</strong>：定义模块。
- <strong>`move`</strong>：让闭包取得其全部捕获项的所有权。
- <strong>`mut`</strong>：表示引用、原始指针或模式绑定中的可变性。
- <strong>`pub`</strong>：表示结构体字段、`impl` 块或模块的公开可见性。
- <strong>`ref`</strong>：按引用绑定。
- <strong>`return`</strong>：从函数返回。
- <strong>`Self`</strong>：当前正在定义或实现之类型的类型别名。
- <strong>`self`</strong>：方法主体所作用的对象，或当前模块。
- <strong>`static`</strong>：全局变量，或持续整个程序执行期间的生命周期。
- <strong>`struct`</strong>：定义结构体。
- <strong>`super`</strong>：当前模块的父模块。
- <strong>`trait`</strong>：定义 trait。
- <strong>`true`</strong>：布尔值 true 字面量。
- <strong>`type`</strong>：定义类型别名或关联类型。
- <strong>`union`</strong>：定义[联合体][union]<!-- ignore -->；只有在联合体声明中使用时才是关键字。
- <strong>`unsafe`</strong>：表示不安全代码、函数、trait 或实现。
- <strong>`use`</strong>：把符号引入作用域。
- <strong>`where`</strong>：表示约束类型的子句。
- <strong>`while`</strong>：根据表达式结果进行条件循环。

[union]: https://doc.rust-lang.org/reference/items/unions.html

### 保留供未来使用的关键字

以下关键字目前还没有任何功能，但 Rust 将它们保留供未来可能使用：

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

<a id="raw-identifiers"></a>

### 原始标识符

<em>原始标识符(raw identifier)</em>是一种语法，让你能够在通常不允许使用关键字的位置使用关键字。使用原始标识符时，在关键字前添加 `r#` 前缀。

例如，`match` 是一个关键字。如果尝试编译下面这个使用 `match` 作为名称的函数：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

会得到以下错误：

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

错误表明不能把关键字 `match` 用作函数标识符。要把 `match` 用作函数名，需要像下面这样使用原始标识符语法：

<span class="filename">文件名：src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

这段代码可以编译且不会出现任何错误。请注意，函数定义中的函数名以及 `main` 中调用函数的位置都有 `r#` 前缀。

原始标识符允许你选择任何词作为标识符，即使它恰好是保留关键字。这既给了我们更多选择标识符名称的自由，也让我们能够与使用另一种语言编写、而这些词在该语言中不是关键字的程序集成。此外，原始标识符还允许你使用以不同于当前 crate 之 Rust 版本编写的库。例如，`try` 在 2015 版中不是关键字，但在 2018、2021 和 2024 版中是。如果依赖一个使用 2015 版编写、带有 `try` 函数的库，就需要用原始标识符语法——这里是 `r#try`——从采用后续版本的代码中调用该函数。有关版本的更多信息，请参阅[附录 E][appendix-e]<!-- ignore -->。

[appendix-e]: appendix-05-editions.html
