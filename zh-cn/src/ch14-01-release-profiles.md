## 使用发布配置自定义构建

在 Rust 中，<em>发布配置(release profile)</em>是预先定义、可以自定义且具有不同设置的配置，使程序员能够更细致地控制编译代码的各种选项。每项配置都独立于其他配置进行设置。

Cargo 有两项主要配置：运行 `cargo build` 时使用的 `dev` 配置，以及运行 `cargo build --release` 时使用的 `release` 配置。`dev` 配置具有适合开发的良好默认设置，`release` 配置则具有适合发布构建的默认设置。

你可能在构建输出中见过这些配置名称：

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` 和 `release` 就是编译器使用的两项不同配置。

如果没有在项目的 <em>Cargo.toml</em> 文件中显式添加任何 `[profile.*]` 部分，Cargo 会对每项配置应用默认设置。为任何希望自定义的配置添加 `[profile.*]` 部分，就可以覆盖默认设置的任意子集。例如，以下是 `dev` 和 `release` 配置中 `opt-level` 设置的默认值：

<span class="filename">文件名：Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 设置控制 Rust 对代码应用的优化数量，取值范围为 0 到 3。应用更多优化会延长编译时间，因此在开发期间频繁编译代码时，即使最终代码运行得更慢，也会希望减少优化来加快编译。因此，`dev` 的默认 `opt-level` 是 `0`。准备发布代码时，最好花更多时间编译。发布模式只编译一次，却会多次运行编译后的程序，因此它以更长的编译时间换取运行更快的代码。这就是 `release` 配置的默认 `opt-level` 为 `3` 的原因。

可以在 <em>Cargo.toml</em> 中为某项设置添加不同的值，从而覆盖默认设置。例如，如果希望在开发配置中使用优化级别 1，可以把下面两行添加到项目的 <em>Cargo.toml</em> 文件：

<span class="filename">文件名：Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

这段代码覆盖了默认设置 `0`。现在运行 `cargo build` 时，Cargo 会使用 `dev` 配置的默认设置，再加上我们对 `opt-level` 的自定义。由于把 `opt-level` 设为 `1`，Cargo 应用的优化会多于默认情况，但少于发布构建。

有关每项配置的完整配置选项和默认值列表，请参阅 [Cargo 文档](https://doc.rust-lang.org/cargo/reference/profiles.html)。
