# Rust 参考手册

这是 [《The Rust Reference》](https://github.com/rust-lang/reference/) 的中文翻译项目，同时包含[《The Rust Reference Developer Guide》](https://rust-lang.github.io/reference/dev-guide/)的一个中文翻译。项目通过[DeepSeek](https://www.deepseek.com/)进行初翻，人工通篇阅读进行校对的方式产生，翻译力求严谨、代码力求规范。因书里专业内容强，学者个人能力有限，翻译内容难免会有疏漏和欠妥之处，恳请各位读者批评指正，同时也十分欢迎您参与贡献翻译内容以完善本项目。

## 构建

```powershell
cd reference-zh
mdbook build
```

构建需要 nightly Rust 和 `cargo install --locked mdbook`。

## 规则与规范

- **[docs/authoring.md](docs/authoring.md)** — 中文翻译编写规范也是给DeepSeek的提示词
- **[CONTRIBUTING.md](CONTRIBUTING.md)** — 如何参与贡献本项目

> 如果你希望向英文原文贡献内容，请查阅[上游项目的 CONTRIBUTING 文档](https://github.com/rust-lang/reference/blob/master/CONTRIBUTING.md)。上游项目在仓库里包含了一个文件夹`dev-guide`项目，是《The Rust Reference Developer Guide》的源代码, 用来指导《Rust Reference》开发，本仓库尝试翻译了一个中文版本，方便读者熟悉大概内容，具体请以英文内容作准。

## 术语参考

[《The Rust Programming Language》](https://doc.rust-lang.org/book/)，亦称《The Book》，早些年网络上戏称其为 “Rust语言圣经”，是学习Rust的重要入门参考书之一，[《Rust 程序设计语言 简体中文版》](https://github.com/KaiserY/trpl-zh-cn) 是社区里对《The Book》高质量的中文翻译，本项目积极参考其翻译内容用以完善《The Rust Reference》的翻译。
