r[expr.literal]
# 字面量表达式

r[expr.literal.syntax]
```grammar,expressions
LiteralExpression ->
      CHAR_LITERAL
    | STRING_LITERAL
    | RAW_STRING_LITERAL
    | BYTE_LITERAL
    | BYTE_STRING_LITERAL
    | RAW_BYTE_STRING_LITERAL
    | C_STRING_LITERAL
    | RAW_C_STRING_LITERAL
    | INTEGER_LITERAL
    | FLOAT_LITERAL
    | `true`
    | `false`
```

r[expr.literal.intro]
*字面量表达式*是由单个记号而非记号序列组成的表达式，它直接且立即表示它所求值为的值，而不是通过名称或其他求值规则来引用它。

r[expr.literal.const-expr]
字面量是[常量表达式][constant expression]的一种形式，因此（主要在）编译时求值。

r[expr.literal.literal-token]
前面描述的每种词法[字面量][literal tokens]形式都可以构成字面量表达式，关键字 `true` 和 `false` 也可以。

```rust
"hello";   // 字符串类型
'5';       // 字符类型
5;         // 整数类型
```

r[expr.literal.string-representation]
在下面的描述中，记号的*字符串表示*是输入中与该记号在*词法器*语法片段中的产生式相匹配的字符序列。

> [!NOTE]
> 此字符串表示永远不会包含紧跟在 `U+000A`（LF）之前的字符 `U+000D`（CR）：这对字符会预先转换为单个 `U+000A`（LF）。

r[expr.literal.escape]
## 转义

r[expr.literal.escape.intro]
以下文本字面量表达式的描述中使用了多种形式的*转义*。

r[expr.literal.escape.sequence]
每种形式的转义具有以下特征：
 * 一个*转义序列*：一个字符序列，始终以 `U+005C`（`\`）开头
 * 一个*转义值*：单个字符或空字符序列

在下面的转义定义中：
 * *八进制数字*是范围 \[`0`-`7`] 中的任何字符。
 * *十六进制数字*是范围 \[`0`-`9`]、\[`a`-`f`] 或 \[`A`-`F`] 中的任何字符。

r[expr.literal.escape.simple]
### 简单转义

下表中第一列出现的每个字符序列都是一个转义序列。

在每种情况下，转义值是第二列相应条目中给出的字符。

| 转义序列       | 转义值                   |
|----------------|--------------------------|
| `\0`           | U+0000 (NUL)             |
| `\t`           | U+0009 (HT)              |
| `\n`           | U+000A (LF)              |
| `\r`           | U+000D (CR)              |
| `\"`           | U+0022 (QUOTATION MARK)  |
| `\'`           | U+0027 (APOSTROPHE)      |
| `\\`           | U+005C (REVERSE SOLIDUS) |

r[expr.literal.escape.hex-octet]
### 8 位转义

转义序列由 `\x` 后跟两个十六进制数字组成。

转义值是其 [Unicode 标量值][Unicode scalar value]等于将转义序列最后两个字符解释为十六进制整数的字符，如同使用 [`u8::from_str_radix`] 以基数 16 解释。

> [!NOTE]
> 因此，转义值的 [Unicode 标量值][Unicode scalar value]在 [`u8`][numeric types] 的范围内。

r[expr.literal.escape.hex-ascii]
### 7 位转义

转义序列由 `\x` 后跟一个八进制数字和一个十六进制数字组成。

转义值是其 [Unicode 标量值][Unicode scalar value]等于将转义序列最后两个字符解释为十六进制整数的字符，如同使用 [`u8::from_str_radix`] 以基数 16 解释。

r[expr.literal.escape.unicode]
### Unicode 转义

转义序列由 `\u{` 后跟一串字符（每个字符是十六进制数字或 `_`）再后跟 `}` 组成。

转义值是其 [Unicode 标量值][Unicode scalar value]等于将转义序列中包含的十六进制数字解释为十六进制整数的字符，如同使用 [`u32::from_str_radix`] 以基数 16 解释。

> [!NOTE]
> [CHAR_LITERAL] 或 [STRING_LITERAL] 记号的允许形式确保存在这样一个字符。

r[expr.literal.continuation]
### 字符串续行转义

转义序列由 `\` 后紧跟 `U+000A`（LF）以及下一个非空白字符之前的所有空白字符组成。为此，空白字符是 `U+0009`（HT）、`U+000A`（LF）、`U+000D`（CR）和 `U+0020`（SPACE）。

转义值是一个空字符序列。

> [!NOTE]
> 这种转义形式的效果是字符串续行会跳过后续的空白字符，包括额外的换行符。因此 `a`、`b` 和 `c` 是相等的：
>
> ```rust
> let a = "foobar";
> let b = "foo\
>          bar";
> let c = "foo\
>
>      bar";
>
> assert_eq!(a, b);
> assert_eq!(b, c);
> ```
>
> 跳过额外的换行符（如示例 c 中）可能令人困惑和意外。这种行为将来可能会调整。在做出决定之前，建议避免依赖用行续行跳过多个换行符。有关更多信息，请参阅[此议题](https://github.com/rust-lang/reference/pull/1042)。

r[expr.literal.char]
## 字符字面量表达式

r[expr.literal.char.intro]
字符字面量表达式由单个 [CHAR_LITERAL] 记号组成。

r[expr.literal.char.type]
该表达式的类型是原始 [`char`] 类型。

r[expr.literal.char.no-suffix]
记号不能有后缀。

r[expr.literal.char.literal-content]
记号的*字面量内容*是在记号字符串表示中跟在第一个 `U+0027`（`'`）之后且在最后一个 `U+0027`（`'`）之前的字符序列。

r[expr.literal.char.represented]
字面量表达式的*表示字符*按如下方式从字面量内容派生：

r[expr.literal.char.escape]
* 如果字面量内容是以下形式的转义序列之一，则表示字符是该转义序列的转义值：
    * [简单转义][Simple escapes]
    * [7 位转义][7-bit escapes]
    * [Unicode 转义][Unicode escapes]

r[expr.literal.char.single]
* 否则，表示字符是构成字面量内容的单个字符。

r[expr.literal.char.result]
表达式的值是与表示字符的 [Unicode 标量值][Unicode scalar value]相对应的 [`char`]。

> [!NOTE]
> [CHAR_LITERAL] 记号的允许形式确保这些规则总是产生单个字符。

字符字面量表达式示例：

```rust
'R';                               // R
'\'';                              // '
'\x52';                            // R
'\u{00E6}';                        // 拉丁文小写字母 AE (U+00E6)
```

r[expr.literal.string]
## 字符串字面量表达式

r[expr.literal.string.intro]
字符串字面量表达式由单个 [STRING_LITERAL] 或 [RAW_STRING_LITERAL] 记号组成。

r[expr.literal.string.type]
该表达式的类型是对原始 [`str`] 类型的共享引用（具有 `static` 生命周期）。即类型为 `&'static str`。

r[expr.literal.string.no-suffix]
记号不能有后缀。

r[expr.literal.string.literal-content]
记号的*字面量内容*是在记号字符串表示中跟在第一个 `U+0022`（`"`）之后且在最后一个 `U+0022`（`"`）之前的字符序列。

r[expr.literal.string.represented]
字面量表达式的*表示字符串*是按如下方式从字面量内容派生的字符序列：

r[expr.literal.string.escape]
* 如果记号是 [STRING_LITERAL]，则字面量内容中出现的以下任何形式的每个转义序列都被该转义序列的转义值替换。
    * [简单转义][Simple escapes]
    * [7 位转义][7-bit escapes]
    * [Unicode 转义][Unicode escapes]
    * [字符串续行转义][String continuation escapes]

  这些替换按从左到右的顺序进行。例如，记号 `"\\x41"` 被转换为字符 `\` `x` `4` `1`。

r[expr.literal.string.raw]
* 如果记号是 [RAW_STRING_LITERAL]，则表示字符串与字面量内容完全相同。

r[expr.literal.string.result]
表达式的值是对一个静态分配的 [`str`] 的引用，该 [`str`] 包含表示字符串的 UTF-8 编码。

字符串字面量表达式示例：

```rust
"foo"; r"foo";                     // foo
"\"foo\""; r#""foo""#;             // "foo"

"foo #\"# bar";
r##"foo #"# bar"##;                // foo #"# bar

"\x52"; "R"; r"R";                 // R
"\\x52"; r"\x52";                  // \x52
```

r[expr.literal.byte-char]
## 字节字面量表达式

r[expr.literal.byte-char.intro]
字节字面量表达式由单个 [BYTE_LITERAL] 记号组成。

r[expr.literal.byte-char.literal]
该表达式的类型是原始 [`u8`][numeric types] 类型。

r[expr.literal.byte-char.no-suffix]
记号不能有后缀。

r[expr.literal.byte-char.literal-content]
记号的*字面量内容*是在记号字符串表示中跟在第一个 `U+0027`（`'`）之后且在最后一个 `U+0027`（`'`）之前的字符序列。

r[expr.literal.byte-char.represented]
字面量表达式的*表示字符*按如下方式从字面量内容派生：

r[expr.literal.byte-char.escape]
* 如果字面量内容是以下形式的转义序列之一，则表示字符是该转义序列的转义值：
    * [简单转义][Simple escapes]
    * [8 位转义][8-bit escapes]

r[expr.literal.byte-char.single]
* 否则，表示字符是构成字面量内容的单个字符。

r[expr.literal.byte-char.result]
表达式的值是表示字符的 [Unicode 标量值][Unicode scalar value]。

> [!NOTE]
> [BYTE_LITERAL] 记号的允许形式确保这些规则总是产生单个字符，其 Unicode 标量值在 [`u8`][numeric types] 的范围内。

字节字面量表达式示例：

```rust
b'R';                              // 82
b'\'';                             // 39
b'\x52';                           // 82
b'\xA0';                           // 160
```

r[expr.literal.byte-string]
## 字节串字面量表达式

r[expr.literal.byte-string.intro]
字节串字面量表达式由单个 [BYTE_STRING_LITERAL] 或 [RAW_BYTE_STRING_LITERAL] 记号组成。

r[expr.literal.byte-string.type]
该表达式的类型是对一个数组的共享引用（具有 `static` 生命周期），其元素类型为 [`u8`][numeric types]。即类型为 `&'static [u8; N]`，其中 `N` 是下文所述表示字符串中的字节数。

r[expr.literal.byte-string.no-suffix]
记号不能有后缀。

r[expr.literal.byte-string.literal-content]
记号的*字面量内容*是在记号字符串表示中跟在第一个 `U+0022`（`"`）之后且在最后一个 `U+0022`（`"`）之前的字符序列。

r[expr.literal.byte-string.represented]
字面量表达式的*表示字符串*是按如下方式从字面量内容派生的字符序列：

r[expr.literal.byte-string.escape]
* 如果记号是 [BYTE_STRING_LITERAL]，则字面量内容中出现的以下任何形式的每个转义序列都被该转义序列的转义值替换。
    * [简单转义][Simple escapes]
    * [8 位转义][8-bit escapes]
    * [字符串续行转义][String continuation escapes]

  这些替换按从左到右的顺序进行。例如，记号 `b"\\x41"` 被转换为字符 `\` `x` `4` `1`。

r[expr.literal.byte-string.raw]
* 如果记号是 [RAW_BYTE_STRING_LITERAL]，则表示字符串与字面量内容完全相同。

r[expr.literal.byte-string.result]
表达式的值是对一个静态分配的数组的引用，该数组按相同顺序包含表示字符串中每个字符的 [Unicode 标量值][Unicode scalar values]。

> [!NOTE]
> [BYTE_STRING_LITERAL] 和 [RAW_BYTE_STRING_LITERAL] 记号的允许形式确保这些规则始终产生在 [`u8`][numeric types] 范围内的数组元素值。

字节串字面量表达式示例：

```rust
b"foo"; br"foo";                     // foo
b"\"foo\""; br#""foo""#;             // "foo"

b"foo #\"# bar";
br##"foo #"# bar"##;                 // foo #"# bar

b"\x52"; b"R"; br"R";                // R
b"\\x52"; br"\x52";                  // \x52
```

r[expr.literal.c-string]
## C 字符串字面量表达式

r[expr.literal.c-string.intro]
C 字符串字面量表达式由单个 [C_STRING_LITERAL] 或 [RAW_C_STRING_LITERAL] 记号组成。

r[expr.literal.c-string.type]
该表达式的类型是对标准库 [CStr] 类型的共享引用（具有 `static` 生命周期）。即类型为 `&'static core::ffi::CStr`。

r[expr.literal.c-string.no-suffix]
记号不能有后缀。

r[expr.literal.c-string.literal-content]
记号的*字面量内容*是在记号字符串表示中跟在第一个 `"` 之后且在最后一个 `"` 之前的字符序列。

r[expr.literal.c-string.represented]
字面量表达式的*表示字节*是按如下方式从字面量内容派生的字节序列：

r[expr.literal.c-string.escape]
* 如果记号是 [C_STRING_LITERAL]，字面量内容被视为一个项序列，每一项要么是除 `\` 之外的单个 Unicode 字符，要么是一个[转义][Escape]。该序列按如下方式转换为字节序列：
  * 每个单个 Unicode 字符贡献其 UTF-8 表示。
  * 每个[简单转义][simple escape]贡献其转义值的 [Unicode 标量值][Unicode scalar value]。
  * 每个 [8 位转义][8-bit escape]贡献一个包含其转义值的 [Unicode 标量值][Unicode scalar value]的单个字节。
  * 每个 [Unicode 转义][unicode escape]贡献其转义值的 UTF-8 表示。
  * 每个[字符串续行转义][string continuation escape]不贡献任何字节。

r[expr.literal.c-string.raw]
* 如果记号是 [RAW_C_STRING_LITERAL]，则表示字节是字面量内容的 UTF-8 编码。

> [!NOTE]
> [C_STRING_LITERAL] 和 [RAW_C_STRING_LITERAL] 记号的允许形式确保表示字节绝不包含空字节。

r[expr.literal.c-string.result]
表达式的值是对一个静态分配的 [CStr] 的引用，其字节数组包含表示字节后跟一个空字节。

C 字符串字面量表达式示例：

```rust
c"foo"; cr"foo";                     // foo
c"\"foo\""; cr#""foo""#;             // "foo"

c"foo #\"# bar";
cr##"foo #"# bar"##;                 // foo #"# bar

c"\x52"; c"R"; cr"R";                // R
c"\\x52"; cr"\x52";                  // \x52

c"æ";                                // 拉丁文小写字母 AE (U+00E6)
c"\u{00E6}";                         // 拉丁文小写字母 AE (U+00E6)
c"\xC3\xA6";                         // 拉丁文小写字母 AE (U+00E6)

c"\xE6".to_bytes();                  // [230]
c"\u{00E6}".to_bytes();              // [195, 166]
```

r[expr.literal.int]
## 整数字面量表达式

r[expr.literal.int.intro]
整数字面量表达式由单个 [INTEGER_LITERAL] 记号组成。

r[expr.literal.int.suffix]
如果记号有[后缀][suffix]，则该后缀必须是[原始整数类型][numeric types]之一：`u8`、`i8`、`u16`、`i16`、`u32`、`i32`、`u64`、`i64`、`u128`、`i128`、`usize` 或 `isize`，并且表达式具有该类型。

r[expr.literal.int.infer]
如果记号没有后缀，则表达式的类型通过类型推断确定：

r[expr.literal.int.inference-unique-type]
* 如果可以从周围的程序上下文*唯一地*确定一个整数类型，则表达式具有该类型。

r[expr.literal.int.inference-default]
* 如果程序上下文对类型的约束不足，则默认为有符号 32 位整数 `i32`。

r[expr.literal.int.inference-error]
* 如果程序上下文对类型的约束过多，则被视为静态类型错误。

整数字面量表达式示例：

```rust
123;                               // 类型 i32
123i32;                            // 类型 i32
123u32;                            // 类型 u32
123_u32;                           // 类型 u32
let a: u64 = 123;                  // 类型 u64

0xff;                              // 类型 i32
0xff_u8;                           // 类型 u8

0o70;                              // 类型 i32
0o70_i16;                          // 类型 i16

0b1111_1111_1001_0000;             // 类型 i32
0b1111_1111_1001_0000i64;          // 类型 i64

0usize;                            // 类型 usize
```

r[expr.literal.int.representation]
表达式的值按如下方式从记号的字符串表示确定：

r[expr.literal.int.radix]
* 通过检查字符串的前两个字符选择一个整数基数，规则如下：

    * `0b` 表示基数为 2
    * `0o` 表示基数为 8
    * `0x` 表示基数为 16
    * 否则基数为 10。

r[expr.literal.int.radix-prefix-stripped]
* 如果基数不是 10，则从字符串中删除前两个字符。

r[expr.literal.int.type-suffix-stripped]
* 从字符串中删除任何后缀。

r[expr.literal.int.separators-stripped]
* 从字符串中删除任何下划线。

r[expr.literal.int.u128-value]
* 将字符串转换为 `u128` 值，如同使用 [`u128::from_str_radix`] 并以所选基数转换。如果该值不适合 `u128`，则为编译错误。

r[expr.literal.int.cast]
* 通过[数值转换][numeric cast]将 `u128` 值转换为表达式的类型。

> [!NOTE]
> 如果字面量的值不适合表达式类型，最终转换将截断该值。`rustc` 包含一个名为 `overflowing_literals` 的 [lint 检查][lint check]，默认值为 `deny`，当发生这种情况时拒绝表达式。

> [!NOTE]
> 例如 `-1i8` 是[取反运算符][negation operator]应用于字面量表达式 `1i8`，而不是单个整数字面量表达式。参见[溢出][Overflow]获取有关表示有符号类型最负值的说明。

r[expr.literal.float]
## 浮点字面量表达式

r[expr.literal.float.intro]
浮点字面量表达式有两种形式之一：
 * 一个单独的 [FLOAT_LITERAL] 记号
 * 一个有后缀且无基数指示符的单独 [INTEGER_LITERAL] 记号

r[expr.literal.float.suffix]
如果记号有[后缀][suffix]，则该后缀必须是[原始浮点类型][floating-point types]之一：`f32` 或 `f64`，并且表达式具有该类型。

r[expr.literal.float.infer]
如果记号没有后缀，则表达式的类型通过类型推断确定：

r[expr.literal.float.inference-unique-type]
* 如果可以从周围的程序上下文*唯一地*确定一个浮点类型，则表达式具有该类型。

r[expr.literal.float.inference-default]
* 如果程序上下文对类型的约束不足，则默认为 `f64`。

r[expr.literal.float.inference-error]
* 如果程序上下文对类型的约束过多，则被视为静态类型错误。

浮点字面量表达式示例：

```rust
123.0f64;        // 类型 f64
0.1f64;          // 类型 f64
0.1f32;          // 类型 f32
12E+99_f64;      // 类型 f64
5f32;            // 类型 f32
let x: f64 = 2.; // 类型 f64
```

r[expr.literal.float.result]
表达式的值按如下方式从记号的字符串表示确定：

r[expr.literal.float.type-suffix-stripped]
* 从字符串中删除任何后缀。

r[expr.literal.float.separators-stripped]
* 从字符串中删除任何下划线。

r[expr.literal.float.value]
* 将字符串转换为表达式的类型，如同使用 [`f32::from_str`] 或 [`f64::from_str`]。

> [!NOTE]
> 例如 `-1.0` 是[取反运算符][negation operator]应用于字面量表达式 `1.0`，而不是单个浮点字面量表达式。

> [!NOTE]
> `inf` 和 `NaN` 不是字面量记号。可以使用 [`f32::INFINITY`]、[`f64::INFINITY`]、[`f32::NAN`] 和 [`f64::NAN`] 常量来代替字面量表达式。在 `rustc` 中，足够大到被求值为无穷大的字面量将触发 `overflowing_literals` lint 检查。

r[expr.literal.bool]
## 布尔字面量表达式

r[expr.literal.bool.intro]
布尔字面量表达式由关键字 `true` 或 `false` 之一组成。

r[expr.literal.bool.result]
表达式的类型是原始[布尔类型][boolean type]，其值为：
 * 如果关键字是 `true`，则为 true
 * 如果关键字是 `false`，则为 false

[Escape]: #escapes
[Simple escape]: #simple-escapes
[Simple escapes]: #simple-escapes
[8-bit escape]: #8-bit-escapes
[8-bit escapes]: #8-bit-escapes
[7-bit escape]: #7-bit-escapes
[7-bit escapes]: #7-bit-escapes
[Unicode escape]: #unicode-escapes
[Unicode escapes]: #unicode-escapes
[String continuation escape]: #string-continuation-escapes
[String continuation escapes]: #string-continuation-escapes
[boolean type]: ../types/boolean.md
[constant expression]: ../const_eval.md#constant-expressions
[CStr]: core::ffi::CStr
[floating-point types]: ../types/numeric.md#floating-point-types
[lint check]: ../attributes/diagnostics.md#lint-check-attributes
[literal tokens]: ../tokens.md#literals
[numeric cast]: operator-expr.md#numeric-cast
[numeric types]: ../types/numeric.md
[suffix]: ../tokens.md#suffixes
[negation operator]: operator-expr.md#negation-operators
[overflow]: operator-expr.md#overflow
[Unicode scalar value]: http://www.unicode.org/glossary/#unicode_scalar_value
[Unicode scalar values]: http://www.unicode.org/glossary/#unicode_scalar_value
[`char`]: ../types/char.md
[`f32::from_str`]: ../../core/primitive.f32.md#method.from_str
[`f64::from_str`]: ../../core/primitive.f64.md#method.from_str
[`str`]: ../types/str.md
