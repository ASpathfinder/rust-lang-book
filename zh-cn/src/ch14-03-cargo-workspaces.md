## Cargo 工作区

第 12 章构建了一个包含二进制 crate 和库 crate 的软件包。随着项目发展，你可能发现库 crate 持续变大，希望进一步把软件包拆分为多个库 crate。Cargo 提供了名为<em>工作区(workspace)</em>的功能，可帮助管理共同开发的多个相关软件包。

### 创建工作区

工作区是一组共享同一个 <em>Cargo.lock</em> 和输出目录的软件包。让我们使用工作区创建一个项目——会采用简单代码，以便专注于工作区结构。工作区有多种组织方式，这里只展示一种常见方式。我们的工作区会包含一个二进制程序和两个库。提供主要功能的二进制程序会依赖这两个库。一个库提供 `add_one` 函数，另一个库提供 `add_two` 函数。这三个 crate 都属于同一个工作区。首先为工作区创建新目录：

```console
$ mkdir add
$ cd add
```

接着，在 <em>add</em> 目录中创建配置整个工作区的 <em>Cargo.toml</em> 文件。该文件没有 `[package]` 部分，而是以允许向工作区添加成员的 `[workspace]` 部分开头。我们还把 `resolver` 值设为 `"3"`，明确让工作区使用 Cargo 最新且最佳版本的<em>解析器算法(resolver algorithm)</em>：

<span class="filename">文件名：Cargo.toml</span>

```toml
{{#include ../../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

接下来，在 <em>add</em> 目录中运行 `cargo new`，创建 `adder` 二进制 crate：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

在工作区内部运行 `cargo new`，还会自动把新创建的软件包添加到工作区 <em>Cargo.toml</em> 中 `[workspace]` 定义的 `members` 键：

```toml
{{#include ../../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

此时，可以运行 `cargo build` 构建工作区。<em>add</em> 目录中的文件应该如下所示：

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

工作区在顶层只有一个 <em>target</em> 目录，用来存放编译产物；`adder` 软件包没有自己的 <em>target</em> 目录。即使从 <em>adder</em> 目录内运行 `cargo build`，编译产物仍会进入 <em>add/target</em>，而不是 <em>add/adder/target</em>。Cargo 这样组织工作区的 <em>target</em> 目录，是因为工作区中的 crate 旨在彼此依赖。如果每个 crate 都有自己的 <em>target</em> 目录，每个 crate 就必须重新编译工作区中的其他 crate，把产物放入自己的 <em>target</em> 目录。通过共享一个 <em>target</em> 目录，crate 可以避免不必要的重复构建。

### 在工作区中创建第二个软件包

接下来，在工作区中再创建一个成员软件包，并命名为 `add_one`。生成名为 `add_one` 的新库 crate：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

现在，顶层 <em>Cargo.toml</em> 会在 `members` 列表中包含 <em>add_one</em> 路径：

<span class="filename">文件名：Cargo.toml</span>

```toml
{{#include ../../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

<em>add</em> 目录现在应该包含以下目录和文件：

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

在 <em>add_one/src/lib.rs</em> 文件中添加 `add_one` 函数：

<span class="filename">文件名：add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

现在，可以让包含二进制程序的 `adder` 软件包依赖包含库的 `add_one` 软件包。首先，需要向 <em>adder/Cargo.toml</em> 添加对 `add_one` 的路径依赖。

<span class="filename">文件名：adder/Cargo.toml</span>

```toml
{{#include ../../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo 不会假定工作区中的 crate 彼此依赖，因此需要明确指定依赖关系。

接下来，在 `adder` crate 中使用来自 `add_one` crate 的 `add_one` 函数。打开 <em>adder/src/main.rs</em> 文件，修改 `main` 函数，像示例 14-7 那样调用 `add_one` 函数。

<Listing number="14-7" file-name="adder/src/main.rs" caption="在 `adder` crate 中使用 `add_one` 库 crate">

```rust,ignore
{{#rustdoc_include ../../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

在顶层 <em>add</em> 目录中运行 `cargo build`，构建工作区！

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

要从 <em>add</em> 目录运行二进制 crate，可以对 `cargo run` 使用 `-p` 实参和软件包名称，指定要运行工作区中的哪个软件包：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

这会运行依赖 `add_one` crate 的 <em>adder/src/main.rs</em> 中的代码。

<!-- Old headings. Do not remove or links may break. -->

<a id="depending-on-an-external-package-in-a-workspace"></a>

### 依赖外部软件包

请注意，工作区只在顶层有一个 <em>Cargo.lock</em> 文件，而不是每个 crate 的目录中都有一个。这可以确保所有 crate 对全部依赖使用相同版本。如果分别向 <em>adder/Cargo.toml</em> 和 <em>add_one/Cargo.toml</em> 文件添加 `rand` 软件包，Cargo 会把两者解析到同一个 `rand` 版本，并记录在唯一的 <em>Cargo.lock</em> 中。让工作区中的所有 crate 使用相同依赖，意味着它们始终彼此兼容。让我们把 `rand` crate 添加到 <em>add_one/Cargo.toml</em> 文件的 `[dependencies]` 部分，以便在 `add_one` crate 中使用 `rand`：

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">文件名：add_one/Cargo.toml</span>

```toml
{{#include ../../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

现在，可以向 <em>add_one/src/lib.rs</em> 文件添加 `use rand;`，再在 <em>add</em> 目录中运行 `cargo build` 构建整个工作区，以引入并编译 `rand` crate。由于没有引用引入作用域的 `rand`，会得到一条警告：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.10.1
   --snip--
   Compiling rand v0.10.1
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` (part of `#[warn(unused)]`) on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

现在，顶层 <em>Cargo.lock</em> 包含 `add_one` 对 `rand` 依赖的信息。不过，虽然工作区中的某个位置使用了 `rand`，除非也把 `rand` 添加到其他 crate 的 <em>Cargo.toml</em> 文件，否则不能在这些 crate 中使用它。例如，如果在 `adder` 软件包的 <em>adder/src/main.rs</em> 文件中添加 `use rand;`，就会得到错误：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

要修复这个问题，请编辑 `adder` 软件包的 <em>Cargo.toml</em> 文件，指出 `rand` 也是它的依赖。构建 `adder` 软件包会把 `rand` 添加到 <em>Cargo.lock</em> 中 `adder` 的依赖列表，但不会下载 `rand` 的额外副本。只要工作区中使用 `rand` 软件包的每个 crate 指定相互兼容的 `rand` 版本，Cargo 就会确保它们使用相同版本，从而节省空间，并确保工作区中的 crate 相互兼容。

如果工作区中的 crate 指定同一依赖的不兼容版本，Cargo 会分别解析每个版本，但仍会尝试解析出尽可能少的版本。

### 向工作区添加测试

为了进一步改进，让我们在 `add_one` crate 中添加对 `add_one::add_one` 函数的测试：

<span class="filename">文件名：add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

现在，在顶层 <em>add</em> 目录中运行 `cargo test`。在这种结构的工作区中运行 `cargo test`，会运行工作区内所有 crate 的测试：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

输出的第一部分表明 `add_one` crate 中的 `it_works` 测试通过。下一部分表明 `adder` crate 中没有找到测试，最后一部分表明 `add_one` crate 中没有找到文档测试。

还可以从顶层目录运行工作区中某个特定 crate 的测试：使用 `-p` 标志，并指定要测试的 crate 名称：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

输出表明 `cargo test` 只运行了 `add_one` crate 的测试，没有运行 `adder` crate 的测试。

如果把工作区中的 crate 发布到 [crates.io](https://crates.io/)，需要分别发布每个 crate。与 `cargo test` 类似，可以使用 `-p` 标志并指定要发布的 crate 名称，发布工作区中的某个特定 crate。

作为额外练习，请以与 `add_one` crate 类似的方式，向该工作区添加 `add_two` crate！

随着项目增长，请考虑使用工作区：与一个庞大的代码块相比，它让你可以使用更小、更易理解的组件。此外，如果 crate 经常同时修改，把它们保留在工作区中可以更容易地协调 crate 之间的变更。
