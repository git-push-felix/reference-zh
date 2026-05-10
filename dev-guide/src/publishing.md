# 发布流程

将参考手册内容纳入 [Rust 发布][rust release]并发布到网站的流程如下：

1. 更改合入此仓库。
2. [Triagebot](https://forge.rust-lang.org/triagebot/doc-updates.html) 将每两周自动将此仓库同步到 [rust-lang/rust]。参考手册在 [rust-lang/rust] 中作为[子模块](https://github.com/rust-lang/rust/tree/master/src/doc)进行跟踪。
   - 这将在 [rust-lang/rust] 上打开一个需要合并的 PR，可能需要几天时间。
3. 在 UTC 午夜，[rust-lang/rust] 默认分支上的内容将成为该 nightly 发布的一部分，并在几小时后发布到 <https://doc.rust-lang.org/nightly/reference/>。
4. 按照 Rust 的[发布流程](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html)，每 6 周，nightly 会提升为 beta（<https://doc.rust-lang.org/beta/reference/>），再过 6 周，它会被提升为稳定版（<https://doc.rust-lang.org/stable/reference/>）。

[rust release]: https://doc.rust-lang.org/reference/#rust-releases
[rust-lang/rust]: https://github.com/rust-lang/rust/
