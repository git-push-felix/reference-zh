r[comments]
# 注释

r[comments.syntax]
```grammar,lexer
@root COMMENT ->
      LINE_COMMENT
    | INNER_LINE_DOC
    | OUTER_LINE_DOC
    | INNER_BLOCK_DOC
    | OUTER_BLOCK_DOC
    | BLOCK_COMMENT

LINE_COMMENT ->
      `//` (~[`/` `!` LF] | `//`) ~LF*
    | `//` EOF
    | `//` _immediately followed by LF_

BLOCK_COMMENT ->
    `/*` ^
      ( BLOCK_COMMENT_OR_DOC | (!`*/` CHAR) )*
    `*/`

INNER_LINE_DOC ->
    `//!` ^ LINE_DOC_COMMENT_CONTENT (LF | EOF)

LINE_DOC_COMMENT_CONTENT -> (!CR ~LF)*

INNER_BLOCK_DOC ->
    `/*!` ^ ( BLOCK_COMMENT_OR_DOC | BLOCK_CHAR )* `*/`

OUTER_LINE_DOC ->
    `///` ^ LINE_DOC_COMMENT_CONTENT (LF | EOF)

OUTER_BLOCK_DOC ->
    `/**` ![`*` `/`]
      ^
      ( ~`*` | BLOCK_COMMENT_OR_DOC )
      ( BLOCK_COMMENT_OR_DOC | BLOCK_CHAR )*
    `*/`

BLOCK_CHAR -> (!(`*/` | CR) CHAR)

BLOCK_COMMENT_OR_DOC ->
      INNER_BLOCK_DOC
    | OUTER_BLOCK_DOC
    | BLOCK_COMMENT
```

r[comments.normal]
## 非文档注释

注释遵循通用的 C++ 风格，支持行（`//`）和块（`/* ... */`）注释形式。支持嵌套块注释。

r[comments.normal.tokenization]
非文档注释被解释为一种空白字符。

r[comments.doc]
## 文档注释 {#doc-comments}

r[comments.doc.syntax]
以恰好*三*条斜杠（`///`）开头的行文档注释，以及块文档注释（`/** ... */`），都是外部文档注释，会被解释为 [`doc` 属性][`doc` attributes] 的特殊语法。

r[comments.doc.attributes]
也就是说，它们等价于在注释主体周围书写 `#[doc="..."]`，即 `/// Foo` 转变为 `#[doc=" Foo"]`，`/** Bar */` 转变为 `#[doc=" Bar "]`。它们因此必须出现在接受外部属性的项之前。

r[comments.doc.inner-syntax]
以 `//!` 开头的行注释和块注释 `/*! ... */` 是应用于注释父项的文档注释，而不是应用于其后跟随的项。

r[comments.doc.inner-attributes]
也就是说，它们等价于在注释主体周围书写 `#![doc="..."]`。`//!` 注释通常用于为占据一个源文件的模块编写文档。

r[comments.doc.bare-crs]
字符 `U+000D`（CR）不允许出现在文档注释中。

> [!NOTE]
> 按照惯例，文档注释会包含 Markdown，这是 `rustdoc` 所期望的。但是，注释语法本身并不遵循任何 Markdown 规则。`` /** `glob = "*/*.rs";` */`` 会在第一个 `*/` 处终止注释，剩余代码将导致语法错误。与行文档注释相比，这稍微限制了块文档注释的内容。

> [!NOTE]
> `U+000D`（CR）后紧跟 `U+000A`（LF）的序列在之前已被转换为单个 `U+000A`（LF）。

## 示例

```rust
//! 应用于此 crate 的隐式匿名模块的文档注释

pub mod outer_module {

    //!  - 内部行文档注释
    //!! - 仍然是内部行文档注释（但开头有一个感叹号）

    /*!  - 内部块文档注释 */
    /*!! - 仍然是内部块文档注释（但开头有一个感叹号） */

    //   - 仅是一个注释
    ///  - 外部行文档注释（恰好 3 条斜杠）
    //// - 仅是一个注释

    /*   - 仅是一个注释 */
    /**  - 外部块文档注释（恰好 2 个星号） */
    /*** - 仅是一个注释 */

    pub mod inner_module {}

    pub mod nested_comments {
        /* Rust 中 /* 我们可以 /* 嵌套注释 */ */ */

        // 所有三种块注释都可以包含或被嵌套在任何其他类型的注释中：

        /*   /* */  /** */  /*! */  */
        /*!  /* */  /** */  /*! */  */
        /**  /* */  /** */  /*! */  */
        pub mod dummy_item {}
    }

    pub mod degenerate_cases {
        // 空内部行文档注释
        //!

        // 空内部块文档注释
        /*!*/

        // 空行注释
        //

        // 空外部行文档注释
        ///

        // 空块注释
        /**/

        pub mod dummy_item {}

        // 空 2 星号块不是文档块，而是块注释
        /***/

    }

    /* 下一个是不允许的，因为外层文档注释需要一个接收该文档的项 */

    /// 我的项在哪里？
#   mod boo {}
}
```

[`doc` attributes]: ../rustdoc/the-doc-attribute.html
