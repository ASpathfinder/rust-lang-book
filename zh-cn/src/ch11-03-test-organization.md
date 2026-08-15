## 测试组织

正如本章开头提到的，测试是一门复杂的学科，不同的人会使用不同术语和组织方式。Rust 社区主要从两个类别看待测试：单元测试和集成测试。<em>单元测试(unit test)</em>规模小且更专注，每次隔离测试一个模块，并且可以测试私有接口。<em>集成测试(integration test)</em>完全位于库的外部，以其他外部代码使用库的相同方式使用代码，只使用公有接口，而且每个测试可能会检验多个模块。

编写这两类测试都很重要，可以确保库的各个部分无论单独工作还是协同工作，都符合预期。

### 单元测试

单元测试的目的是把每个代码单元与其余代码隔离开来测试，以便快速定位代码在哪些地方符合或不符合预期。单元测试会放在 <em>src</em> 目录下，与被测代码位于同一文件中。惯例是在每个文件中创建名为 `tests` 的模块来包含测试函数，并使用 `cfg(test)` 标注该模块。

#### `tests` 模块与 `#[cfg(test)]`

`tests` 模块上的 `#[cfg(test)]` 标注告诉 Rust：只在运行 `cargo test` 时编译并运行测试代码，运行 `cargo build` 时不要这样做。当只想构建库时，这可以节省编译时间；因为测试不会包含在最终编译产物中，也能节省空间。你会看到，因为集成测试位于不同目录，所以不需要 `#[cfg(test)]` 标注。不过，由于单元测试与代码位于同一文件，需要使用 `#[cfg(test)]` 指定它们不应包含在编译结果中。

回想本章第一节中生成新的 `adder` 项目时，Cargo 为我们生成了以下代码：

<span class="filename">文件名：src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

在自动生成的 `tests` 模块上，`cfg` 属性是<em>配置(configuration)</em>的缩写，它告诉 Rust 只有在给定某个配置选项时才包含后面的条目。在这里，配置选项是 `test`，由 Rust 提供，用于编译和运行测试。通过使用 `cfg` 属性，只有主动使用 `cargo test` 运行测试时，Cargo 才会编译测试代码。除了带有 `#[test]` 标注的函数，这还包括模块中可能存在的所有辅助函数。

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-private-functions"></a>

#### 私有函数测试

测试社区对于是否应该直接测试私有函数存在争论，其他语言也会让测试私有函数变得困难或不可能。无论遵循哪种测试理念，Rust 的私有性规则都允许测试私有函数。考虑示例 11-12 中包含私有函数 `internal_adder` 的代码。

<Listing number="11-12" file-name="src/lib.rs" caption="测试私有函数">

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

请注意，`internal_adder` 函数没有标记为 `pub`。测试只是 Rust 代码，`tests` 模块也只是另一个模块。正如[“引用模块树中条目的路径”][paths]中讨论的，子模块中的条目可以使用其祖先模块中的条目。在这个测试中，我们使用 `use super::*` 把 `tests` 模块父模块的所有条目引入作用域，随后测试便可以调用 `internal_adder`。如果你认为不应该测试私有函数，Rust 也不会强迫你这样做。

### 集成测试

在 Rust 中，集成测试完全位于库的外部。它们以其他代码使用库的相同方式使用库，也就是说，只能调用属于库公有 API 的函数。其目的是测试库的多个部分能否正确协同工作。单独工作正常的代码单元在集成时可能出现问题，因此覆盖集成代码的测试也很重要。要创建集成测试，首先需要一个 <em>tests</em> 目录。

#### <em>tests</em> 目录

我们在项目目录的顶层、<em>src</em> 旁边创建 <em>tests</em> 目录。Cargo 知道要在此目录中查找集成测试文件。随后可以创建任意多个测试文件，Cargo 会把每个文件编译为独立的 crate。

让我们创建一个集成测试。保持示例 11-12 中的代码仍位于 <em>src/lib.rs</em> 文件，新建 <em>tests</em> 目录，并创建名为 <em>tests/integration_test.rs</em> 的文件。目录结构应该如下所示：

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

把示例 11-13 中的代码输入 <em>tests/integration_test.rs</em> 文件。

<Listing number="11-13" file-name="tests/integration_test.rs" caption="`adder` crate 中函数的集成测试">

```rust,ignore
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

<em>tests</em> 目录中的每个文件都是独立 crate，因此需要把库引入每个测试 crate 的作用域。为此，我们在代码顶部添加 `use adder::add_two;`；单元测试中不需要这样做。

不需要使用 `#[cfg(test)]` 标注 <em>tests/integration_test.rs</em> 中的任何代码。Cargo 会特别对待 <em>tests</em> 目录，只在运行 `cargo test` 时编译其中的文件。现在运行 `cargo test`：

```console
{{#include ../../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

输出的三个部分分别包含单元测试、集成测试和文档测试。请注意，如果某个部分中的任何测试失败，后续部分就不会运行。例如，如果单元测试失败，就不会有集成测试和文档测试的输出，因为只有在所有单元测试都通过时，才会运行这些测试。

单元测试的第一部分与我们一直看到的内容相同：每个单元测试各有一行（其中一个是示例 11-12 中添加的、名为 `internal` 的测试），然后是一行单元测试摘要。

集成测试部分以 `Running tests/integration_test.rs` 行开头。接下来，集成测试中的每个测试函数各占一行；在 `Doc-tests adder` 部分开始之前，会有一行集成测试结果摘要。

每个集成测试文件都有自己的部分，因此如果在 <em>tests</em> 目录中添加更多文件，输出中就会出现更多集成测试部分。

仍然可以把特定集成测试函数的名称作为实参传给 `cargo test`，运行该函数。要运行某个集成测试文件中的所有测试，请使用 `cargo test` 的 `--test` 实参，后跟文件名称：

```console
{{#include ../../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

该命令只运行 <em>tests/integration_test.rs</em> 文件中的测试。

#### 集成测试中的子模块

随着集成测试不断增加，可能希望在 <em>tests</em> 目录中创建更多文件来组织它们；例如，可以按所测试的功能对测试函数分组。前面提到，<em>tests</em> 目录中的每个文件都会编译成独立 crate，这有助于创建相互独立的作用域，更贴近最终用户使用 crate 的方式。不过，这意味着 <em>tests</em> 目录中的文件与 <em>src</em> 中的文件行为不同；第 7 章介绍了如何把代码拆分为模块和文件。

如果有一组辅助函数要在多个集成测试文件中使用，并尝试按照第 7 章[“把模块拆分到不同文件”][separating-modules-into-files]一节的步骤把它们提取到公共模块中，<em>tests</em> 目录中文件的不同行为最为明显。例如，如果创建 <em>tests/common.rs</em> 并在其中放入名为 `setup` 的函数，就可以向 `setup` 添加希望从多个测试文件中的多个测试函数调用的代码：

<span class="filename">文件名：tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

再次运行测试时，测试输出中会出现 <em>common.rs</em> 文件的新部分，即使该文件不包含任何测试函数，而且我们从未调用 `setup` 函数：

```console
{{#include ../../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

我们不希望 `common` 出现在测试结果中，并显示 `running 0 tests`；只想与其他集成测试文件共享一些代码。为了避免 `common` 出现在测试输出中，我们不创建 <em>tests/common.rs</em>，而是创建 <em>tests/common/mod.rs</em>。现在，项目目录如下：

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

这是第 7 章[“其他文件路径”][alt-paths]中提到、Rust 也能理解的旧命名约定。以这种方式命名文件，会告诉 Rust 不要把 `common` 模块视为集成测试文件。把 `setup` 函数代码移入 <em>tests/common/mod.rs</em> 并删除 <em>tests/common.rs</em> 文件后，测试输出中的相应部分就不会再出现。<em>tests</em> 目录子目录中的文件不会被编译为独立 crate，也不会在测试输出中拥有自己的部分。

创建 <em>tests/common/mod.rs</em> 后，可以把它作为模块用于任意集成测试文件。下面是在 <em>tests/integration_test.rs</em> 中的 `it_adds_two` 测试里调用 `setup` 函数的例子：

<span class="filename">文件名：tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

请注意，`mod common;` 声明与示例 7-21 中展示的模块声明相同。随后，在测试函数中可以调用 `common::setup()` 函数。

#### 二进制 crate 的集成测试

如果项目是只包含 <em>src/main.rs</em> 而没有 <em>src/lib.rs</em> 的二进制 crate，就无法在 <em>tests</em> 目录中创建集成测试，并通过 `use` 语句把 <em>src/main.rs</em> 文件中定义的函数引入作用域。只有库 crate 才会暴露可供其他 crate 使用的函数；二进制 crate 旨在独立运行。

这也是为什么提供二进制文件的 Rust 项目通常有一个简单的 <em>src/main.rs</em> 文件，用来调用位于 <em>src/lib.rs</em> 文件中的逻辑。采用这种结构，集成测试就<em>可以</em>使用 `use` 测试库 crate，使重要功能可用。如果重要功能正常工作，<em>src/main.rs</em> 文件中的少量代码也会正常工作，而且无需测试这少量代码。

## 总结

Rust 的测试功能提供了一种指定代码应如何工作的方式，从而确保即使做出修改，代码仍会按预期工作。单元测试分别检验库的不同部分，并且可以测试私有实现细节。集成测试检查库的多个部分是否正确协同工作，并使用库的公有 API，以外部代码使用库的相同方式测试代码。尽管 Rust 的类型系统和所有权规则有助于防止某些种类的 bug，测试对于减少与代码预期行为有关的逻辑 bug 仍然十分重要。

让我们结合本章和前面各章学到的知识，完成一个项目！

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
