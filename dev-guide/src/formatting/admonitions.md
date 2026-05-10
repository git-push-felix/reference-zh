# 提示块

[`mdbook-spec`](../tooling/mdbook-spec.md) 提供了一些提示块，使用类似 GitHub 风格 Markdown 的样式。样式名称放置在引用块的开头，例如：

```markdown
> [!WARNING]
> This is a warning.

> [!NOTE]
> This is a note.

> [!EDITION-2024]
> This is an edition-specific difference.

> [!EXAMPLE]
> This is an example.
```

颜色和样式定义在 [`theme/reference.css`](https://github.com/rust-lang/reference/blob/HEAD/theme/reference.css) 中，转换和图标位于 [`tools/mdbook-spec/src/admonitions.rs`](https://github.com/rust-lang/reference/blob/HEAD/tools/mdbook-spec/src/admonitions.rs) 中。

关于如何使用这些提示块，请参见参考手册引言中的 **[约定][Conventions]**。

[Conventions]: https://doc.rust-lang.org/nightly/reference/#conventions
