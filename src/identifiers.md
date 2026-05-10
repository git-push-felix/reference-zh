r[ident]
# 标识符

r[ident.syntax]
```grammar,lexer
IDENTIFIER_OR_KEYWORD -> ( XID_Start | `_` ) XID_Continue*

XID_Start -> <`XID_Start` defined by Unicode>

XID_Continue -> <`XID_Continue` defined by Unicode>

RAW_IDENTIFIER -> `r#` IDENTIFIER_OR_KEYWORD

NON_KEYWORD_IDENTIFIER -> IDENTIFIER_OR_KEYWORD _except a [strict][lex.keywords.strict] or [reserved][lex.keywords.reserved] keyword_

IDENTIFIER -> NON_KEYWORD_IDENTIFIER | RAW_IDENTIFIER

RESERVED_RAW_IDENTIFIER ->
    `r#` (`_` | `crate` | `self` | `Self` | `super`) !XID_Continue
```

<!-- When updating the version, update the UAX links, too. -->
r[ident.unicode]
标识符遵循 [Unicode 标准附录 #31][UAX31] 中 Unicode 版本 17.0 的规范，并包含下文所述的补充。以下是一些标识符的示例：

* `foo`
* `_identifier`
* `r#true`
* `Москва`
* `東京`

r[ident.profile]
UAX #31 使用的配置为：

* Start := [`XID_Start`]，加上下划线字符（U+005F）
* Continue := [`XID_Continue`]
* Medial := 空

> [!NOTE]
> 以下划线开头的标识符通常用于表示有意未使用的标识符，并且会消除 `rustc` 中"未使用"的警告。

r[ident.keyword]
标识符不能是[严格][strict]或[保留][reserved]关键字，除非使用下文[原始标识符](#raw-identifiers)中描述的 `r#` 前缀。

r[ident.zero-width-chars]
零宽度非连接符（ZWNJ U+200C）和零宽度连接符（ZWJ U+200D）字符不允许出现在标识符中。

r[ident.ascii-limitations]
在以下情况下，标识符仅限于 [`XID_Start`] 和 [`XID_Continue`] 的 ASCII 子集：

* [`extern crate`][extern crate] 声明（除 [AsClause] 标识符外）
* 在[路径][path]中引用的外部 crate 名称
* 从文件系统加载且不带 [`path` 属性][path attribute]的[模块][module]名称
* 带有 [`no_mangle`][no_mangle] 属性的程序项
* [外部块][external blocks]中的程序项名称

r[ident.normalization]
## 规范化

标识符使用 [Unicode 标准附录 #15][UAX15] 中定义的规范化形式 C（NFC）进行规范化。如果两个标识符的 NFC 形式相等，则它们相等。

[过程宏][proc-macro]和[声明宏][mbe]在其输入中接收规范化后的标识符。

r[ident.raw]
## 原始标识符 {#raw-identifiers}

r[ident.raw.intro]
原始标识符类似于普通标识符，但带有 `r#` 前缀。（注意，`r#` 前缀不算作实际标识符的一部分。）

r[ident.raw.allowed]
与普通标识符不同，原始标识符可以使用任何严格或保留关键字，但 `RAW_IDENTIFIER` 中以上列出的除外。

r[ident.raw.reserved]
使用 [RESERVED_RAW_IDENTIFIER] token 是错误的。

[`extern crate`]: items/extern-crates.md
[`no_mangle`]: abi.md#the-no_mangle-attribute
[`path` attribute]: items/modules.md#the-path-attribute
[`XID_Continue`]: http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AXID_Continue%3A%5D&abb=on&g=&i=
[`XID_Start`]:  http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AXID_Start%3A%5D&abb=on&g=&i=
[external blocks]: items/external-blocks.md
[mbe]: macros-by-example.md
[module]: items/modules.md
[path]: paths.md
[proc-macro]: procedural-macros.md
[reserved]: keywords.md#reserved-keywords
[strict]: keywords.md#strict-keywords
[UAX15]: https://www.unicode.org/reports/tr15/tr15-57.html
[UAX31]: https://www.unicode.org/reports/tr31/tr31-43.html
