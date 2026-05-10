r[type.text]
# 文本类型

r[type.text.intro]
类型 `char` 和 `str` 保存文本数据。

r[type.text.char-value]
`char` 类型的值是一个 [Unicode 标量值]（即不是代理项对的码位），表示为一个 32 位无符号字，取值范围在 0x0000 到 0xD7FF 或 0xE000 到 0x10FFFF 之间。

r[type.text.char-precondition]
创建超出此范围的 `char` 将立即构成[未定义行为]。`[char]` 实际上是长度为 1 的 UCS-4 / UTF-32 字符串。

r[type.text.str-value]
`str` 类型的值以与 `[u8]`（8 位无符号字节切片）相同的方式表示。不过，Rust 标准库对 `str` 有额外的假定：操作 `str` 的方法假定并确保其中的数据是有效的 UTF-8。使用非 UTF-8 缓冲区调用 `str` 方法可能现在或将来引发[未定义行为]。

r[type.text.str-unsized]
由于 `str` 是[动态大小类型]，它只能通过指针类型（如 `&str`）进行实例化。`&str` 的布局与 `&[u8]` 的布局相同。

r[type.text.layout]
## 布局与位有效性

r[type.layout.char-layout]
`char` 在所有平台上保证与 `u32` 具有相同的大小和对齐。

r[type.layout.char-validity]
`char` 的每个字节都保证被初始化（换句话说，`transmute::<char, [u8; size_of::<char>()]>(...)` 始终是可靠的——但由于某些位模式是无效的 `char`，反过来并不总是可靠的）。

[Unicode 标量值]: http://www.unicode.org/glossary/#unicode_scalar_value
[未定义行为]: ../behavior-considered-undefined.md
[动态大小类型]: ../dynamically-sized-types.md
