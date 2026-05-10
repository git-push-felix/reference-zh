# 引言

感谢你对贡献 **Rust 参考手册**的关注。本文档概述了如何为参考手册做出贡献，并作为编辑者和评审者的指南。

为参考手册提供帮助有几种方式：评论参考手册、编辑参考手册、修正错误信息、添加示例和术语表条目，以及为 Rust 中新的或尚无文档的特性编写文档。

我们鼓励你阅读参考手册的[引言][introduction]，以熟悉参考手册预期包含的内容类型及其使用的约定。

## 评论参考手册

这是最简单的贡献方式。当你阅读参考手册时，如果发现令人困惑、错误或缺失的内容，可以针对参考手册提交 issue 来说明你的关切。

## 编辑参考手册

笔误和错误的链接时不时会出现。如果你发现了它们，欢迎提交 PR 来修复。

## 添加示例和术语表条目

示例非常棒。许多人只会阅读示例而忽略正文。理想情况下，每个特性的每个方面都应该有一个示例。

同样，参考手册有一个术语表。它不需要解释所有内容或包含所有可能的定义，但确实需要在现有基础上进行扩充。理想情况下，术语表中的条目应该链接到相关文档。

## 添加文档

有很多特性完全没有文档或文档质量很差。这是最困难但最有价值的任务。从 [issue 跟踪器][issue tracker]中选择一个未分配的 issue 并为其编写文档。

在编写过程中，你可能会发现打开一个 [playground] 来测试你所编写的内容会很方便。

你可以酌情从标准库和 Rustonomicon 中获取信息。

请注意，我们不为纯库特性（如线程和 I/O）编写文档，也不编写关于未来 Rust 的文档。文档的编写方式是将 Rust 的当前稳定版本视为最后一个版本。参考手册的 `master` 分支对应于 [rust-lang/rust] `main` 分支（"nightly"）上**稳定**的内容。如果你想编写关于未来 Rust 的内容，你需要 **[不稳定特性手册][unstable]**。

[introduction]: https://doc.rust-lang.org/nightly/reference/introduction.html
[issue tracker]: https://github.com/rust-lang/reference/issues
[playground]: https://play.rust-lang.org/
[rust-lang/rust]: https://github.com/rust-lang/rust/
[unstable]: https://doc.rust-lang.org/nightly/unstable-book/
