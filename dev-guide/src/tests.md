# 运行测试

你可以运行几类不同的测试（这些测试在 CI 中强制执行）：

- [`cargo xtask test-all`](#all-tests) --- 运行所有测试。
- [`mdbook test`](#inline-tests) --- 测试内联 Rust 代码块。
- [`cargo xtask linkcheck`](#linkcheck) --- 验证 Markdown 链接没有失效。
- [`cargo xtask style-check`](#style-checks) --- 验证各种风格检查。
- [代码格式化](#code-formatting) --- 检查所有 Rust 工具代码是否已格式化。
- [mdbook-spec 测试](#mdbook-spec-tests) --- `mdbook-spec` 的内部测试。

## 所有测试{#all-tests}

```sh
cargo xtask test-all
```

此命令运行下面列出的所有测试。

我们建议在提交 PR 之前最后一步运行此命令。它会运行 CI 通过所需的大部分测试。关于其具体功能，请参见 [`tools/xtask/src/main.rs`](https://github.com/rust-lang/reference/blob/HEAD/tools/xtask/src/main.rs)。

## 内联测试{#inline-tests}

```sh
mdbook test
```

此命令运行 Markdown 中内联的所有测试。内部使用 [`rustdoc`](https://doc.rust-lang.org/rustdoc/) 来运行测试，并支持所有相同的功能。任何带有 `rust` 语言的代码块都将被编译，除非被忽略。更多信息请参见[示例][Examples]。

## 链接检查{#linkcheck}

```sh
cargo xtask linkcheck
```

此命令验证链接没有失效。它下载并使用托管在 `rust-lang/rust` 仓库中的 [`linkchecker`](https://github.com/rust-lang/rust/tree/main/src/tools/linkchecker) 脚本。

这需要通过 `rustup` 安装的最新 nightly 以及 `rust-docs` 组件。

编译脚本后，它使用 `mdbook` 构建参考手册，然后扫描所有本地链接以验证它们是否有效，特别是跨不同书籍之间的链接。这不会检查任何网络链接。

## 风格检查{#style-checks}

```sh
cargo xtask style-check
```

这使用 [`style-check`](https://github.com/rust-lang/reference/tree/HEAD/tools/style-check) 工具来强制执行各种格式化规则。

## 代码格式化{#code-formatting}

CI 使用 `cargo fmt --check` 来验证所有工具（如 `mdbook-spec`）的 Rust 源代码是否已正确格式化。所有代码必须使用 `rustfmt` 格式化。

## mdbook-spec 测试{#mdbook-spec-tests}

```sh
cargo test --manifest-path mdbook-spec/Cargo.toml
```

CI 对 `mdbook-spec` 运行 `cargo test` 来执行该工具本身的测试。

[Examples]: examples.md
