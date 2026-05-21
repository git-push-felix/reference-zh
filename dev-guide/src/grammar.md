# Rust 语法

参考手册的语法使用改良的类 BNF 语法（融合了正则表达式和其他任意元素）编写在 Markdown 代码块中。[`mdbook-spec`] 扩展解析这些规则并将其转换为可渲染的格式，包括铁路图。

代码块应该有一个语言字符串，包含单词 `grammar`、逗号和语法类别，像这样：

~~~
```grammar,items
ProductionName -> SomeExpression
```
~~~

类别用于在附录的语法摘要页面上对类似产生式进行分组。

## 语法语法

语法本身的语法类似于 **[记法][Notation]** 中描述的那样，不过渲染上有一些差异。

用 `@root` 标记的"根"产生式是不会在任何其他产生式中使用的产生式。

语法记法的语法（用其自身的记法描述）如下：

```
Grammar -> Production+

BACKTICK -> U+0060

LF -> U+000A

Production ->
    ( Comment LF )*
    `@root`? Name ` ->` Expression

Name -> <Alphanumeric or `_`>+

Expression -> Sequence (` `* `|` ` `* Sequence)*

Sequence ->
        (` `* AdornedExpr)* ` `* Cut
      | (` `* AdornedExpr)+

AdornedExpr -> Prefix? Expr1 Quantifier? Suffix? Footnote?

Prefix -> NegativeLookahead

NegativeLookahead -> `!`

Suffix -> ` _` <not underscore, unless in backtick>* `_`

Footnote -> `[^` ~[`]` LF]+ `]`

Quantifier ->
      Optional
    | Repeat
    | RepeatPlus
    | RepeatRange
    | RepeatRangeInclusive
    | RepeatRangeNamed

Optional -> `?`

Repeat -> `*`

RepeatPlus -> `+`

RepeatRange -> `{` ( Name `:` )? Range? `..` Range? `}`

RepeatRangeInclusive -> `{` ( Name `:` )? Range? `..=` Range `}`

RepeatRangeNamed -> `{` Name `}`

Range -> [0-9]+

Expr1 ->
      Unicode
    | NonTerminal
    | Break
    | Comment
    | Terminal
    | Charset
    | Prose
    | Group
    | NegativeExpression

Unicode -> `U+` [`A`-`Z` `0`-`9`]4..=6

NonTerminal -> Name

Break -> LF ` `+

Comment -> `//` ~[LF]+

Terminal -> BACKTICK ~[LF]+ BACKTICK

Charset -> `[` (` `* Characters)+ ` `* `]`

Characters ->
      CharacterRange
    | CharacterTerminal
    | CharacterName

CharacterRange -> Character `-` Character

Character ->
        BACKTICK <any char> BACKTICK
      | Unicode

CharacterTerminal -> Terminal

CharacterName -> Name

Prose -> `<` ~[`>` LF]+ `>`

Group -> `(` ` `* Expression ` `* `)`

NegativeExpression -> `~` ( Charset | Terminal | NonTerminal )

Cut -> `^` Sequence
```

通用格式是由空行分隔的一系列产生式。表达式如下：

| 表达式 | 示例 | 描述 |
|------------|---------|-------------|
| Unicode | U+0060 | 单个 Unicode 字符。 |
| NonTerminal | FunctionParameters | 按名称引用另一个产生式。 |
| Break | | 由渲染器内部使用，用于检测换行和缩进。 |
| Comment | // 单行注释。 | 延伸到行尾的注释。 |
| Terminal | \`example\` | 用反引号包围的精确字符序列。 |
| Charset | \[ \`A\`-\`Z\` \`0\`-\`9\` \`_\` \] | 从一组字符中选择，用空格分隔。有三种不同的形式。 |
| CharacterRange | \[ \`A\`-\`Z\` \] | 一个字符范围。字符可以是 Unicode 表达式或用反引号包围的字面字符。 |
| CharacterTerminal | \[ \`x\` \] | 单个字符，用反引号包围。 |
| CharacterName | \[ LF \] | 一个非终结符，引用另一个产生式。 |
| Prose | \<除 CR 外的任何 ASCII 字符\> | 用尖括号包围的关于应匹配内容的英文描述。 |
| Group | (\`,\` Parameter)+ | 将表达式分组以便确定优先级，例如将重复运算符应用于一系列其他表达式。 |
| NegativeExpression | ~\[\` \` LF\] | 匹配除给定 Charset、Terminal 或 Nonterminal 以外的任何内容。 |
| Cut | Expr1 ^ Expr2 \| Expr3 | 硬剪切运算符。一旦序列中 `^` 之前的表达式匹配，序列的其余部分必须匹配，否则解析将无条件失败 --- 任何封闭表达式都不能在剪切点之后回溯。 |
| Sequence | \`fn\` Name Parameters | 必须按顺序匹配的一系列表达式。 |
| Alternation | Expr1 \| Expr2 | 仅匹配给定表达式中的一个，用竖线字符分隔。 |
| Suffix | \_except \[LazyBooleanExpression\]\_  | 为前一个表达式添加后缀，提供附加的英文描述，以脚标渲染。这里可以包含有限的 Markdown，但尽量避免使用除链接等基础内容以外的任何东西。 |
| Footnote | \[^extern-safe\] | 添加一个脚注，可以提供对用户可能有帮助的额外信息。脚注本身应在代码块外部定义，如普通 Markdown 脚注。 |
| Optional | Expr? | 前面的表达式是可选的。 |
| NegativeLookahead | !Expr | 如果 Expr 不跟在后面且不消耗任何输入，则匹配。 |
| Repeat | Expr* | 前面的表达式重复 0 次或更多次。 |
| RepeatPlus | Expr+ | 前面的表达式重复 1 次或更多次。 |
| RepeatRange | Expr{2..4} | 前面的表达式在指定的次数范围内重复。任意边界都可以省略，就像 Rust 范围一样。 |
| RepeatRangeInclusive | Expr{2..=4} | 前面的表达式在指定的闭区间次数范围内重复。下界可以省略。 |
| RepeatRange (named) | Expr{name:2..4} | 当名称在范围之前时，重复次数会绑定到该名称，以便后续的 RepeatRangeNamed 表达式可以引用它。同样适用于 RepeatRangeInclusive。 |
| RepeatRangeNamed | Expr{name} | 前面的表达式重复的次数由先前命名的 RepeatRange 或 RepeatRangeInclusive 确定。 |

## 自动链接 {#automatic-linking}

[`mdbook-spec`] 插件会自动在每一页上为所有产生式名称添加 Markdown 链接定义。要直接链接到一个产生式名称，只需将其用方括号包围，如 `[ArrayExpression]`。

在某些情况下，可能与规则名称的自动链接产生名称冲突。在这种情况下，可以使用 `grammar-` 前缀来消除歧义，如 `[Type][grammar-Type]`。当明确性有助于提高清晰度时，也可以使用该前缀。

产生式名称也可以在链接引用定义中使用，以提供自定义链接文本，可以带也可以不带 `grammar-` 前缀。

```markdown
We accept any [type].

[type]: grammar-Type
```

```markdown
We accept any [type].

[type]: Type
```

[`mdbook-spec`]: tooling/mdbook-spec.md
[Notation]: https://doc.rust-lang.org/nightly/reference/notation.html
