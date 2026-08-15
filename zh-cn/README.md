# 《Rust 程序设计语言》简体中文试译版

本目录包含与英文书稿独立维护的简体中文 mdBook 源文件。试译内容以英文仓库
提交 `917544888a55e4da7109bdba8c88c893c0da70f4` 为基准，对应 Rust
1.97.0 和 Rust 2024 Edition。

当前范围包括标题页、前言、引言和第 1～3 章。尚未翻译的章节不会出现在中文目录
中；试译内容对这些章节的引用暂时链接至 Rust 官方英文在线版本。

## 构建

请在仓库根目录运行：

```console
$ mdbook build zh-cn
```

生成结果位于 `book-zh-cn/`。可以直接在浏览器中打开
`book-zh-cn/index.html` 阅读。

运行书中示例测试：

```console
$ mdbook test zh-cn
```

翻译和术语约定参见 [TRANSLATION_GUIDE.md](TRANSLATION_GUIDE.md)。
