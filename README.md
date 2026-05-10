# Rust 参考手册（中文翻译）

这是 [The Rust Reference](https://github.com/rust-lang/reference/) 的中文翻译项目。包含《The Rust Reference Developer Guide》的一个中文翻译，项目通过Deepseek

## 构建

```powershell
cd reference-zh
mdbook build
```

需要 nightly Rust 和 `cargo install --locked mdbook`。

## 规则与规范

- **[docs/authoring.md](docs/authoring.md)** — 中文翻译编写规范
- **[CONTRIBUTING.md](CONTRIBUTING.md)** — 如何参与贡献

> 如果你希望向英文原文贡献内容，请查阅[上游项目的 CONTRIBUTING 文档](https://github.com/rust-lang/reference/blob/master/CONTRIBUTING.md)。上游项目在仓库里包含了一个文件夹`dev-guide`项目，是《The Rust Reference Developer Guide》的源代码, 用来指导《Rust Reference》开发，本仓库尝试翻译了一个中文版本，方便读者熟悉大概内容，具体请以英文内容作准。

## 术语参考

翻译术语以 [Rust 程序设计语言（中文版）](https://github.com/KaiserY/trpl-zh-cn) 为主要参考。
