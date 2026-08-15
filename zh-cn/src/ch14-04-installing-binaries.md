<!-- Old headings. Do not remove or links may break. -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## 使用 `cargo install` 安装二进制程序

`cargo install` 命令允许你在本地安装和使用二进制 crate。它并非用于替代系统软件包，而是为 Rust 开发者提供一种便利方式，安装他人在 [crates.io](https://crates.io/) 上分享的工具。请注意，只能安装具有<em>二进制目标(binary target)</em>的软件包。二进制目标是在 crate 包含 <em>src/main.rs</em> 文件或另一个被指定为二进制程序的文件时创建的可运行程序；与之相对的是不能独立运行、但适合包含在其他程序中的库目标。通常，crate 的 README 文件会说明它是库、具有二进制目标，还是两者兼有。

通过 `cargo install` 安装的所有二进制程序，都存储在安装根目录的 <em>bin</em> 文件夹中。如果使用 <em>rustup.rs</em> 安装 Rust，而且没有任何自定义配置，该目录就是 <em>$HOME/.cargo/bin</em>。请确保该目录位于 `$PATH` 中，以便运行通过 `cargo install` 安装的程序。

例如，第 12 章提到一个用于搜索文件、名为 `ripgrep` 的 Rust `grep` 工具实现。可以运行以下命令安装 `ripgrep`：

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

输出倒数第二行显示已安装二进制程序的位置和名称；对于 `ripgrep`，名称是 `rg`。只要安装目录如前所述位于 `$PATH` 中，就可以运行 `rg --help`，开始使用这个速度更快、更具 Rust 风格的文件搜索工具！
