## 使用 `Result` 处理可恢复错误

大多数错误并没有严重到需要程序完全停止。有时函数失败的原因很容易理解和应对。例如，尝试打开文件时，如果操作因为文件不存在而失败，你可能希望创建该文件，而不是终止进程。

回想一下第 2 章[“使用 `Result` 处理潜在错误”][handle_failure]中介绍的内容：`Result` 枚举定义了 `Ok` 和 `Err` 两个变体：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` 和 `E` 是<em>泛型类型形参(generic type parameter)</em>；第 10 章会更详细地讨论泛型。现在只需知道，`T` 代表操作成功时 `Ok` 变体中返回值的类型，`E` 代表操作失败时 `Err` 变体中返回错误的类型。由于 `Result` 带有这些泛型类型形参，我们可以在许多不同情形下使用 `Result` 类型及其定义的方法，即使希望返回的成功值和错误值各不相同。

让我们调用一个可能失败、因而返回 `Result` 值的函数。在示例 9-3 中，我们尝试打开一个文件。

<Listing number="9-3" file-name="src/main.rs" caption="打开文件">

```rust
{{#rustdoc_include ../../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

`File::open` 的返回类型是 `Result<T, E>`。在 `File::open` 的实现中，泛型参数 `T` 被具体类型 `std::fs::File` 填充，它代表文件句柄；错误值所使用的 `E` 类型则是 `std::io::Error`。这个返回类型表示，对 `File::open` 的调用可能成功并返回可供读写的文件句柄，也可能失败：例如文件不存在，或者我们没有访问文件的权限。`File::open` 函数既需要一种方式告诉我们操作成功还是失败，同时又要提供文件句柄或错误信息；`Result` 枚举恰好传达了这些信息。

如果 `File::open` 成功，变量 `greeting_file_result` 中的值会是包含文件句柄的 `Ok` 实例；如果失败，则会是包含更多错误类型信息的 `Err` 实例。

我们需要扩展示例 9-3 中的代码，根据 `File::open` 返回的值采取不同操作。示例 9-4 展示了一种使用基础工具处理 `Result` 的方式，也就是第 6 章讨论过的 `match` 表达式。

<Listing number="9-4" file-name="src/main.rs" caption="使用 `match` 表达式处理可能返回的 `Result` 变体">

```rust,should_panic
{{#rustdoc_include ../../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

请注意，与 `Option` 枚举一样，`Result` 枚举及其变体已经由预导入模块引入作用域，因此在 `match` 分支中不必在 `Ok` 和 `Err` 变体前写 `Result::`。

当结果为 `Ok` 时，这段代码会从 `Ok` 变体中取出内部的 `file` 值，再把该文件句柄赋给变量 `greeting_file`。在 `match` 之后，我们便可以使用该文件句柄进行读写。

`match` 的另一个分支处理从 `File::open` 得到 `Err` 值的情况。在这个例子中，我们选择调用 `panic!` 宏。如果当前目录中没有名为 <em>hello.txt</em> 的文件，运行这段代码会看到 `panic!` 宏产生的以下输出：

```console
{{#include ../../listings/ch09-error-handling/listing-09-04/output.txt}}
```

与往常一样，这段输出准确地告诉我们出了什么问题。

### 匹配不同的错误

无论 `File::open` 因何失败，示例 9-4 中的代码都会 `panic!`。然而，我们希望根据不同的失败原因采取不同操作。如果 `File::open` 因为文件不存在而失败，我们希望创建文件并返回新文件的句柄；如果它由于其他原因失败——例如没有打开文件的权限——仍希望代码像示例 9-4 那样 `panic!`。为此，我们添加一个内部 `match` 表达式，如示例 9-5 所示。

<Listing number="9-5" file-name="src/main.rs" caption="以不同方式处理不同种类的错误">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

`File::open` 在 `Err` 变体中返回的值类型是 `io::Error`，这是标准库提供的结构体。该结构体有一个 `kind` 方法，调用它可以得到 `io::ErrorKind` 值。标准库提供的枚举 `io::ErrorKind` 包含代表 `io` 操作可能产生的不同错误种类的变体。我们要使用的是 `ErrorKind::NotFound`，它表示尝试打开的文件尚不存在。因此，我们不仅对 `greeting_file_result` 进行匹配，还会在内部对 `error.kind()` 进行匹配。

内部匹配要检查的条件是 `error.kind()` 返回的值是否为 `ErrorKind` 枚举的 `NotFound` 变体。如果是，就尝试使用 `File::create` 创建文件。不过，由于 `File::create` 也可能失败，内部 `match` 表达式还需要第二个分支。当文件无法创建时，会打印另一条错误信息。外部 `match` 的第二个分支保持不变，所以除了文件缺失错误之外，其他错误都会使程序 panic。

> #### 使用 `match` 处理 `Result<T, E>` 的替代方式
>
> 这里的 `match` 可真多！`match` 表达式非常实用，但也是一种相当基础的工具。第 13 章会介绍闭包，它们会与 `Result<T, E>` 上定义的许多方法配合使用。处理代码中的 `Result<T, E>` 值时，这些方法可能比 `match` 更简洁。
>
> 例如，下面是示例 9-5 中相同逻辑的另一种写法，这次使用闭包和 `unwrap_or_else` 方法：
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> 虽然这段代码的行为与示例 9-5 相同，但它不含任何 `match` 表达式，读起来也更清晰。读完第 13 章后可以再回来看这个例子，并在标准库文档中查阅 `unwrap_or_else` 方法。在处理错误时，还有许多这类方法可以简化庞大且嵌套的 `match` 表达式。

<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### 遇到错误时 panic 的快捷方式

使用 `match` 已经足够好，但可能有些冗长，而且不一定能很好地表达意图。`Result<T, E>` 类型定义了许多辅助方法，用于完成更具体的任务。`unwrap` 方法是一种快捷方法，其实现方式与示例 9-4 中编写的 `match` 表达式相同。如果 `Result` 值是 `Ok` 变体，`unwrap` 会返回 `Ok` 内部的值；如果是 `Err` 变体，`unwrap` 会替我们调用 `panic!` 宏。下面是使用 `unwrap` 的例子：

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

如果在没有 <em>hello.txt</em> 文件的情况下运行这段代码，会看到 `unwrap` 方法调用 `panic!` 所产生的错误信息：

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

类似地，`expect` 方法还允许我们选择 `panic!` 的错误信息。使用 `expect` 而不是 `unwrap`，并提供良好的错误信息，可以传达你的意图，也更容易追踪 panic 的来源。`expect` 的语法如下：

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

我们使用 `expect` 的方式与 `unwrap` 相同：要么返回文件句柄，要么调用 `panic!` 宏。`expect` 调用 `panic!` 时使用的错误信息是传给 `expect` 的参数，而不是 `unwrap` 使用的默认 `panic!` 信息。输出如下：

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

在生产质量的代码中，大多数 Rust 开发者会选择 `expect` 而不是 `unwrap`，并提供更多上下文，说明为何预期该操作总能成功。这样，如果你的假设有朝一日被证明有误，就能获得更多可用于调试的信息。

### 传播错误

当函数的实现调用了可能失败的操作时，可以不在函数内部处理错误，而是把错误返回给调用方，让调用代码决定怎么做。这称为<em>传播错误(propagating errors)</em>，它给予调用代码更多控制权，因为调用方可能拥有更多信息或逻辑，能够决定应如何处理错误，而这些在当前代码的上下文中并不可用。

例如，示例 9-6 展示了一个从文件读取用户名的函数。如果文件不存在或无法读取，这个函数会把相应错误返回给调用它的代码。

<Listing number="9-6" file-name="src/main.rs" caption="使用 `match` 向调用代码返回错误的函数">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

这个函数可以写得短得多，但为了探究错误处理，我们会先手动完成其中的大部分工作，最后再展示更简短的写法。先来看函数的返回类型：`Result<String, io::Error>`。这表示函数返回 `Result<T, E>` 类型的值，其中泛型参数 `T` 被具体类型 `String` 填充，泛型类型 `E` 被具体类型 `io::Error` 填充。

如果这个函数顺利执行成功，调用代码会收到包含 `String` 的 `Ok` 值，也就是函数从文件中读取的 `username`。如果函数遇到任何问题，调用代码会收到包含 `io::Error` 实例的 `Err` 值，其中含有关于问题的更多信息。我们选择 `io::Error` 作为这个函数的错误返回类型，是因为函数体中调用的两个可能失败的操作——`File::open` 函数和 `read_to_string` 方法——恰好都返回这种类型的错误值。

函数体首先调用 `File::open` 函数。然后，我们使用类似示例 9-4 中的 `match` 来处理 `Result` 值。如果 `File::open` 成功，模式变量 `file` 中的文件句柄会成为可变变量 `username_file` 的值，函数继续执行。在 `Err` 情况下，我们不调用 `panic!`，而是使用 `return` 关键字立即从整个函数返回，把来自 `File::open`、此时位于模式变量 `e` 中的错误值作为该函数的错误值传回调用代码。

因此，如果 `username_file` 中已有文件句柄，函数接着在变量 `username` 中创建一个新的 `String`，并对 `username_file` 中的文件句柄调用 `read_to_string` 方法，把文件内容读入 `username`。即使 `File::open` 成功，`read_to_string` 方法仍可能失败，所以它也会返回 `Result`。因此还需要另一个 `match` 来处理这个 `Result`：如果 `read_to_string` 成功，函数便成功了，我们返回用 `Ok` 包裹的 `username`，其中包含从文件读取的用户名。如果 `read_to_string` 失败，我们会用处理 `File::open` 返回值的 `match` 中相同的方式返回错误值。不过，由于这是函数中的最后一个表达式，不需要显式写出 `return`。

调用这段代码的代码随后会处理两种结果：包含用户名的 `Ok` 值，或包含 `io::Error` 的 `Err` 值。如何处理由调用代码决定。例如，如果得到 `Err` 值，它可以调用 `panic!` 使程序崩溃、使用默认用户名，或者从文件以外的地方查找用户名。我们没有足够信息了解调用代码实际要做什么，因此把所有成功或错误信息向上传播，交由它妥善处理。

传播错误的模式在 Rust 中非常常见，因此 Rust 提供了问号运算符 `?` 来简化它。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### `?` 运算符这一快捷方式

示例 9-7 展示了 `read_username_from_file` 的另一种实现，其功能与示例 9-6 相同，但使用了 `?` 运算符。

<Listing number="9-7" file-name="src/main.rs" caption="使用 `?` 运算符向调用代码返回错误的函数">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

放在 `Result` 值之后的 `?`，其工作方式与示例 9-6 中为处理 `Result` 值而定义的 `match` 表达式几乎相同。如果 `Result` 的值是 `Ok`，`Ok` 中的值会成为这个表达式的结果，程序继续执行。如果值是 `Err`，则会像使用了 `return` 关键字一样，从整个函数提前返回 `Err`，从而把错误值传播给调用代码。

示例 9-6 中的 `match` 表达式与 `?` 运算符之间存在一个区别：对错误值使用 `?` 运算符时，错误值会经过标准库 `From` trait 中定义的 `from` 函数，该函数用于把一种类型的值转换为另一种类型。当 `?` 运算符调用 `from` 函数时，收到的错误类型会转换为当前函数返回类型中定义的错误类型。当函数用一种错误类型代表所有可能的失败方式，而函数的不同部分可能因许多不同原因失败时，这一点非常有用。

例如，可以修改示例 9-7 中的 `read_username_from_file` 函数，让它返回我们定义的名为 `OurError` 的自定义错误类型。如果还定义 `impl From<io::Error> for OurError`，以便从 `io::Error` 构造 `OurError` 实例，那么 `read_username_from_file` 函数体中的 `?` 运算符调用就会调用 `from` 并转换错误类型，而无需向函数添加更多代码。

在示例 9-7 的上下文中，`File::open` 调用末尾的 `?` 会把 `Ok` 内部的值返回给变量 `username_file`。如果发生错误，`?` 运算符会立即从整个函数返回，并把所有 `Err` 值交给调用代码。`read_to_string` 调用末尾的 `?` 也是如此。

`?` 运算符消除了大量样板代码，让这个函数的实现更简单。还可以在 `?` 之后立即链式调用方法，进一步缩短代码，如示例 9-8 所示。

<Listing number="9-8" file-name="src/main.rs" caption="在 `?` 运算符之后链式调用方法">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

我们把 `username` 中新 `String` 的创建移到了函数开头，这部分并没有变化。我们不再创建 `username_file` 变量，而是把对 `read_to_string` 的调用直接链接到 `File::open("hello.txt")?` 的结果上。`read_to_string` 调用末尾仍有一个 `?`，并且当 `File::open` 和 `read_to_string` 都成功时，我们仍会返回包含 `username` 的 `Ok` 值，而不是返回错误。其功能仍与示例 9-6 和示例 9-7 相同，只是写法不同且更符合人体工学。

示例 9-9 展示了使用 `fs::read_to_string` 让代码更短的方法。

<Listing number="9-9" file-name="src/main.rs" caption="使用 `fs::read_to_string`，而不是先打开再读取文件">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

把文件读入字符串是一项相当常见的操作，因此标准库提供了方便的 `fs::read_to_string` 函数，它会打开文件、创建新的 `String`、读取文件内容、把内容放入该 `String`，然后返回它。当然，使用 `fs::read_to_string` 就没有机会讲解所有错误处理细节，所以我们先采用了较长的写法。

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### 可以在哪里使用 `?` 运算符

`?` 运算符只能用于返回类型与 `?` 所作用值兼容的函数。这是因为 `?` 运算符被定义为以提前返回值的方式离开函数，与示例 9-6 中定义的 `match` 表达式相同。在示例 9-6 中，`match` 使用的是 `Result` 值，而提前返回的分支返回 `Err(e)` 值。函数的返回类型必须是 `Result`，才能与这个 `return` 兼容。

示例 9-10 展示了：如果在返回类型与 `?` 所作用值类型不兼容的 `main` 函数中使用 `?` 运算符，会得到什么错误。

<Listing number="9-10" file-name="src/main.rs" caption="尝试在返回 `()` 的 `main` 函数中使用 `?`，代码无法编译">

```rust,ignore,does_not_compile
{{#rustdoc_include ../../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

这段代码会打开一个文件，而操作可能失败。`?` 运算符跟在 `File::open` 返回的 `Result` 值之后，但这个 `main` 函数的返回类型是 `()`，而不是 `Result`。编译这段代码时，会得到以下错误信息：

```console
{{#include ../../listings/ch09-error-handling/listing-09-10/output.txt}}
```

该错误指出，只允许在返回 `Result`、`Option` 或其他实现了 `FromResidual` 的类型的函数中使用 `?` 运算符。

要修复错误，有两种选择。一种是在没有任何限制阻止这样做的前提下，修改函数的返回类型，使其与 `?` 运算符所作用的值兼容；另一种是使用 `match` 或某个 `Result<T, E>` 方法，以适当方式处理 `Result<T, E>`。

错误信息还提到，`?` 也可用于 `Option<T>` 值。与对 `Result` 使用 `?` 一样，只能在返回 `Option` 的函数中对 `Option` 使用 `?`。`?` 运算符作用于 `Option<T>` 时的行为，与其作用于 `Result<T, E>` 时相似：如果值为 `None`，函数会在此处提前返回 `None`；如果值为 `Some`，则 `Some` 内部的值会成为表达式的结果，函数继续执行。示例 9-11 展示了一个在给定文本中查找第一行最后一个字符的函数。

<Listing number="9-11" caption="对 `Option<T>` 值使用 `?` 运算符">

```rust
{{#rustdoc_include ../../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

这个函数返回 `Option<char>`，因为那里可能有字符，也可能没有。代码接收字符串切片参数 `text`，并对其调用 `lines` 方法，该方法返回一个遍历字符串各行的迭代器。由于函数想检查第一行，它会对迭代器调用 `next`，取得第一个值。如果 `text` 是空字符串，这次 `next` 调用会返回 `None`；此时我们使用 `?` 停止执行并从 `last_char_of_first_line` 返回 `None`。如果 `text` 不是空字符串，`next` 会返回一个 `Some` 值，其中包含 `text` 第一行的字符串切片。

`?` 会提取字符串切片，我们可以对它调用 `chars`，获得一个遍历其中字符的迭代器。我们关心的是第一行的最后一个字符，因此调用 `last` 返回迭代器的最后一项。这里返回的是 `Option`，因为第一行可能是空字符串；例如 `text` 以空行开头，但其他行含有字符，如 `"\nhi"`。不过，如果第一行有最后一个字符，它就会在 `Some` 变体中返回。中间的 `?` 运算符让我们能够简洁地表达这段逻辑，从而用一行代码实现函数。如果不能对 `Option` 使用 `?` 运算符，就必须通过更多方法调用或 `match` 表达式实现这段逻辑。

请注意：可以在返回 `Result` 的函数中对 `Result` 使用 `?` 运算符，也可以在返回 `Option` 的函数中对 `Option` 使用 `?` 运算符，但不能混用。`?` 运算符不会自动把 `Result` 转换为 `Option`，反之亦然；在这些情况下，可以使用 `Result` 的 `ok` 方法或 `Option` 的 `ok_or` 方法显式完成转换。

到目前为止，我们用过的所有 `main` 函数都返回 `()`。`main` 函数很特殊，因为它是可执行程序的入口点和退出点；为了让程序按预期运行，其返回类型受到一些限制。

幸运的是，`main` 也可以返回 `Result<(), E>`。示例 9-12 使用了示例 9-10 中的代码，但我们把 `main` 的返回类型改为 `Result<(), Box<dyn Error>>`，并在末尾添加返回值 `Ok(())`。现在这段代码可以编译了。

<Listing number="9-12" file-name="src/main.rs" caption="把 `main` 改为返回 `Result<(), E>`，即可对 `Result` 值使用 `?` 运算符">

```rust,ignore
{{#rustdoc_include ../../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

`Box<dyn Error>` 类型是一个 trait 对象，我们会在第 18 章[“使用 trait 对象抽象共同的行为”][trait-objects]中讨论它。现在，可以把 `Box<dyn Error>` 理解为“任何种类的错误”。在错误类型为 `Box<dyn Error>` 的 `main` 函数中，可以对 `Result` 值使用 `?`，因为这允许提前返回任意 `Err` 值。尽管这个 `main` 函数的函数体只会返回 `std::io::Error` 类型的错误，但通过指定 `Box<dyn Error>`，即使以后向 `main` 函数体添加返回其他错误的代码，这个签名仍然正确。

当 `main` 函数返回 `Result<(), E>` 时，如果 `main` 返回 `Ok(())`，可执行程序会以值 `0` 退出；如果返回 `Err` 值，则以非零值退出。用 C 语言编写的可执行程序退出时会返回整数：成功退出的程序返回整数 `0`，出错的程序返回 `0` 以外的整数。为了与这一惯例兼容，Rust 的可执行程序也返回整数。

`main` 函数可以返回任何实现了 [`std::process::Termination` trait][termination] 的类型，该 trait 包含一个返回 `ExitCode` 的 `report` 函数。有关为自定义类型实现 `Termination` trait 的更多信息，请查阅标准库文档。

至此，我们已经讨论了调用 `panic!` 或返回 `Result` 的细节。现在回到如何判断在不同情况下应该使用哪一种方式的问题。

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: https://doc.rust-lang.org/std/process/trait.Termination.html
