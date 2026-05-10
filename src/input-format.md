r[input]
# 输入格式

r[input.syntax]
```grammar,lexer
CHAR -> [U+0000-U+D7FF U+E000-U+10FFFF] // a Unicode scalar value

ASCII -> [U+0000-U+007F]

NUL -> U+0000

EOF -> !CHAR  // End of file or input
```

r[input.intro]
本章描述源文件如何被解释为 token 序列。

关于程序如何组织为文件的描述，请参阅 [crate 和源文件][Crates and source files]。

r[input.encoding]
## 源文件编码

r[input.encoding.utf8]
每个源文件被解释为以 UTF-8 编码的 Unicode 字符序列。

r[input.encoding.invalid]
如果文件不是有效的 UTF-8，则视为错误。

r[input.byte-order-mark]
## 字节顺序标记移除

如果序列中的第一个字符是 `U+FEFF`（[BYTE ORDER MARK]），则将其移除。

r[input.crlf]
## CRLF 规范化

每对字符 `U+000D`（CR）后紧跟 `U+000A`（LF）将被替换为单个 `U+000A`（LF）。此过程仅执行一次，而非重复执行，因此规范化之后，输入中仍可能存在紧跟 `U+000A`（LF）的 `U+000D`（CR）（例如，原始输入包含 "CR CR LF LF"）。

其他位置的字符 `U+000D`（CR）将保留（它们被视为[空白字符][whitespace]）。

r[input.shebang]
## Shebang 移除

r[input.shebang.removal]
如果存在 [shebang]，则将其从输入序列中移除（因此被忽略）。

r[input.tokenization]
## Token 化

然后将生成的字符序列转换为 token，具体描述见本章其余部分。

> [!NOTE]
> 标准库的 [`include!`] 宏会对其读取的文件应用以下转换：
>
> - 字节顺序标记移除。
> - CRLF 规范化。
> - 在程序项上下文中（而非表达式或语句上下文）调用时的 shebang 移除。
>
> [`include_str!`] 和 [`include_bytes!`] 宏不应用这些转换。

[BYTE ORDER MARK]: https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
[Crates and source files]: crates-and-source-files.md
[shebang]: shebang.md
[whitespace]: whitespace.md
