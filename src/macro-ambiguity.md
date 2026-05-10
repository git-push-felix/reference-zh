r[macro.ambiguity]
# 附录：宏跟随集歧义形式规范

本页记录了[声明宏][Macros By Example]的跟随规则的形式规范。它们最初在 [RFC 550] 中指定，本文大部分内容复制自那里，并在后续 RFC 中扩展。

r[macro.ambiguity.convention]
## 定义和约定

r[macro.ambiguity.convention.defs]
  - `macro`：任何可以在源代码中作为 `foo!(...)` 调用的东西。
  - `MBE`：声明宏，由 `macro_rules` 定义的宏。
  - `matcher`：`macro_rules` 调用中某条规则的左手边，或其子部分。
  - `macro parser`：Rust 解析器中用于使用从所有匹配器派生的语法解析输入的代码部分。
  - `fragment`：给定匹配器将接受（或"匹配"）的 Rust 语法类别。
  - `repetition`：一个遵循常规重复模式的片段
  - `NT`：非终结符，可以出现在匹配器中的各种"元变量"或重复匹配器，在 MBE 语法中以前导 `$` 字符指定。
  - `simple NT`：一种"元变量"非终结符（下文进一步讨论）。
  - `complex NT`：一种重复匹配非终结符，通过重复运算符（`*`、`+`、`?`）指定。
  - `token`：匹配器的原子元素；即标识符、运算符、开/闭分隔符，*以及*简单 NT。
  - `token tree`：由 token（叶子）、复杂 NT 和有限序列的 token tree 组成的树结构。
  - `delimiter token`：一个 token，旨在分隔一个片段的结束和下一个片段的开始。
  - `separator token`：复杂 NT 中可选的定界 token，分隔匹配重复中每对元素。
  - `separated complex NT`：具有自己的分隔 token 的复杂 NT。
  - `delimited sequence`：具有适当的开分隔符和闭分隔符的 token tree 序列。
  - `empty fragment`：分隔 token 的不可见 Rust 语法类别，即空白，或（在某些词法上下文中）空 token 序列。
  - `fragment specifier`：简单 NT 中的标识符，指定该 NT 接受哪个片段。
  - `language`：一个上下文无关语言。

示例：

```rust,compile_fail
macro_rules! i_am_an_mbe {
    (start $foo:expr $($i:ident),* end) => ($foo)
}
```

r[macro.ambiguity.convention.matcher]
`(start $foo:expr $($i:ident),* end)` 是一个匹配器。整个匹配器是一个定界序列（具有开分隔符和闭分隔符 `(` 和 `)`），`$foo` 和 `$i` 是简单 NT，其片段指示符分别为 `expr` 和 `ident`。

r[macro.ambiguity.convention.complex-nt]
`$(i:ident),*` *也*是一个 NT；它是一个复杂 NT，匹配以逗号分隔的标识符重复。`,` 是该复杂 NT 的分隔 token；它出现在已匹配片段的每对元素（如果有的话）之间。

复杂 NT 的另一个示例是 `$(hi $e:expr ;)+`，它匹配形式为 `hi <expr>; hi <expr>; ...` 的任何片段，其中 `hi <expr>;` 至少出现一次。请注意，此复杂 NT 没有专用的分隔 token。

（请注意，Rust 的解析器确保定界序列始终以适当的 token tree 结构嵌套和正确的开/闭分隔符匹配出现。）

r[macro.ambiguity.convention.vars]
我们通常使用变量 "M" 表示匹配器，变量 "t" 和 "u" 表示任意单独的 token，变量 "tt" 和 "uu" 表示任意 token tree。（使用 "tt" 确实可能与其作为片段指示符的附加角色产生歧义；但从上下文中可以清楚地看出哪种解释是预期的。）

r[macro.ambiguity.convention.set]
"SEP" 范围在分隔 token 上，"OP" 范围在重复运算符 `*`、`+` 和 `?` 上，"OPEN"/"CLOSE" 范围在包围定界序列的匹配 token 对上（例如 `[` 和 `]`）。

r[macro.ambiguity.convention.sequence-vars]
希腊字母 "α" "β" "γ" "δ" 表示可能为空的 token-tree 序列。（然而，希腊字母 "ε"（epsilon）在表述中具有特殊角色，不表示 token-tree 序列。）

  * 此希腊字母约定通常仅在序列的存在是技术细节时使用；特别是，当我们希望*强调*正在对 token-tree 序列进行操作时，我们将对序列使用符号 "tt ..."，而不是希腊字母。

请注意，匹配器仅仅是一个 token tree。如上所述的"简单 NT"是一个元变量 NT；因此它不是重复。例如，`$foo:ty` 是一个简单 NT，但 `$($foo:ty)+` 是一个复杂 NT。

另请注意，在此形式规范的上下文中，术语"token"通常*包括*简单 NT。

最后，读者应记住，根据此形式规范的定义，没有简单 NT 匹配空片段，同样没有 token 匹配 Rust 语法的空片段。（因此，*唯一*可以匹配空片段的 NT 是复杂 NT。）这实际上并非完全正确，因为 `vis` 匹配器可以匹配空片段。因此，在此形式规范中，我们将 `$v:vis` 视为实际上是 `$($v:vis)?`，并要求匹配器匹配空片段。

r[macro.ambiguity.invariant]
### 匹配器不变式

r[macro.ambiguity.invariant.list]
要成为有效的，匹配器必须满足以下三个不变式。FIRST 和 FOLLOW 的定义稍后描述。

1.  对于匹配器 `M` 中的任何两个连续的 token tree 序列（即 `M = ... tt uu ...`）且 `uu ...` 非空，我们必须有 FOLLOW(`... tt`) ∪ {ε} ⊇ FIRST(`uu ...`)。
1.  对于匹配器中的任何带分隔的复杂 NT，`M = ... $(tt ...) SEP OP ...`，我们必须有 `SEP` ∈ FOLLOW(`tt ...`)。
1.  对于匹配器中的无分隔复杂 NT，`M = ... $(tt ...) OP ...`，如果 OP = `*` 或 `+`，我们必须有 FOLLOW(`tt ...`) ⊇ FIRST(`tt ...`)。

r[macro.ambiguity.invariant.follow-matcher]
第一个不变式表示，匹配器之后出现的任何实际 token（如果有的话）必须在预定的跟随集中。这确保了合法的宏定义将继续对 `... tt` 的结束位置和 `uu ...` 的开始位置分配相同的判断，即使新的语法形式被添加到语言中。

r[macro.ambiguity.invariant.separated-complex-nt]
第二个不变式表示，带分隔的复杂 NT 必须使用作为 NT 内部内容的预定跟随集一部分的分隔 token。这确保了合法的宏定义将继续将输入片段解析为相同的 `tt ...` 定界序列，即使新的语法形式被添加到语言中。

r[macro.ambiguity.invariant.unseparated-complex-nt]
第三个不变式表示，当我们有一个可以匹配同一事物的两个或多个副本且中间没有分隔的复杂 NT 时，必须允许它们按照第一个不变式彼此相邻放置。此不变式还要求它们非空，这消除了一种可能的歧义。

**注意：由于历史上的疏忽和对该行为的重大依赖，第三个不变式目前未强制执行。目前尚未决定如何处理此事。不遵守该行为的宏可能在 Rust 的未来版次中变为无效。请参见[跟踪 issue][tracking issue]。**

r[macro.ambiguity.sets]
### FIRST 和 FOLLOW，非正式地

r[macro.ambiguity.sets.intro]
给定的匹配器 M 映射到三个集合：FIRST(M)、LAST(M) 和 FOLLOW(M)。

三个集合中的每一个都由 token 组成。FIRST(M) 和 LAST(M) 还可能包含一个特殊的非 token 元素 ε（"epsilon"），表示 M 可以匹配空片段。（但 FOLLOW(M) 始终只是 token 的集合。）

非正式地：

r[macro.ambiguity.sets.first]
  * FIRST(M)：收集在将片段匹配到 M 时可能首先使用的 token。

r[macro.ambiguity.sets.last]
  * LAST(M)：收集在将片段匹配到 M 时可能最后使用的 token。

r[macro.ambiguity.sets.follow]
  * FOLLOW(M)：允许紧跟在由 M 匹配的某个片段之后的 token 集合。

    换句话说：t ∈ FOLLOW(M) 当且仅当存在（可能为空的）token 序列 α, β, γ, δ，其中：

      * M 匹配 β，

      * t 匹配 γ，且

      * 连接 α β γ δ 是一个可解析的 Rust 程序。

r[macro.ambiguity.sets.universe]
我们使用简写 ANYTOKEN 表示所有 token（包括简单 NT）的集合。例如，如果任何 token 在匹配器 M 之后都是合法的，那么 FOLLOW(M) = ANYTOKEN。

r[macro.ambiguity.sets.def]
### FIRST、LAST

r[macro.ambiguity.sets.def.intro]
以下是 FIRST 和 LAST 的形式归纳定义。

r[macro.ambiguity.sets.def.notation]
"A ∪ B" 表示集合并，"A ∩ B" 表示集合交，"A \ B" 表示集合差（即 A 中不在 B 中的所有元素）。

r[macro.ambiguity.sets.def.first]
#### FIRST

r[macro.ambiguity.sets.def.first.intro]
FIRST(M) 通过对序列 M 及其第一个 token-tree（如果有）的结构进行案例分析来定义：

r[macro.ambiguity.sets.def.first.epsilon]
  * 如果 M 是空序列，则 FIRST(M) = { ε }，

r[macro.ambiguity.sets.def.first.token]
  * 如果 M 以 token t 开头，则 FIRST(M) = { t }，

    （注意：这涵盖了 M 以定界 token-tree 序列开头的情况，`M = OPEN tt ... CLOSE ...`，其中 `t = OPEN`，因此 FIRST(M) = { `OPEN` }。）

    （注意：这关键依赖于没有简单 NT 匹配空片段的性质。）

r[macro.ambiguity.sets.def.first.complex]
  * 否则，M 是一个以复杂 NT 开头的 token-tree 序列：`M = $( tt ... ) OP α`，或 `M = $( tt ... ) SEP OP α`，（其中 `α` 是匹配器其余部分的（可能为空的）token tree 序列）。

      * 令 SEP\_SET(M) = { SEP } 如果 SEP 存在且 ε ∈ FIRST(`tt ...`)；否则 SEP\_SET(M) = {}。

  * 令 ALPHA\_SET(M) = FIRST(`α`) 如果 OP = `*` 或 `?`，且如果 OP = `+` 则 ALPHA\_SET(M) = {}。
  * FIRST(M) = (FIRST(`tt ...`) \\ {ε}) ∪ SEP\_SET(M) ∪ ALPHA\_SET(M)。

对复杂 NT 的定义值得一些论证。SEP\_SET(M) 定义了分隔符可能是 M 的有效第一个 token 的可能性，这发生在定义了分隔符且重复的片段可能为空时。ALPHA\_SET(M) 定义了复杂 NT 可能为空的可能性，这意味着 M 的有效第一个 token 是后续 token-tree 序列 `α` 的 token。这发生在使用 `*` 或 `?` 时，此时可能有零次重复。理论上，如果使用 `+` 与可能为空的重复片段，这也可能发生，但第三个不变式禁止了这点。

由此，显然 FIRST(M) 可以包括来自 SEP\_SET(M) 或 ALPHA\_SET(M) 的任何 token，如果复杂 NT 匹配非空，那么任何开始 FIRST(`tt ...`) 的 token 也可以工作。最后要考虑的是 ε。SEP\_SET(M) 和 FIRST(`tt ...`) \ {ε} 不能包含 ε，但 ALPHA\_SET(M) 可以。因此，此定义允许 M 接受 ε 当且仅当 ε ∈ ALPHA\_SET(M)。这是正确的，因为在复杂 NT 情况下，要让 M 接受 ε，复杂 NT 和 α 都必须接受它。如果 OP = `+`，意味着复杂 NT 不能为空，则根据定义 ε ∉ ALPHA\_SET(M)。否则，复杂 NT 可以接受零次重复，然后 ALPHA\_SET(M) = FOLLOW(`α`)。所以此定义相对于 ε 也是正确的。

r[macro.ambiguity.sets.def.last]
#### LAST

r[macro.ambiguity.sets.def.last.intro]
LAST(M)，通过案例分析 M 本身（一个 token-tree 序列）定义：

r[macro.ambiguity.sets.def.last.empty]
  * 如果 M 是空序列，则 LAST(M) = { ε }

r[macro.ambiguity.sets.def.last.token]
  * 如果 M 是单个 token t，则 LAST(M) = { t }

r[macro.ambiguity.sets.def.last.rep-star]
  * 如果 M 是重复零次或多次的单个复杂 NT，`M = $( tt ... ) *`，或 `M = $( tt ... ) SEP *`

      * 令 sep_set = { SEP } 如果 SEP 存在；否则 sep_set = {}。

      * 如果 ε ∈ LAST(`tt ...`)，则 LAST(M) = LAST(`tt ...`) ∪ sep_set

      * 否则，序列 `tt ...` 必须非空；LAST(M) = LAST(`tt ...`) ∪ {ε}。

r[macro.ambiguity.sets.def.last.rep-plus]
  * 如果 M 是重复一次或多次的单个复杂 NT，`M = $( tt ... ) +`，或 `M = $( tt ... ) SEP +`

      * 令 sep_set = { SEP } 如果 SEP 存在；否则 sep_set = {}。

      * 如果 ε ∈ LAST(`tt ...`)，则 LAST(M) = LAST(`tt ...`) ∪ sep_set

      * 否则，序列 `tt ...` 必须非空；LAST(M) = LAST(`tt ...`)

r[macro.ambiguity.sets.def.last.rep-question]
  * 如果 M 是重复零次或一次的单个复杂 NT，`M = $( tt ...) ?`，则 LAST(M) = LAST(`tt ...`) ∪ {ε}。

r[macro.ambiguity.sets.def.last.delim]
  * 如果 M 是一个定界 token-tree 序列 `OPEN tt ... CLOSE`，则 LAST(M) = { `CLOSE` }。

r[macro.ambiguity.sets.def.last.sequence]
  * 如果 M 是非空 token-tree 序列 `tt uu ...`，

      * 如果 ε ∈ LAST(`uu ...`)，则 LAST(M) = LAST(`tt`) ∪ (LAST(`uu ...`) \ { ε })。

      * 否则，序列 `uu ...` 必须非空；则 LAST(M) = LAST(`uu ...`)。

### FIRST 和 LAST 的示例

以下是一些 FIRST 和 LAST 的示例。（特别注意特殊元素 ε 如何基于输入的各个部分之间的交互被引入和消除。）

我们的第一个示例以树结构呈现，以详细说明匹配器的分析如何组合。

```text
INPUT:  $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
            ~~~~~~~~   ~~~~~~~                ~
                |         |                   |
FIRST:   { $d:ident }  { $e:expr }          { h }


INPUT:  $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+
            ~~~~~~~~~~~~~~~~~~             ~~~~~~~           ~~~
                        |                      |               |
FIRST:          { $d:ident }               { h, ε }         { f }

INPUT:  $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~    ~~~~~~~~~~~~~~    ~~~~~~~~~   ~
                        |                       |              |       |
FIRST:        { $d:ident, ε }            {  h, ε, ;  }      { f }   { g }


INPUT:  $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                        |
FIRST:                       { $d:ident, h, ;,  f }
```

因此：

 * FIRST(`$($d:ident $e:expr );* $( $(h)* );* $( f ;)+ g`) = { `$d:ident`, `h`, `;`, `f` }

但请注意：

 * FIRST(`$($d:ident $e:expr );* $( $(h)* );* $($( f ;)+ g)*`) = { `$d:ident`, `h`, `;`, `f`, ε }

以下是类似的示例，但现在是 LAST。

 * LAST(`$d:ident $e:expr`) = { `$e:expr` }
 * LAST(`$( $d:ident $e:expr );*`) = { `$e:expr`, ε }
 * LAST(`$( $d:ident $e:expr );* $(h)*`) = { `$e:expr`, ε, `h` }
 * LAST(`$( $d:ident $e:expr );* $(h)* $( f ;)+`) = { `;` }
 * LAST(`$( $d:ident $e:expr );* $(h)* $( f ;)+ g`) = { `g` }

r[macro.ambiguity.sets.def.follow]
### FOLLOW(M)

r[macro.ambiguity.sets.def.follow.intro]
最后，FOLLOW(M) 的定义如下。pat、expr 等表示具有给定片段指示符的简单非终结符。

r[macro.ambiguity.sets.def.follow.pat]
  * FOLLOW(pat) = {`=>`, `,`, `=`, `|`, `if`, `in`}`。

r[macro.ambiguity.sets.def.follow.expr-stmt]
  * FOLLOW(expr) = FOLLOW(expr_2021) = FOLLOW(stmt) = {`=>`, `,`, `;`}`。

r[macro.ambiguity.sets.def.follow.ty-path]
  * FOLLOW(ty) = FOLLOW(path) = {`{`, `[`, `,`, `=>`, `:`, `=`, `>`, `>>`, `;`, `|`, `as`, `where`, 块非终结符}。

r[macro.ambiguity.sets.def.follow.vis]
  * FOLLOW(vis) = {`,`l 除非原始 `priv` 之外的任何关键字或标识符；任何可以开始类型的 token；ident、ty 和 path 非终结符}。

r[macro.ambiguity.sets.def.follow.simple]
  * FOLLOW(t) = ANYTOKEN 对于任何其他简单 token，包括 block、ident、tt、item、lifetime、literal 和 meta 简单非终结符，以及所有终结符。

r[macro.ambiguity.sets.def.follow.other-matcher]
  * FOLLOW(M)，对于任何其他 M，定义为当 t 遍历 (LAST(M) \ {ε}) 时，FOLLOW(t) 的交集。

r[macro.ambiguity.sets.def.follow.type-first]
在撰写本文时，可以开始类型的 token 是 {`(`, `[`, `!`, `*`, `&`, `&&`, `?`, 生命周期, `>`, `>>`, `::`, 任何非关键字标识符, `super`, `self`, `Self`, `extern`, `crate`, `$crate`, `_`, `for`, `impl`, `fn`, `unsafe`, `typeof`, `dyn`}，尽管此列表可能不完整，因为人们不总是记得在添加新 token 时更新附录。

复杂 M 的 FOLLOW 示例：

 * FOLLOW(`$( $d:ident $e:expr )*`) = FOLLOW(`$e:expr`)
 * FOLLOW(`$( $d:ident $e:expr )* $(;)*`) = FOLLOW(`$e:expr`) ∩ ANYTOKEN = FOLLOW(`$e:expr`)
 * FOLLOW(`$( $d:ident $e:expr )* $(;)* $( f |)+`) = ANYTOKEN

### 有效和无效匹配器的示例

有了上述规范，我们可以为为什么特定匹配器是合法的而其他是非法的提供论证。

 * `($ty:ty < foo ,)` : 非法，因为 FIRST(`< foo ,`) = { `<` } ⊈ FOLLOW(`ty`)

 * `($ty:ty , foo <)` : 合法，因为 FIRST(`, foo <`) = { `,` } ⊆ FOLLOW(`ty`)。

 * `($pa:pat $pb:pat $ty:ty ,)` : 非法，因为 FIRST(`$pb:pat $ty:ty ,`) = { `$pb:pat` } ⊈ FOLLOW(`pat`)，并且 FIRST(`$ty:ty ,`) = { `$ty:ty` } ⊈ FOLLOW(`pat`)。

 * `( $($a:tt $b:tt)* ; )` : 合法，因为 FIRST(`$b:tt`) = { `$b:tt` } ⊆ FOLLOW(`tt`) = ANYTOKEN，FIRST(`;`) = { `;` } 也是。

 * `( $($t:tt),* , $(t:tt),* )` : 合法（尽管实际尝试使用此宏将在展开期间发出局部歧义错误）。

 * `($ty:ty $(; not sep)* -)` : 非法，因为 FIRST(`$(; not sep)* -`) = { `;`, `-` } 不在 FOLLOW(`ty`) 中。

 * `($($ty:ty)-+)` : 非法，因为分隔符 `-` 不在 FOLLOW(`ty`) 中。

 * `($($e:expr)*)` : 非法，因为 expr NT 不在 FOLLOW(expr NT) 中。

[Macros by Example]: macros-by-example.md
[RFC 550]: https://github.com/rust-lang/rfcs/blob/master/text/0550-macro-future-proofing.md
[tracking issue]: https://github.com/rust-lang/rust/issues/56575
