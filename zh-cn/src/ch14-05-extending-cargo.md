## 使用自定义命令扩展 Cargo

Cargo 的设计允许你使用新子命令扩展它，而无需修改 Cargo。如果 `$PATH` 中的某个二进制程序名为 `cargo-something`，就可以运行 `cargo something`，像 Cargo 子命令一样运行它。运行 `cargo --list` 时，也会列出这类自定义命令。能够使用 `cargo install` 安装扩展，再像 Cargo 内置工具一样运行，是 Cargo 设计带来的一项极其便利的好处！

## 总结

通过 Cargo 和 [crates.io](https://crates.io/) 分享代码，是 Rust 生态系统能够用于许多不同任务的一部分原因。Rust 标准库小巧而稳定，但 crate 易于分享、使用和改进，并且可以采用不同于语言本身的演进节奏。不要羞于在 [crates.io](https://crates.io/) 上分享对自己有用的代码；它很可能对其他人也有用！
