# 构建参考手册

## 检出源代码

要构建参考手册，首先克隆项目：

```sh
git clone https://github.com/rust-lang/reference.git
cd reference
```

## 安装 mdbook

参考手册使用 [mdbook] 构建。

首先，确保你安装了最新版本的 nightly Rust 编译器，因为运行测试需要它：

```sh
rustup toolchain install nightly
```

现在，确保你安装了 `mdbook`，因为构建参考手册需要它：

```sh
cargo install --locked mdbook
```

[mdbook]: https://rust-lang.github.io/mdBook/

## 运行 mdbook

`mdbook` 提供了若干命令和选项来帮助你进行本书的工作：

- `mdbook build --open`：构建本书并在 web 浏览器中打开。
- `mdbook serve --open`：在 localhost 上启动一个 web 服务器。每当任何文件发生更改时，它会自动重新构建本书并刷新你的 web 浏览器。

本书内容由 `SUMMARY.md` 文件驱动，每个文件都必须在那里链接。更多信息请参见 <https://rust-lang.github.io/mdBook/>。

### `SPEC_RELATIVE`

`SPEC_RELATIVE=0` 环境变量会使标准库的链接指向 <https://doc.rust-lang.org/> 而不是相对路径。这在你本地浏览时很有用，因为通常你没有标准库的副本。

```sh
SPEC_RELATIVE=0 mdbook serve --open
```

发布的网站位于 <https://doc.rust-lang.org/reference/>（或使用 `rustup doc` 的本地文档）不设置此变量，这意味着它使用相对链接。这支持离线浏览，并链接到正确的版本（例如，<https://doc.rust-lang.org/1.81.0/reference/> 中的链接将保持在 1.81.0 目录内）。

### `SPEC_DENY_WARNINGS`

`SPEC_DENY_WARNINGS=1` 环境变量将 `mdbook-spec` 生成的所有警告变为错误。这在 CI 中使用，以确保本书内容没有任何问题。

```sh
SPEC_DENY_WARNINGS=1 mdbook serve --open
```

### `SPEC_RUST_ROOT`

`SPEC_RUST_ROOT` 环境变量可用于指向 <https://github.com/rust-lang/rust> 检出的目录。测试链接功能使用它来查找链接到参考手册规则的测试。如果未设置此变量，测试将不会被链接。

```sh
SPEC_RUST_ROOT=/path/to/rust mdbook serve --open
```
