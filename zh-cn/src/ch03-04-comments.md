## 注释

所有程序员都希望代码易于理解，但有时仍需要额外说明。这时可以在源代码中留下
<em>注释(comment)</em>；编译器会忽略它们，阅读代码的人却可能从中受益。

简单注释如下：

```rust
// hello, world
```

Rust 的惯用注释以两个斜杠开始，一直延续到行末。多行注释需要每行都写 `//`：

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

注释也能放在含有代码的行末：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

但更常见的是把注释放在所说明代码上方的独立行：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust 还有文档注释，将在第 14 章[“将 crate 发布到 Crates.io”][publishing]<!-- ignore -->一节讨论。

[publishing]: ch14-02-publishing-to-crates-io.html
