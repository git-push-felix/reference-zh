r[notation]
# 记法

r[notation.grammar]
## 语法

r[notation.grammar.syntax]

以下记法约定用于*词法分析器*和*语法*片段：

| 记法          | 示例                      | 含义                                   |
|-------------------|-------------------------------|-------------------------------------------|
| CAPITAL           | KW_IF, INTEGER_LITERAL        | 词法分析器产生的 token             |
| _ItalicCamelCase_ | _LetStatement_, _Item_        | 语法产生式                  |
| `string`          | `x`, `while`, `*`             | 精确的字符                    |
| x<sup>?</sup>     | `pub`<sup>?</sup>             | 可选项                          |
| x<sup>\*</sup>    | _OuterAttribute_<sup>\*</sup> | 0 个或多个 x                            |
| x<sup>+</sup>     |  _MacroMatch_<sup>+</sup>     | 1 个或多个 x                            |
| x<sup>a..b</sup>  | HEX_DIGIT<sup>1..6</sup>      | x 重复 a 到 b 次，不含 b   |
| x<sup>a..=b</sup> | HEX_DIGIT<sup>1..=5</sup>     | x 重复 a 到 b 次，含 b   |
| x<sup>n:a..=b</sup> | `#`<sup>n:1..=255</sup>      | x 重复 a 到 b 次（含 b），计数绑定到名称 n |
| x<sup>n</sup>     | `#`<sup>n</sup>               | x 按前一个带标签的重复所绑定的次数 n 重复 |
| Rule1 Rule2       | `fn` _Name_ _Parameters_      | 按顺序排列的规则序列                |
| \|                | `u8` \| `u16`, Block \| Item  | 二者之一                     |
| !                 | !COMMENT                      | 若其后没有该表达式则匹配，且不消耗输入 |
| \[ ]               | \[`b` `B`]                     | 所列字符中的任意一个              |
| \[ - ]             | \[`a`-`z`]                     | 该范围内的任意字符        |
| ~\[ ]              | ~\[`b` `B`]                    | 除所列字符外的任意字符       |
| ~`string`         | ~`\n`, ~`*/`                  | 除该序列外的任意字符      |
| ( )               | (`,` _Parameter_)<sup>?</sup> | 将项分组                              |
| ^                 | `b'` ^ ASCII_FOR_CHAR         | 序列剩余部分必须匹配，否则解析将无条件失败（[硬切割运算符][hard cut operator]） |
| U+xxxx..xxxxxx    | U+0060                        | 单个 Unicode 字符                |
| \<text\>          | \<any ASCII char except CR\>  | 对应匹配内容的英文描述 |
| Rule <sub>suffix</sub> | IDENTIFIER_OR_KEYWORD <sub>_except `crate`_</sub> | 对前一条规则的修改 |
| // Comment. | // 单行注释。 | 延伸到行尾的注释。 |

序列的优先级高于 `|` 交替。

r[notation.grammar.cut]
### 硬切割运算符 {#the-hard-cut-operator}

该语法使用有序交替：解析器从左到右尝试各分支，采纳第一个匹配的分支。如果某个分支在序列中途失败，解析器通常回溯并尝试下一个分支。切割运算符（`^`）可以阻止这种情况。一旦序列中 `^` 左侧的所有表达式都已匹配，序列剩余部分必须匹配，否则解析将无条件失败。

Mizushima 等人将[切割运算符][cut operator paper]引入了解析表达式文法。在 PEG 文献中，*软切割*仅阻止在最内层有序选择的范围内回溯------外层选择仍可恢复。*硬切割*则阻止切割点之后的所有回溯；失败是确定性的。本语法中使用的 `^` 是硬切割。

硬切割运算符是必要的，因为 Rust 中的某些 token 以本身也是合法 token 的前缀开头。例如，`c"` 开始一个 C 字符串字面量，但单独的 `c` 是一个合法的标识符。如果没有切割，当 `c"\0"` 无法被词法分析为 C 字符串字面量时（因为 C 字符串不允许空字节），解析器可能回溯并将其词法分析为两个 token：标识符 `c` 和字符串字面量 `"\0"`。[`c"` 后的切割][cut after `c"`]可以防止这种情况------一旦识别出起始定界符，解析器就不能回退。同样的原理适用于[字节字面量][byte literals]、[字节字符串字面量][byte string literals]、[原始字符串字面量][raw string literals]以及其他以前缀开头的字面量，这些前缀本身也是合法的 token。

r[notation.grammar.string-tables]
### 字符串表产生式 {#string-table-productions}

语法中的某些规则------特别是[一元运算符][unary operators]、[二元运算符][binary operators]和[关键字][keywords]------以简化形式给出：作为可打印字符串的列表。这些情况构成了关于 [token][tokens] 规则的一个子集，并假定是向解析器提供输入的词法分析阶段的结果，该阶段由 <abbr title="Deterministic Finite Automaton">DFA</abbr> 驱动，操作于所有此类字符串表条目的析取之上。

当语法中出现 `monospace` 字体的字符串时，它是对此类字符串表产生式中单个成员的隐式引用。详见 [token 词法单元][tokens]。

r[notation.grammar.visualizations]
### 语法可视化

每个语法块下方有一个按钮，可以切换显示[语法图][syntax diagram]。方形元素是非终结规则，圆角矩形是终结符。

[binary operators]: expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[byte literals]: tokens.md#r-lex.token.byte.syntax
[byte string literals]: tokens.md#r-lex.token.str-byte.syntax
[cut after `c"`]: tokens.md#r-lex.token.str-c.syntax
[cut operator paper]: https://kmizu.github.io/papers/paste513-mizushima.pdf
[hard cut operator]: notation.md#the-hard-cut-operator
[keywords]: keywords.md
[raw string literals]: tokens.md#r-lex.token.literal.str-raw.syntax
[syntax diagram]: https://en.wikipedia.org/wiki/Syntax_diagram
[tokens]: tokens.md
[unary operators]: expressions/operator-expr.md#borrow-operators
