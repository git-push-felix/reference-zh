r[type.char]
# 字符类型

r[type.char.intro]
`char` 类型表示单个 [Unicode 标量值]（即不是代理项对的码位）。

> [!EXAMPLE]
> ```rust
> let c: char = 'a';
> let emoji: char = '😀';
> let unicode: char = '\u{1F600}';
> ```

> [!NOTE]
> 有关 `char` 类型的实现信息，请参阅[标准库文档][`char`]。

r[type.char.value]
`char` 类型的值表示为一个 32 位无符号字，取值范围在 0x0000 到 0xD7FF 或 0xE000 到 0x10FFFF 之间。创建超出此范围的 `char` 将立即构成[未定义行为]。

r[type.char.layout]
`char` 在所有平台上保证与 `u32` 具有相同的大小和对齐。

r[type.char.validity]
`char` 的每个字节都保证被初始化。换句话说，`transmute::<char, [u8; size_of::<char>()]>(...)` 始终是可靠的——但由于某些位模式是无效的 `char`，反过来并不总是可靠的。

[Unicode 标量值]: http://www.unicode.org/glossary/#unicode_scalar_value
[未定义行为]: ../behavior-considered-undefined.md
