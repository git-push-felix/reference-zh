# Markdown

其中一些规则有自动化检查。运行 [`cargo xtask style-check`] 可以在本地运行它们。

## 格式化风格

- 使用 [ATX 风格标题][atx]（不要用 Setext），并采用[句首字母大写][sentence case]。
- 不要使用 tab；只使用空格。
- 文件必须以换行符结尾。
- 行尾不能有空格。双空格有语义含义，但可能是不可见的。如果你需要硬换行，请使用尾部反斜杠。
- 如果可能，避免连续的空行。
- 不要对长行进行换行。这有助于评审源文件的差异。
- 使用[智能标点][smart punctuation]而不是 Unicode 字符。例如，使用 `---` 表示 em dash 而不是 Unicode 字符。像 em dash 这样的字符在等宽编辑器中可能难以看到，而且某些编辑器可能没有方便的方法来输入这些字符。
- 有关格式化注释、版本差异和警告等提示框的信息，请参见[提示块][Admonitions]。

## 代码块

- 不要使用缩进的代码块；应使用带 3 个或更多反引号的围栏代码块。
- 代码块应该有明确的语言标签。

## 链接

关于链接的更多信息，请参见[链接][Links]。

- 指向其他章节的链接应该是相对路径，并使用 `.md` 扩展名。
- 指向与参考手册一起发布的其他 rust-lang 书籍的链接也应该是相对路径，这样链接检查器可以验证它们。请参见[外部书籍链接][outside book links]。
- 指向标准库的链接应使用 rustdoc 风格的链接，如[标准库链接][standard library links]中所述。
- 优先使用引用链接，在适当的地方使用快捷引用链接。将排序后的链接引用定义放在文件底部，或者如果某个小节有特别多的特定链接，就放在该小节的底部。

    ```markdown
    Example of shortcut link: [enumerations]
    Example of reference link with label: [block expression][block]

    [block]: expressions/block-expr.md
    [enumerations]: types/enum.md
    ```

[`cargo xtask style-check`]: ../tests.md#style-checks
[Admonitions]: admonitions.md
[atx]: https://spec.commonmark.org/0.31.2/#atx-headings
[Links]: ../links.md
[outside book links]: ../links.md#outside-book-links
[sentence case]: https://apastyle.apa.org/style-grammar-guidelines/capitalization/sentence-case
[smart punctuation]: https://rust-lang.github.io/mdBook/format/markdown.html#smart-punctuation
[standard library links]: ../links.md#standard-library-links
