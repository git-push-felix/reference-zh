# mdbook-spec

[`mdbook-spec`] 是一个 mdBook 预处理器，为参考手册添加了若干功能。它提供：

- [语法图][grammar diagrams]的解析和生成。
  - [语法产生式自动链接][Automatic grammar production links]。
  - [语法摘要附录][grammar summary appendix]的生成。
- [标准库自动链接][Automatic standard library links]。
- [规则名称][rule names]的处理。
  - 名称验证。
  - 将规则名称转换为链接。
  - [规则链接自动引用][Automatic rule link references]。
  - [规则测试链接][links to rule tests]的生成。
  - [测试摘要][test summary]的生成。
- 对[提示块][admonitions]的支持。

## 环境变量

`mdbook-spec` 使用的一些环境变量在 **[构建参考手册][Building the Reference]** 中有详细说明：

- [`SPEC_RELATIVE`] --- 可以设置以将外部书籍链接到在线站点。
- [`SPEC_DENY_WARNINGS`] --- 是否应将警告视为错误。
- [`SPEC_RUST_ROOT`] --- [`rust-lang/rust`] GitHub 仓库检出的路径。用于测试链接。

[`mdbook-spec`]: https://github.com/rust-lang/reference/tree/HEAD/tools/mdbook-spec
[`rust-lang/rust`]: https://github.com/rust-lang/rust
[`SPEC_DENY_WARNINGS`]: building.md#SPEC_DENY_WARNINGS
[`SPEC_RELATIVE`]: building.md#SPEC_RELATIVE
[`SPEC_RUST_ROOT`]: building.md#SPEC_RUST_ROOT
[admonitions]: ../formatting/admonitions.md
[Automatic grammar production links]: ../grammar.md#automatic-linking
[Automatic rule link references]: ../links.md#rule-links
[Automatic standard library links]: ../links.md#standard-library-links
[Building the Reference]: building.md
[grammar diagrams]: ../grammar.md
[grammar summary appendix]: https://doc.rust-lang.org/nightly/reference/grammar.html
[links to rule tests]: ../rules/test-annotations.md
[rule names]: ../rules/index.md
[test summary]: https://doc.rust-lang.org/nightly/reference/test-summary.html
