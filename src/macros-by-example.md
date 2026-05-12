r[macro.decl]
# 声明宏

r[macro.decl.syntax]
```grammar,macros
MacroRulesDefinition ->
    `macro_rules` `!` IDENTIFIER MacroRulesDef

MacroRulesDef ->
      `(` MacroRules `)` `;`
    | `[` MacroRules `]` `;`
    | `{` MacroRules `}`

MacroRules ->
    MacroRule ( `;` MacroRule )* `;`?

MacroRule ->
    MacroMatcher `=>` MacroTranscriber

MacroMatcher ->
      `(` MacroMatch* `)`
    | `[` MacroMatch* `]`
    | `{` MacroMatch* `}`

MacroMatch ->
      Token _except `$` and [delimiters][lex.token.delim]_
    | MacroMatcher
    | `$` ( IDENTIFIER_OR_KEYWORD _except `crate`_ | RAW_IDENTIFIER ) `:` MacroFragSpec
    | `$` `(` MacroMatch+ `)` MacroRepSep? MacroRepOp

MacroFragSpec ->
      `block` | `expr` | `expr_2021` | `ident` | `item` | `lifetime` | `literal`
    | `meta` | `pat` | `pat_param` | `path` | `stmt` | `tt` | `ty` | `vis`

MacroRepSep -> Token _except [delimiters][lex.token.delim] and [MacroRepOp]_

MacroRepOp -> `*` | `+` | `?`

MacroTranscriber -> DelimTokenTree
```

r[macro.decl.intro]
`macro_rules` 允许用户以声明式的方式定义语法扩展。我们称此类扩展为"声明宏"或简称为"宏"。

每个声明宏都有一个名称，以及一条或多条*规则*。每条规则有两部分：一个*匹配器*，描述其匹配的语法；和一个*转写器*，描述将替换成功匹配调用的语法。匹配器和转写器都必须由分隔符包围。宏可以展开为表达式、语句、项（包括 trait、impl 和外部项）、类型或模式。

r[macro.decl.transcription]
## 转写

r[macro.decl.transcription.intro]
当调用宏时，宏展开器按名称查找宏调用，并依次尝试每条宏规则。它转写第一个成功匹配的规则；如果这导致错误，则不会尝试后续的匹配。

r[macro.decl.transcription.lookahead]
匹配时不执行前瞻；如果编译器无法每次只用一个 token 明确地确定如何解析宏调用，则这是一个错误。在以下示例中，编译器不会前瞻标识符之后的下一个 token 是否是 `)`，尽管这可以让它明确地解析调用：

```rust,compile_fail
macro_rules! ambiguity {
    ($($i:ident)* $j:ident) => { };
}

ambiguity!(error); // Error: local ambiguity
```

r[macro.decl.transcription.syntax]
在匹配器和转写器中，`$` token 用于调用宏引擎的特殊行为（在下面的[元变量](#metavariables)和[重复](#repetitions)中描述）。不属于此类调用的 token 会被逐字匹配和转写，但有一个例外。例外是匹配器的外部分隔符将匹配任何一对分隔符。因此，例如，匹配器 `(())` 将匹配 `{()}` 但不会匹配 `{{}}`。字符 `$` 不能被逐字匹配或转写。

r[macro.decl.transcription.fragment]
### 转发已匹配的片段

当将已匹配的片段转发给另一个声明宏时，第二个宏中的匹配器将看到该片段类型的不透明 AST。第二个宏不能在匹配器中使用字面 token 来匹配这些片段，只能使用相同类型的片段说明符。`ident`、`lifetime` 和 `tt` 片段类型是例外，*可以*通过字面 token 匹配。以下示例说明了此限制：

```rust,compile_fail
macro_rules! foo {
    ($l:expr) => { bar!($l); }
// ERROR:               ^^ no rules expected this token in macro call
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

以下示例说明了如何在匹配 `tt` 片段后直接匹配 token：

```rust
// compiles OK
macro_rules! foo {
    ($l:tt) => { bar!($l); }
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

r[macro.decl.meta]
## 元变量 {#metavariables}

r[macro.decl.meta.intro]
在匹配器中，`$`*名称*`:`*片段说明符*匹配指定类型的 Rust 语法片段，并将其绑定到元变量 `$`*名称*。

r[macro.decl.meta.specifier]
有效的片段说明符有：

  * `block`：一个[块表达式][BlockExpression]
  * `expr`：一个[表达式][Expression]
  * `expr_2021`：一个[表达式][Expression]，但不包括[下划线表达式][UnderscoreExpression]和[常量块表达式][ConstBlockExpression]（参见 [macro.decl.meta.edition2024]）
  * `ident`：一个 [IDENTIFIER_OR_KEYWORD]，不包括 `_`、[RAW_IDENTIFIER] 或 [`$crate`]
  * `item`：一个[项][Item]
  * `lifetime`：一个 [LIFETIME_TOKEN]
  * `literal`：匹配 `-`<sup>?</sup>[字面量表达式][LiteralExpression]
  * `meta`：一个 [Attr]，即属性的内容
  * `pat`：一个[模式][Pattern]（参见 [macro.decl.meta.edition2021]）
  * `pat_param`：一个 [PatternNoTopAlt]
  * `path`：一个 [TypePath]
  * `stmt`：一个[语句][grammar-Statement]，不含尾部分号（需要分号的项语句除外）
  * `tt`：一个 [TokenTree]（一个单独的 [token] 或位于匹配的分隔符 `()`、`[]` 或 `{}` 中的 token）
  * `ty`：一个[类型][grammar-Type]
  * `vis`：一个可能为空的[可见性][Visibility]限定符

r[macro.decl.meta.transcription]
在转写器中，元变量简单地通过 `$`*名称*引用，因为片段类型在匹配器中已指定。元变量被替换为与它们匹配的语法元素。元变量可以被转写多次，或根本不转写。

r[macro.decl.meta.dollar-crate]
关键字元变量 [`$crate`] 可用于引用当前 crate。

r[macro.decl.meta.edition2021]
> [!EDITION-2021]
> 从 2021 版开始，`pat` 片段说明符匹配顶层或模式（即接受 [Pattern]）。
>
> 在 2021 版之前，它们匹配的片段与 `pat_param` 完全相同（即接受 [PatternNoTopAlt]）。
>
> 相关的版次是 `macro_rules!` 定义所在的版次。

r[macro.decl.meta.edition2024]
> [!EDITION-2024]
> 在 2024 版之前，`expr` 片段说明符在顶层不匹配[下划线表达式][UnderscoreExpression]或[常量块表达式][ConstBlockExpression]。它们允许在子表达式中出现。
>
> `expr_2021` 片段说明符存在是为了与 2024 之前的版次保持向后兼容。

r[macro.decl.repetition]
## 重复 {#repetitions}

r[macro.decl.repetition.intro]
在匹配器和转写器中，重复通过将需要重复的 token 放入 `$(`…`)` 中，后跟一个重复运算符，中间可选地有一个分隔符 token 来表示。

r[macro.decl.repetition.separator]
分隔符 token 可以是除分隔符或重复运算符之外的任何 token，但 `;` 和 `,` 是最常见的。例如，`$( $i:ident ),*` 表示任意数量的由逗号分隔的标识符。允许嵌套重复。

r[macro.decl.repetition.operators]
重复运算符有：

- `*` --- 表示任意次数的重复。
- `+` --- 表示至少一次重复。
- `?` --- 表示一个可选的片段，零次或一次出现。

r[macro.decl.repetition.optional-restriction]
由于 `?` 表示至多一次出现，因此不能与分隔符一起使用。

r[macro.decl.repetition.fragment]
重复片段会匹配并转写为指定数量的片段，由分隔符 token 分隔。元变量会匹配其对应片段的每一次重复。例如，上面的 `$( $i:ident ),*` 示例将 `$i` 匹配到列表中的所有标识符。

在转写期间，对重复施加了额外的限制，以便编译器知道如何正确地展开它们：

1.  元变量在转写器中必须以与匹配器中完全相同的数量、种类和重复嵌套顺序出现。因此，对于匹配器 `$( $i:ident ),*`，转写器 `=> { $i }`、`=> { $( $( $i )* )* }` 和 `=> { $( $i )+ }` 都是不合法的，但 `=> { $( $i );* }` 是正确的，并用分号分隔的列表替换逗号分隔的标识符列表。
2.  转写器中的每个重复必须包含至少一个元变量，以决定展开多少次。如果多个元变量出现在同一个重复中，它们必须绑定到相同数量的片段。例如，`( $( $i:ident ),* ; $( $j:ident ),* ) => (( $( ($i,$j) ),* ))` 必须将相同数量的 `$i` 片段和 `$j` 片段绑定。这意味着用 `(a, b, c; d, e, f)` 调用宏是合法的，并展开为 `((a,d), (b,e), (c,f))`，但 `(a, b, c; d, e)` 是不合法的，因为它没有相同的数量。此要求适用于嵌套重复的每一层。

r[macro.decl.scope]
## 作用域、导出和导入

r[macro.decl.scope.intro]
出于历史原因，声明宏的作用域不完全像项那样工作。宏有两种作用域形式：文本作用域和基于路径的作用域。文本作用域基于事物在源文件中出现的顺序，甚至跨越多个文件，也是默认的作用域。下面将进一步解释。基于路径的作用域与项作用域的工作方式完全相同。宏的作用域、导出和导入主要受属性控制。

r[macro.decl.scope.unqualified]
当通过非限定标识符（不是多段路径的一部分）调用宏时，首先在文本作用域中查找。如果没有结果，则在基于路径的作用域中查找。如果宏的名称是通过路径限定的，则仅在基于路径的作用域中查找。

<!-- ignore: requires external crates -->
```rust,ignore
use lazy_static::lazy_static; // 基于路径的导入。

macro_rules! lazy_static { // 文本定义。
    (lazy) => {};
}

lazy_static!{lazy} // 文本查找首先找到我们的宏。
self::lazy_static!{} // 基于路径的查找忽略我们的宏，找到导入的那个。
```

r[macro.decl.scope.textual]
### 文本作用域

r[macro.decl.scope.textual.intro]
文本作用域很大程度上基于事物在源文件中出现的顺序，其工作方式类似于用 `let` 声明的局部变量的作用域，只是它也适用于模块级别。当使用 `macro_rules!` 定义宏时，宏在定义之后进入作用域（注意，它仍然可以递归使用，因为名称是从调用位置查找的），直到其外围作用域（通常是一个模块）关闭。这可以进入子模块，甚至跨越多个文件：

<!-- ignore: requires external modules -->
```rust,ignore
//// src/lib.rs
mod has_macro {
    // m!{} // 错误：m 不在作用域中。

    macro_rules! m {
        () => {};
    }
    m!{} // OK：在声明 m 之后出现。

    mod uses_macro;
}

// m!{} // 错误：m 不在作用域中。

//// src/has_macro/uses_macro.rs

m!{} // OK：在 src/lib.rs 中声明 m 之后出现
```

r[macro.decl.scope.textual.shadow]
多次定义宏不是错误；最近的声明将遮蔽先前的声明，除非它已经离开作用域。

```rust
macro_rules! m {
    (1) => {};
}

m!(1);

mod inner {
    m!(1);

    macro_rules! m {
        (2) => {};
    }
    // m!(1); // 错误：没有规则匹配 '1'
    m!(2);

    macro_rules! m {
        (3) => {};
    }
    m!(3);
}

m!(1);
```

宏也可以在函数内部局部声明和使用，其工作方式类似：

```rust
fn foo() {
    // m!(); // 错误：m 不在作用域中。
    macro_rules! m {
        () => {};
    }
    m!();
}

// m!(); // 错误：m 不在作用域中。
```

r[macro.decl.scope.textual.shadow.path-based]
文本作用域的名称绑定对宏的基于路径作用域绑定具有遮蔽效果。

```rust
macro_rules! m2 {
    () => {
        println!("m2");
    };
}

// 解析到下面 use 声明带来的基于路径的候选项。
m!(); // 打印 "m2\n"

// 用文本作用域引入第二个 `m` 候选项。
//
// 从这之后，这会遮蔽下面那个基于路径的候选项。
macro_rules! m {
    () => {
        println!("m");
    };
}

// 将 `m2` 宏作为基于路径的候选项引入。
//
// 这项在整个示例中都在作用域中，不仅仅是在 use 声明之后。
use m2 as m;

// 解析到 use 声明上方的文本宏候选项。
m!(); // 打印 "m\n"
```

> [!NOTE]
> 关于不允许遮蔽的区域，参见[名称解析歧义][name resolution ambiguities]。

r[macro.decl.scope.path-based]
### 基于路径的作用域

r[macro.decl.scope.path-based.intro]
默认情况下，宏没有基于路径的作用域。宏可以通过两种方式获得基于路径的作用域：

- [use 声明重导出][Use declaration re-export]
- [`macro_export`]

r[macro.decl.scope.path.reexport]
宏可以被重导出，以使其从 crate 根以外的模块获得基于路径的作用域。

```rust
mac::m!(); // OK：基于路径的查找在 mac 模块中找到 `m`。

mod mac {
    // 用文本作用域引入宏 `m`。
    macro_rules! m {
        () => {};
    }

    // 从 `m` 的文本作用域内进行基于路径作用域的重导出。
    pub(crate) use m;
}
```

r[macro.decl.scope.path-based.visibility]
宏具有隐式的 `pub(crate)` 可见性。`#[macro_export]` 将隐式可见性更改为 `pub`。

```rust
// 隐式可见性是 `pub(crate)`。
macro_rules! private_m {
    () => {};
}

// 隐式可见性是 `pub`。
#[macro_export]
macro_rules! pub_m {
    () => {};
}

pub(crate) use private_m as private_macro; // OK。
pub use pub_m as pub_macro; // OK。
```

```rust,compile_fail,E0364
# // 隐式可见性是 `pub(crate)`。
# macro_rules! private_m {
#     () => {};
# }
#
# // 隐式可见性是 `pub`。
# #[macro_export]
# macro_rules! pub_m {
#     () => {};
# }
#
# pub(crate) use private_m as private_macro; // OK。
# pub use pub_m as pub_macro; // OK。
#
pub use private_m; // 错误：`private_m` 仅在该 crate 内公开，
                   // 不能在外部重导出。
```

<!-- template:attributes -->
r[macro.decl.scope.macro_use]
### `macro_use` 属性 {#the-macro_use-attribute}

r[macro.decl.scope.macro_use.intro]
*`macro_use` [属性][attributes]* 有两个用途：它可以用于模块以扩展其中定义的宏的作用域，也可以用于 [`extern crate`][items.extern-crate] 将另一个 crate 的宏导入 [`macro_use` 预导入][`macro_use` prelude]中。

> [!EXAMPLE]
> ```rust
> #[macro_use]
> mod inner {
>     macro_rules! m {
>         () => {};
>     }
> }
> m!();
> ```
>
> ```rust,ignore
> #[macro_use]
> extern crate log;
> ```

r[macro.decl.scope.macro_use.syntax]
当用于模块时，`macro_use` 属性使用 [MetaWord] 语法。

当用于 `extern crate` 时，它使用 [MetaWord] 和 [MetaListIdents] 语法。关于如何使用这些语法，请参见 [macro.decl.scope.macro_use.prelude]。

r[macro.decl.scope.macro_use.allowed-positions]
`macro_use` 属性可以应用于模块或 `extern crate`。

> [!NOTE]
> `rustc` 忽略在其他位置的使用，但会发出 lint。这未来可能成为一个错误。

r[macro.decl.scope.macro_use.extern-crate-self]
`macro_use` 属性不能用于 [`extern crate self`]。

r[macro.decl.scope.macro_use.duplicates]
`macro_use` 属性可以在一项形式体上使用任意多次。

在 [MetaListIdents] 语法中可以指定多个 `macro_use` 实例。所有指定宏的并集将被导入。

> [!NOTE]
> 在模块上，`rustc` 会对第一个之后的任何 [MetaWord] `macro_use` 属性发出 lint。
>
> 在 `extern crate` 上，`rustc` 会对任何由于没有导入任何未被其他 `macro_use` 属性导入的宏而无效的 `macro_use` 属性发出 lint。如果两个或多个 [MetaListIdents] `macro_use` 属性导入了同一个宏，则会对第一个发出 lint。如果存在任何 [MetaWord] `macro_use` 属性，则会对所有 [MetaListIdents] `macro_use` 属性发出 lint。如果存在两个或多个 [MetaWord] `macro_use` 属性，则会对第一个之后的那些发出 lint。

r[macro.decl.scope.macro_use.mod-decl]
当 `macro_use` 用于模块时，该模块的宏作用域将扩展到该模块的词法作用域之外。

> [!EXAMPLE]
> ```rust
> #[macro_use]
> mod inner {
>     macro_rules! m {
>         () => {};
>     }
> }
> m!(); // OK
> ```

r[macro.decl.scope.macro_use.prelude]
在 crate 根中对 `extern crate` 声明指定 `macro_use` 会从该 crate 导入已导出的宏。

以这种方式导入的宏被导入到 [`macro_use` 预导入][`macro_use` prelude]中，而非文本方式，这意味着它们可以被任何其他名称遮蔽。通过 `macro_use` 导入的宏可以在导入语句之前使用。

> [!NOTE]
> `rustc` 当前在有冲突的情况下倾向于最后导入的宏。不要依赖这一点。这种行为是不寻常的，因为 Rust 中的导入通常是无顺序依赖的。`macro_use` 的这种行为未来可能会改变。
>
> 详情请参阅 [Rust issue #148025](https://github.com/rust-lang/rust/issues/148025)。

当使用 [MetaWord] 语法时，所有导出的宏都被导入。当使用 [MetaListIdents] 语法时，仅导入指定的宏。

> [!EXAMPLE]
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[macro_use(lazy_static)] // 或者 `#[macro_use]` 来导入所有宏。
> extern crate lazy_static;
>
> lazy_static!{}
> // self::lazy_static!{} // 错误：lazy_static 在 `self` 中未定义。
> ```

r[macro.decl.scope.macro_use.export]
要通过 `macro_use` 导入的宏必须使用 [`macro_export`][macro.decl.scope.macro_export] 导出。

<!-- template:attributes -->
r[macro.decl.scope.macro_export]
### `macro_export` 属性 {#the-macro_export-attribute}

r[macro.decl.scope.macro_export.intro]
*`macro_export` [属性][attributes]* 从 crate 中导出宏，并使其在 crate 根中可用于基于路径的解析。

> [!EXAMPLE]
> ```rust
> self::m!();
> //  ^^^^ OK：基于路径的查找在当前模块中找到 `m`。
> m!(); // 同上。
>
> mod inner {
>     super::m!();
>     crate::m!();
> }
>
> mod mac {
>     #[macro_export]
>     macro_rules! m {
>         () => {};
>     }
> }
> ```

r[macro.decl.scope.macro_export.syntax]
`macro_export` 属性使用 [MetaWord] 和 [MetaListIdents] 语法。使用 [MetaListIdents] 语法时，它接受一个 [`local_inner_macros`][macro.decl.scope.macro_export.local_inner_macros] 值。

r[macro.decl.scope.macro_export.allowed-positions]
`macro_export` 属性可以应用于 `macro_rules` 定义。

> [!NOTE]
> `rustc` 忽略在其他位置的使用，但会发出 lint。这未来可能成为一个错误。

r[macro.decl.scope.macro_export.duplicates]
只有第一次对宏使用 `macro_export` 才会生效。

> [!NOTE]
> `rustc` 会对第一次之后的任何使用发出 lint。

r[macro.decl.scope.macro_export.path-based]
默认情况下，宏只有[文本作用域][macro.decl.scope.textual]，不能通过路径解析。当使用 `macro_export` 属性时，宏在 crate 根中变得可用，可以通过其路径引用。

> [!EXAMPLE]
> 没有 `macro_export`，宏只有文本作用域，因此对宏的基于路径的解析会失败。
>
> ```rust,compile_fail,E0433
> macro_rules! m {
>     () => {};
> }
> self::m!(); // 错误
> crate::m!(); // 错误
> # fn main() {}
> ```
>
> 使用 `macro_export`，基于路径的解析可以正常工作。
>
> ```rust
> #[macro_export]
> macro_rules! m {
>     () => {};
> }
> self::m!(); // OK
> crate::m!(); // OK
> # fn main() {}
> ```

r[macro.decl.scope.macro_export.export]
`macro_export` 属性使宏从 crate 根导出，以便可以在其他 crate 中通过路径引用。

> [!EXAMPLE]
> 假设在 `log` crate 中有以下定义：
>
> ```rust
> #[macro_export]
> macro_rules! warn {
>     ($message:expr) => { eprintln!("WARN: {}", $message) };
> }
> ```
>
> 从另一个 crate，你可以通过路径引用该宏：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> fn main() {
>     log::warn!("example warning");
> }
> ```

r[macro.decl.scope.macro_export.macro_use]
`macro_export` 允许在 `extern crate` 上使用 [`macro_use`][macro.decl.scope.macro_use] 将宏导入到 [`macro_use` 预导入][`macro_use` prelude]中。

> [!EXAMPLE]
> 假设在 `log` crate 中有以下定义：
>
> ```rust
> #[macro_export]
> macro_rules! warn {
>     ($message:expr) => { eprintln!("WARN: {}", $message) };
> }
> ```
>
> 在依赖 crate 中使用 `macro_use` 允许你从预导入使用该宏：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[macro_use]
> extern crate log;
>
> pub mod util {
>     pub fn do_thing() {
>         // 通过宏预导入解析。
>         warn!("example warning");
>     }
> }
> ```

r[macro.decl.scope.macro_export.local_inner_macros]
在 `macro_export` 属性中添加 `local_inner_macros` 会使宏定义中所有单段宏调用隐式地带有 `$crate::` 前缀。

> [!NOTE]
> 这主要是作为一种工具，用于将 [`$crate`] 添加到语言之前编写的代码迁移到能与 Rust 2018 的基于路径的宏导入一起工作。不建议在新代码中使用。

> [!EXAMPLE]
> ```rust
> #[macro_export(local_inner_macros)]
> macro_rules! helped {
>     () => { helper!() } // 自动转换为 $crate::helper!()。
> }
>
> #[macro_export]
> macro_rules! helper {
>     () => { () }
> }
> ```

r[macro.decl.hygiene]
## 卫生性

r[macro.decl.hygiene.intro]
声明宏具有*混合位置卫生性*（mixed-site hygiene）。这意味着[循环标签][loop labels]、[块标签][block labels]和局部变量在宏定义位置查找，而其他符号在宏调用位置查找。例如：

```rust
let x = 1;
fn func() {
    unreachable!("this is never called")
}

macro_rules! check {
    () => {
        assert_eq!(x, 1); // 使用定义位置的 `x`。
        func();           // 使用调用位置的 `func`。
    };
}

{
    let x = 2;
    fn func() { /* 不会 panic */ }
    check!();
}
```

在宏展开中定义的标签和局部变量不在调用之间共享，因此以下代码无法编译：

```rust,compile_fail,E0425
macro_rules! m {
    (define) => {
        let x = 1;
    };
    (refer) => {
        dbg!(x);
    };
}

m!(define);
m!(refer);
```

r[macro.decl.hygiene.crate]
一个特殊情况是 `$crate` 元变量。它引用定义该宏的 crate，并且可以在路径开头使用，以查找在调用位置不在作用域内的项或宏。

<!-- ignore: requires external crates -->
```rust,ignore
//// 在 `helper_macro` crate 中的定义。
#[macro_export]
macro_rules! helped {
    // () => { helper!() } // 由于 'helper' 不在作用域内，这可能导致错误。
    () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
    () => { () }
}

//// 在另一个 crate 中使用。
// 注意 `helper_macro::helper` 没有被导入！
use helper_macro::helped;

fn unit() {
    helped!();
}
```

注意，因为 `$crate` 引用当前 crate，所以在引用非宏项时，必须使用完全限定的模块路径：

```rust
pub mod inner {
    #[macro_export]
    macro_rules! call_foo {
        () => { $crate::inner::foo() };
    }

    pub fn foo() {}
}
```

r[macro.decl.hygiene.vis]
此外，尽管 `$crate` 允许宏在展开时引用其自身 crate 内的项，但它的使用对可见性没有影响。引用的项或宏仍然必须从调用位置可见。在以下示例中，从 crate 外部调用 `call_foo!()` 的任何尝试都将失败，因为 `foo()` 不是公开的。

```rust
#[macro_export]
macro_rules! call_foo {
    () => { $crate::foo() };
}

fn foo() {}
```

> [!NOTE]
> 在 Rust 1.30 之前，`$crate` 和 [`local_inner_macros`][macro.decl.scope.macro_export.local_inner_macros] 不被支持。它们是与[基于路径的宏导入][macro.decl.scope.macro_export]一起添加的，以确保辅助宏不需要由宏导出 crate 的用户手动导入。为较早版本 Rust 编写的、使用辅助宏的 crate 需要修改为使用 `$crate` 或 `local_inner_macros` 才能与基于路径的导入良好配合。

r[macro.decl.follow-set]
## 后继集合歧义限制

r[macro.decl.follow-set.intro]
宏系统使用的解析器相当强大，但为了防止当前或未来版本语言中的歧义，它受到限制。

r[macro.decl.follow-set.token-restriction]
特别是，除了关于歧义展开的规则之外，由元变量匹配的非终结符之后必须跟一个已被确定为可以安全地在该类型匹配之后使用的 token。

举例来说，像 `$i:expr [ , ]` 这样的宏匹配器在今天的 Rust 中理论上可以被接受，因为 `[,]` 不可能是合法表达式的一部分，因此解析总是明确的。然而，因为 `[` 可以开始尾随表达式，`[` 不是一个能够安全排除在表达式之后出现的字符。如果 `[,]` 在 Rust 的后续版本中被接受，则该匹配器将变得歧义或解析错误，破坏正常工作的代码。然而，像 `$i:expr,` 或 `$i:expr;` 这样的匹配器是合法的，因为 `,` 和 `;` 是合法的表达式分隔符。具体规则是：

r[macro.decl.follow-set.token-expr-stmt]
  * `expr` 和 `stmt` 之后只能跟以下之一：`=>`、`,` 或 `;`。

r[macro.decl.follow-set.token-pat_param]
  * `pat_param` 之后只能跟以下之一：`=>`、`,`、`=`、`|`、`if` 或 `in`。

r[macro.decl.follow-set.token-pat]
  * `pat` 之后只能跟以下之一：`=>`、`,`、`=`、`if` 或 `in`。

r[macro.decl.follow-set.token-path-ty]
  * `path` 和 `ty` 之后只能跟以下之一：`=>`、`,`、`=`、`|`、`;`、`:`、`>`、`>>`、`[`、`{`、`as`、`where`，或一个 `block` 片段说明符的宏变量。

r[macro.decl.follow-set.token-vis]
  * `vis` 之后只能跟以下之一：`,`，一个非原生 `priv` 之外的标识符，任何可以开始类型的 token，或一个具有 `ident`、`ty` 或 `path` 片段说明符的元变量。

r[macro.decl.follow-set.token-other]
  * 所有其他片段说明符没有任何限制。

r[macro.decl.follow-set.edition2021]
> [!EDITION-2021]
> 在 2021 版之前，`pat` 之后也可以跟 `|`。

r[macro.decl.follow-set.repetition]
当涉及重复时，规则适用于每种可能的展开次数，并考虑分隔符。这意味着：

  * 如果重复包含分隔符，则该分隔符必须能够跟在重复内容之后。
  * 如果重复可以重复多次（`*` 或 `+`），则内容必须能够跟在自身之后。
  * 重复的内容必须能够跟在其之前的任何内容之后，而其之后的任何内容必须能够跟在重复内容之后。
  * 如果重复可以匹配零次（`*` 或 `?`），则之后的内容必须能够跟在之前的内容之后。

更多细节请参见[形式化规范][formal specification]。

[Metavariables]: #metavariables
[Repetitions]: #repetitions
[`macro_export`]: #the-macro_export-attribute
[`$crate`]: macro.decl.hygiene.crate
[`extern crate self`]: items.extern-crate.self
[`macro_use` prelude]: names/preludes.md#macro_use-prelude
[block labels]: expr.loop.block-labels
[delimiters]: tokens.md#delimiters
[formal specification]: macro-ambiguity.md
[loop labels]: expressions/loop-expr.md#loop-labels
[name resolution ambiguities]: names/name-resolution.md#r-names.resolution.expansion.imports.ambiguity
[token]: tokens.md
[use declaration re-export]: items/use-declarations.md#use-visibility
