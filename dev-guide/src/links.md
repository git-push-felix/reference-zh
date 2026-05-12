# 链接

本章说明参考手册中链接应如何处理。其中一些功能由 [`mdbook-spec`](tooling/mdbook-spec.md) 提供。

关于测试链接，也可参见[链接检查器测试](tests.md#linkcheck)。

## 规则链接{#rule-links}

[规则](rules/index.md)可以通过其 ID 使用 Markdown 进行链接，目标设置为规则 ID。自动链接引用使得任何规则都可以从书中的任何页面被引用。

```markdown
Direct label link: [names.preludes.lang]

Destination label link (custom link text): [language prelude][names.preludes.lang]

Definition link: [namespace kinds]

[namespace kinds]: names.namespaces.kinds
```

## 标准库链接{#standard-library-links}

你应该在不指定 URL 的情况下链接到标准库，方式类似于 [rustdoc 文档内链接][intra]。一些示例：

我们可以链接到 `Option` 的页面：

```markdown
[`std::option::Option`]
```

在这些链接中，泛型会被忽略，可以被包含：

```markdown
[`std::option::Option<T>`]
```

如果我们不想在文本中显示完整路径，可以写成：

```markdown
[`Option`](std::option::Option)
```

宏可以用 `!` 结尾。这有助于消除歧义。例如，这指的是宏而不是模块：

```markdown
[`alloc::vec!`]
```

也支持显式命名空间消歧义：

```markdown
[`std::vec`](mod@std::vec)
```

注意一些限制，例如：

- 由于 <https://github.com/rust-lang/rust/issues/96506>，指向 `std_arch` 的重导出的链接不起作用。
- 不支持指向关键字的链接。
- 指向 trait 不在 prelude 中的 trait impl 的链接不起作用。trait 必须在作用域内，目前没有办法添加它们。
- 如果有多个泛型实现，它会随机链接到一个（参见 <https://github.com/rust-lang/rust/issues/76895>）。

遇到 rustdoc 限制时，可以考虑使用相对链接手动链接到正确的页面。例如，`../std/arch/macro.is_x86_feature_detected.html`。

在本地渲染参考手册时，默认使用相对链接以符合书籍的发布方式。这可能不是你想要的效果，所以你通常需要设置 [`SPEC_RELATIVE=0` 环境变量][rel]，使链接指向在线站点。

[intra]: https://doc.rust-lang.org/rustdoc/write-documentation/linking-to-items-by-name.html
[rel]: tooling/building.md#spec_relative

## 语法链接{#grammar-links}

所有语法产生式名称的链接定义都会自动生成。更多信息请参见[语法自动链接](grammar.md#automatic-linking)。

```markdown
This attribute uses the [MetaWord] syntax.

Explicit grammar links can have the `grammar-` prefix like [Type][grammar-Type].

Grammar links can also appear in link reference definitions, e.g. [type].

[type]: grammar-Type
```

## 外部书籍链接{#outside-book-links}

指向与参考手册一起发布的其他书籍的链接应该是相对链接，指向对应的书籍。这使链接能够指向正确的版本，与离线文档一起工作，并被链接检查器检查。例如：

```markdown
See [`-C panic`].

[`-C panic`]: ../rustc/codegen-options/index.html#panic
```

## 内部链接

在可能的情况下，内部链接应使用[规则链接](#rule-links)或[语法链接](#grammar-links)。否则，链接应该是相对于文件路径的，并使用 `.md` 扩展名。

```markdown
- Rule link: [language prelude][names.preludes.lang]
- Grammar link: [MetaWord]
- Internal link: [Modules](items/modules.md#attributes-on-modules)
```
