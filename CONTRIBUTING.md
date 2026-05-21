# 参与项目建设

**诚挚感谢您前来为《The Rust Reference》的中文翻译《Rust 参考手册》和《The Rust Reference Developer Guide》的中文翻译《Rust 参考手册开发者指南》做出贡献！**

如果你希望向英文原文贡献内容，请查阅[上游项目的 CONTRIBUTING 文档](https://github.com/rust-lang/reference/blob/master/CONTRIBUTING.md)。

本篇文档主要涵盖三个方面的内容：翻译内容的编写规范、向仓库贡献内容、构建与测试项目。

## 翻译内容的编写规范

在项目的 [docs/authoring.md](docs/authoring.md) 文件里记载了从英文翻译成中文所需要注意和调整的地方，以及一些翻译原则。同时，这篇文档也是给 DeepSeek 进行翻译的提示词(prompts)。

## 向仓库贡献内容

十分欢迎您向仓库里的项目贡献内容：可以是在[讨论区](https://github.com/git-push-felix/reference-zh/discussions)提出您的意见和建议；也可以是发现错误或问题向仓库提交 [Issues](https://github.com/git-push-felix/reference-zh/issues)；还可以是向仓库提交您翻译内容的 [Pulls](https://github.com/git-push-felix/reference-zh/pulls)。

为了方便大家的合作，针对常见的事项，在这里和您做一个约定：

- 当英文上游有更新时，您向仓库提交翻译的新内容时，您需要：

    1. 检出英文上游变更
    2. 按文件状态处理（新增 A / 修改 M / 删除 D / 重命名 R）
    3. 翻译差异部分，验证编译，完成所有测试
    4. 提交 PR

- 如果你发现翻译错误或不准确，您向仓库提交修正翻译错误后的内容时，您需要：

    1. 直接提交 PR 修正
    2. 简要说明修改了什么以及为什么

- 如果你发现翻译错误或不准确，您向仓库提交 Issue 希望订正内容时，您需要：
    1. 指明哪个文件
    2. 最好能定位到哪行内容
    3. 问题是什么（翻译错误 / 表述不清 / 格式问题等等）

## 构建与测试项目

### 前置准备

构建项目需要 Rust (nightly channel) 和 `mdbook` CLI.

- Rust (nightly channel) 
    
    1. 根据官网引导[安装 Rust 开发环境](https://rust-lang.org/zh-CN/tools/install/)
    
    2. 安装 nightly 版本工具链：
        ```powershell
        rustup toolchain install nightly
        ```
        **请注意**：当您克隆了本仓库进行首次构建活动时，`rustup`会检测到 [rust-toolchain.toml](rust-toolchain.toml) 文件里的设置，自动下载 nightly 版本的工具链，所以您完全可以不执行本步骤里的手动添加。

- `mdbook` CLI

    安装主要有源码构建和下载二进制文件两种方式：

    - 从源码构建 `mdbook` CLI 您只需要在命令行中执行：
        
        ```powershell
        # 参数 `--locked` 要求 Cargo 强制使用 Cargo.lock 中记录的精确依赖版本, 
        # 而不是根据 Cargo.toml 中的版本要求(e.g. ^1.0)重新解析出最新的兼容版本
        cargo install --locked mdbook
        ```

    - 直接下载 `mdbook` CLI 二进制文件是一个更加方便的方式，您可以直接去到[`mdbook` 仓库](https://github.com/rust-lang/mdBook)下载最新 release 版本的 CLI 文件。
    
    > 此外社区里还有 `cargo-binstall` 等工具，它们提供 `cargo binstall *` 命令，可以直接安装 crate 的二进制文件，有兴趣可以查看[它的仓库](https://github.com/cargo-bins/cargo-binstall)。

### 构建

克隆仓库以进行构建：

```powershell
git clone "https://github.com/git-push-felix/reference-zh.git"
cd "reference-zh"
# 若是直接下载的二进制文件且不打算添加到环境变量里
# 则需要将 `mdbook` 命令替换成文件路径
mdbook build
```

### 测试 

项目测试请您参考《Rust 参考手册开发者指南》里的[运行测试](dev-guide/src/tests.md)板块。

## 许可证

本项目与上游项目一致，采用 [Apache 2.0](../LICENSE-APACHE) 和 [MIT](../LICENSE-MIT) 双许可。
