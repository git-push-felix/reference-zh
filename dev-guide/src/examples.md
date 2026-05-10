# 示例

## 示例代码块

代码示例应使用带三个反引号的代码块。应始终指定语言（如 `rust`）。

```rust
println!("Hello!");
```

关于支持的语言列表，请参见 [mdBook 支持的语言][mdBook supported languages]。

## rustdoc 属性

Rust 示例通过 [rustdoc 进行测试][tested via rustdoc]，应包含适当的注解：

- `edition2015`、`edition2018` 等 --- 如果示例是特定版本的（默认版本见 `book.toml`）。
- `no_run` --- 示例应该编译成功，但不应该执行。
- `should_panic` --- 示例应该编译并运行，但会产生 panic。
- `compile_fail` --- 示例预期编译失败。
- `ignore` --- 示例不应该被构建或测试。应尽可能避免。通常只有在测试框架不支持时（如外部 crate、模块或 proc-macro）或包含不是合法 Rust 的伪代码时才需要。应在示例前放置 HTML 注释，如 `<!-- ignore: requires extern crate -->`，来说明为什么被忽略。
- `Exxxx` --- 如果示例预期编译失败并带有特定的错误代码，请包含该代码，以便 `rustdoc` 检查是否使用了预期的错误代码。

更多细节请参见 [rustdoc 文档][rustdoc documentation]。

## 合并示例

在演示成功情况时，可以在单个代码块中包含多个情况。但对于失败情况，每个示例必须出现在单独的代码块中，这样测试才能确保每种情况确实以适当的错误代码失败。

## 测试示例

Rust 代码块在 CI 中被测试。你可以通过运行 [`mdbook test`] 来验证示例是否通过。

[`mdbook test`]: tests.md#inline-tests
[mdBook supported languages]: https://rust-lang.github.io/mdBook/format/theme/syntax-highlighting.html#supported-languages
[rustdoc documentation]: https://doc.rust-lang.org/rustdoc/documentation-tests.html
[tested via rustdoc]: tests.md#inline-tests
