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
      Token _排除 `$` 和 [delimiters][lex.token.delim]_
    | MacroMatcher
    | `$` ( IDENTIFIER_OR_KEYWORD _排除 `crate`_ | RAW_IDENTIFIER ) `:` MacroFragSpec
    | `$` `(` MacroMatch+ `)` MacroRepSep? MacroRepOp

MacroFragSpec ->
      `block` | `expr` | `expr_2021` | `ident` | `item` | `lifetime` | `literal`
    | `meta` | `pat` | `pat_param` | `path` | `stmt` | `tt` | `ty` | `vis`

MacroRepSep -> Token _排除 [delimiters][lex.token.delim] 和 [MacroRepOp]_

MacroRepOp -> `*` | `+` | `?`

MacroTranscriber -> DelimTokenTree
```

r[macro.decl.intro]
`macro_rules` 允许用户以声明的方式定义语法扩展。我们称这种扩展为"声明宏"或简称为"宏"。

每个声明宏都有一个名称，以及一个或多个*规则*。每条规则有两个部分：一个*匹配器*，描述它匹配的语法；一个*转录器*，描述将替换成功匹配的调用的语法。匹配器和转录器都必须由定界符括起来。宏可以展开为表达式、语句、项（包括 trait、impl 和外部项）、类型或模式。

r[macro.decl.transcription]
## 转录

r[macro.decl.transcription.intro]
当宏被调用时，宏展开器按名称查找宏调用，并依次尝试每条宏规则。它转录第一个成功匹配的规则；如果这导致错误，则不会尝试后续匹配。

r[macro.decl.transcription.lookahead]
在匹配时，不执行前向查看（lookahead）；如果编译器无法一次一个词法单元地明确确定如何解析宏调用，则会产生错误。在以下示例中，编译器不会在标识符之后向前查看以判断下一个词法单元是否是 `)`，即使这能让它无歧义地解析调用：

```rust,compile_fail
macro_rules! ambiguity {
    ($($i:ident)* $j:ident) => { };
}

ambiguity!(error); // 错误：局部歧义
```

r[macro.decl.transcription.syntax]
在匹配器和转录器中，`$` 词法单元用于调用宏引擎的特殊行为（下文在[元变量][Metavariables]和[重复][Repetitions]中描述）。不属于此类调用的词法单元按字面匹配和转录，但有一个例外。例外是匹配器的外部定界符将匹配任意一对定界符。因此，例如，匹配器 `(())` 将匹配 `{()}` 但不匹配 `{{}}`。字符 `$` 不能按字面匹配或转录。

r[macro.decl.transcription.fragment]
### 转发匹配到的片段

当将匹配到的片段转发给另一个声明宏时，第二个宏中的匹配器将看到该片段类型的不透明 AST。第二个宏不能使用字面词法单元来匹配匹配器中的片段，只能使用相同类型的片段限定符。`ident`、`lifetime` 和 `tt` 片段类型是例外，它们*可以*通过字面词法单元匹配。以下示例说明了这一限制：

```rust,compile_fail
macro_rules! foo {
    ($l:expr) => { bar!($l); }
// 错误：                    ^^ 宏调用中没有预期此词法单元的规则
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

以下示例展示了在匹配 `tt` 片段后如何直接匹配词法单元：

```rust
// 编译通过
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
在匹配器中，`$` *名称* `:` *片段限定符* 匹配指定类型的 Rust 语法片段，并将其绑定到元变量 `$`*名称*。

r[macro.decl.meta.specifier]
有效的片段限定符有：

  * `block`：一个[块表达式][BlockExpression]
  * `expr`：一个[表达式][Expression]
  * `expr_2021`：一个[表达式][Expression]，但排除[下划线表达式][UnderscoreExpression]和[常量块表达式][ConstBlockExpression]（参见 [macro.decl.meta.edition2024]）
  * `ident`：一个除 `_`、[RAW_IDENTIFIER] 或 [`$crate`] 之外的 [IDENTIFIER_OR_KEYWORD]
  * `item`：一个[项][Item]
  * `lifetime`：一个 [LIFETIME_TOKEN]
  * `literal`：匹配 `-`<sup>?</sup>[字面量表达式][LiteralExpression]
  * `meta`：一个 [Attr]，即属性的内容
  * `pat`：一个[模式][Pattern]（参见 [macro.decl.meta.edition2021]）
  * `pat_param`：一个 [PatternNoTopAlt]
  * `path`：一个 [TypePath]
  * `stmt`：一个[语句][grammar-Statement]，不带尾随分号（需要分号的项语句除外）
  * `tt`：一个 [TokenTree]（单个[词法单元][token]或匹配定界符 `()`、`[]` 或 `{}` 中的词法单元）
  * `ty`：一个[类型][grammar-Type]
  * `vis`：一个可能为空的[可见性][Visibility]限定符

r[macro.decl.meta.transcription]
在转录器中，元变量仅通过 `$`*名称* 来引用，因为片段类型已在匹配器中指定。元变量被替换为与之匹配的语法元素。元变量可以被转录多次或完全不转录。

r[macro.decl.meta.dollar-crate]
关键字元变量 [`$crate`] 可用于引用当前 crate。

r[macro.decl.meta.edition2021]
> [!EDITION-2021]
> 从 2021 版本开始，`pat` 片段限定符匹配顶层或模式（即，它们接受 [Pattern]）。
>
> 在 2021 版本之前，它们匹配的片段与 `pat_param` 完全相同（即，它们接受 [PatternNoTopAlt]）。
>
> 相关版本是 `macro_rules!` 定义生效时的版本。

r[macro.decl.meta.edition2024]
> [!EDITION-2024]
> 在 2024 版本之前，`expr` 片段限定符在顶层不匹配[下划线表达式][UnderscoreExpression]或[常量块表达式][ConstBlockExpression]。它们允许在子表达式中出现。
>
> `expr_2021` 片段限定符的存在是为了维持与 2024 之前版本的向后兼容性。

r[macro.decl.repetition]
## 重复 {#repetitions}

r[macro.decl.repetition.intro]
在匹配器和转录器中，重复通过将要重复的词法单元放在 `$(`…`)` 内，后跟一个重复运算符来表示，可选地在中间放置一个分隔符词法单元。

r[macro.decl.repetition.separator]
分隔符词法单元可以是除定界符或重复运算符之外的任何词法单元，但 `;` 和 `,` 是最常见的。例如，`$( $i:ident ),*` 表示任意数量的由逗号分隔的标识符。允许嵌套重复。

r[macro.decl.repetition.operators]
重复运算符有：

- `*` --- 表示任意数量的重复。
- `+` --- 表示任意数量但至少一次。
- `?` --- 表示可选的片段，出现零次或一次。

r[macro.decl.repetition.optional-restriction]
由于 `?` 表示最多一次出现，因此不能与分隔符一起使用。

r[macro.decl.repetition.fragment]
重复的片段既匹配也转录为指定数量的片段，由分隔符词法单元分隔。元变量会匹配其对应片段的每一次重复。例如，上例中的 `$( $i:ident ),*` 将 `$i` 匹配到列表中的所有标识符。

在转录过程中，对重复施加了额外限制，以便编译器知道如何正确展开它们：

1.  元变量在转录器中必须以与匹配器中完全相同的次数、类型和嵌套顺序出现在重复中。因此，对于匹配器 `$( $i:ident ),*`，转录器 `=> { $i }`、`=> { $( $( $i )* )* }` 和 `=> { $( $i )+ }` 都是非法的，但 `=> { $( $i );* }` 是正确的，它将逗号分隔的标识符列表替换为分号分隔的列表。
2.  转录器中的每个重复必须包含至少一个元变量，以决定将其展开多少次。如果同一重复中出现多个元变量，它们必须绑定到相同数量的片段。例如，`( $( $i:ident ),* ; $( $j:ident ),* ) => (( $( ($i,$j) ),* ))` 必须将相同数量的 `$i` 片段绑定到 `$j` 片段。这意味着使用 `(a, b, c; d, e, f)` 调用宏是合法的，并展开为 `((a,d), (b,e), (c,f))`，但 `(a, b, c; d, e)` 是非法的，因为它数量不相等。此要求适用于每一层嵌套重复。

r[macro.decl.scope]
## 作用域、导出和导入

r[macro.decl.scope.intro]
由于历史原因，声明宏的作用域不完全像项那样工作。宏有两种形式的作用域：文本作用域和基于路径的作用域。文本作用域基于源文件甚至跨多个文件中各项出现的顺序，是默认的作用域方式。下文将进一步解释。基于路径的作用域的工作方式与项作用域完全相同。宏的作用域、导出和导入主要由属性控制。

r[macro.decl.scope.unqualified]
当通过非限定标识符（不是多段路径的一部分）调用宏时，首先在文本作用域中查找。如果没有找到任何结果，则在基于路径的作用域中查找。如果宏的名称通过路径限定，则只在基于路径的作用域中查找。

<!-- ignore: requires external crates -->
```rust,ignore
use lazy_static::lazy_static; // 基于路径的导入。

macro_rules! lazy_static { // 文本定义。
    (lazy) => {};
}

lazy_static!{lazy} // 文本查找首先找到我们的宏。
self::lazy_static!{} // 基于路径的查找忽略我们的宏，找到导入的宏。
```

r[macro.decl.scope.textual]
### 文本作用域

r[macro.decl.scope.textual.intro]
文本作用域主要基于源文件中各项出现的顺序，其工作方式类似于用 `let` 声明的局部变量的作用域，但同时也适用于模块级别。当使用 `macro_rules!` 定义宏时，该宏在定义之后进入作用域（注意，它仍然可以递归使用，因为名称是从调用点查找的），直到其外围作用域（通常是模块）关闭。这可以进入子模块，甚至可以跨多个文件：

<!-- ignore: requires external modules -->
```rust,ignore
//// src/lib.rs
mod has_macro {
    // m!{} // 错误：m 不在作用域内。

    macro_rules! m {
        () => {};
    }
    m!{} // OK：出现在 m 声明之后。

    mod uses_macro;
}

// m!{} // 错误：m 不在作用域内。

//// src/has_macro/uses_macro.rs

m!{} // OK：出现在 src/lib.rs 中 m 声明之后
```

r[macro.decl.scope.textual.shadow]
多次定义同一个宏不是错误；最近的声明将遮蔽先前的声明，除非先前的声明已离开作用域。

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

宏也可以在函数内部声明和局部使用，其工作方式类似：

```rust
fn foo() {
    // m!(); // 错误：m 不在作用域内。
    macro_rules! m {
        () => {};
    }
    m!();
}

// m!(); // 错误：m 不在作用域内。
```

r[macro.decl.scope.textual.shadow.path-based]
宏的文本作用域名称绑定会遮蔽基于路径的作用域绑定到宏。

```rust
macro_rules! m2 {
    () => {
        println!("m2");
    };
}

// 解析为来自下方 use 声明的基于路径的候选项。
m!(); // 打印 "m2\n"

// 通过文本作用域为 `m` 引入第二个候选项。
//
// 这会遮蔽下方基于路径的候选项，覆盖本示例的剩余部分。
macro_rules! m {
    () => {
        println!("m");
    };
}

// 将 `m2` 宏作为基于路径的候选项引入。
//
// 此项在整个示例中都在作用域内，而不仅仅在 use
// 声明之后。
use m2 as m;

// 解析为来自 use 声明上方的文本宏候选项。
m!(); // 打印 "m\n"
```

> [!NOTE]
> 关于不允许遮蔽的情况，请参见[名称解析歧义][name resolution ambiguities]。

r[macro.decl.scope.path-based]
### 基于路径的作用域

r[macro.decl.scope.path-based.intro]
默认情况下，宏没有基于路径的作用域。宏可以通过两种方式获得基于路径的作用域：

- [使用声明重导出][use declaration re-export]
- [`macro_export`]

r[macro.decl.scope.path.reexport]
宏可以被重导出，使其从 crate 根之外的模块获得基于路径的作用域。

```rust
mac::m!(); // OK：基于路径的查找在 mac 模块中找到 `m`。

mod mac {
    // 通过文本作用域引入宏 `m`。
    macro_rules! m {
        () => {};
    }

    // 从 `m` 的文本作用域内以基于路径的作用域重导出。
    pub(crate) use m;
}
```

r[macro.decl.scope.path-based.visibility]
宏具有隐式的 `pub(crate)` 可见性。`#[macro_export]` 将隐式可见性更改为 `pub`。

```rust
// 隐式可见性为 `pub(crate)`。
macro_rules! private_m {
    () => {};
}

// 隐式可见性为 `pub`。
#[macro_export]
macro_rules! pub_m {
    () => {};
}

pub(crate) use private_m as private_macro; // OK。
pub use pub_m as pub_macro; // OK。
```

```rust,compile_fail,E0364
# // 隐式可见性为 `pub(crate)`。
# macro_rules! private_m {
#     () => {};
# }
#
# // 隐式可见性为 `pub`。
# #[macro_export]
# macro_rules! pub_m {
#     () => {};
# }
#
# pub(crate) use private_m as private_macro; // OK。
# pub use pub_m as pub_macro; // OK。
#
pub use private_m; // 错误：`private_m` 仅在 crate 内公开，
                   // 不能在外部重导出。
```

<!-- template:attributes -->
r[macro.decl.scope.macro_use]
### `macro_use` 属性

r[macro.decl.scope.macro_use.intro]
*`macro_use` [属性][attributes]* 有两个用途：可以用在模块上以扩展其中定义的宏的作用域，也可以用在 [`extern crate`][items.extern-crate] 上以将其他 crate 的宏导入到 [`macro_use` 预导入][`macro_use` prelude]中。

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
在模块上使用时，`macro_use` 属性使用 [MetaWord] 语法。

在 `extern crate` 上使用时，它使用 [MetaWord] 和 [MetaListIdents] 语法。关于如何使用这些语法，请参见 [macro.decl.scope.macro_use.prelude]。

r[macro.decl.scope.macro_use.allowed-positions]
`macro_use` 属性可以应用于模块或 `extern crate`。

> [!NOTE]
> `rustc` 忽略其他位置的使用，但会对其进行 lint 警告。这在将来可能成为错误。

r[macro.decl.scope.macro_use.extern-crate-self]
`macro_use` 属性不能用于 [`extern crate self`]。

r[macro.decl.scope.macro_use.duplicates]
`macro_use` 属性可以在同一个形式上使用任意次数。

可以指定多个 [MetaListIdents] 语法形式的 `macro_use` 实例。将导入所有指定宏的并集。

> [!NOTE]
> 在模块上，`rustc` 会针对第一个之后的任何 [MetaWord] `macro_use` 属性进行 lint 警告。
>
> 在 `extern crate` 上，`rustc` 会针对由于未导入任何尚未由其他 `macro_use` 属性导入的宏而没有效果的 `macro_use` 属性进行 lint 警告。如果两个或更多 [MetaListIdents] `macro_use` 属性导入同一个宏，则对第一个进行 lint 警告。如果存在任何 [MetaWord] `macro_use` 属性，则对所有 [MetaListIdents] `macro_use` 属性进行 lint 警告。如果存在两个或更多 [MetaWord] `macro_use` 属性，则对第一个之后的那些进行 lint 警告。

r[macro.decl.scope.macro_use.mod-decl]
当 `macro_use` 用在模块上时，该模块的宏作用域会扩展到模块的词法作用域之外。

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
在 crate 根中的 `extern crate` 声明上指定 `macro_use` 会导入该 crate 的导出宏。

通过这种方式导入的宏被导入到 [`macro_use` 预导入][`macro_use` prelude]中，而不是文本作用域中，这意味着它们可以被任何其他名称遮蔽。通过 `macro_use` 导入的宏可以在导入语句之前使用。

> [!NOTE]
> `rustc` 目前在冲突的情况下优先选择最后导入的宏。不要依赖这一点。这种行为是不寻常的，因为 Rust 中的导入通常是顺序无关的。`macro_use` 的这种行为将来可能会改变。
>
> 详情请参见 [Rust 问题 #148025](https://github.com/rust-lang/rust/issues/148025)。

使用 [MetaWord] 语法时，所有导出的宏都会被导入。使用 [MetaListIdents] 语法时，只导入指定的宏。

> [!EXAMPLE]
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[macro_use(lazy_static)] // 或 `#[macro_use]` 以导入所有宏。
> extern crate lazy_static;
>
> lazy_static!{}
> // self::lazy_static!{} // 错误：lazy_static 未在 `self` 中定义。
> ```

r[macro.decl.scope.macro_use.export]
要通过 `macro_use` 导入的宏必须使用 [`macro_export`][macro.decl.scope.macro_export] 导出。

<!-- template:attributes -->
r[macro.decl.scope.macro_export]
### `macro_export` 属性

r[macro.decl.scope.macro_export.intro]
*`macro_export` [属性][attributes]* 从 crate 导出宏，并使其在 crate 的根中可用于基于路径的解析。

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
> `rustc` 忽略其他位置的使用，但会对其进行 lint 警告。这在将来可能成为错误。

r[macro.decl.scope.macro_export.duplicates]
只有宏上的第一次 `macro_export` 使用才有效。

> [!NOTE]
> `rustc` 会针对第一次之后的任何使用进行 lint 警告。

r[macro.decl.scope.macro_export.path-based]
默认情况下，宏只有[文本作用域][macro.decl.scope.textual]，不能通过路径解析。当使用 `macro_export` 属性时，宏在 crate 根中可用，可以通过其路径引用。

> [!EXAMPLE]
> 没有 `macro_export` 时，宏只有文本作用域，因此基于路径的宏解析会失败。
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
> 使用 `macro_export` 时，基于路径的解析可以工作。
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
> 假设在 `log` crate 中有以下代码：
>
> ```rust
> #[macro_export]
> macro_rules! warn {
>     ($message:expr) => { eprintln!("WARN: {}", $message) };
> }
> ```
>
> 从另一个 crate 中，可以通过路径引用该宏：
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
> 假设在 `log` crate 中有以下代码：
>
> ```rust
> #[macro_export]
> macro_rules! warn {
>     ($message:expr) => { eprintln!("WARN: {}", $message) };
> }
> ```
>
> 在依赖 crate 中使用 `macro_use` 允许你从预导入中使用该宏：
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
在 `macro_export` 属性中添加 `local_inner_macros` 会使宏定义中所有单段宏调用隐式地添加 `$crate::` 前缀。

> [!NOTE]
> 这主要是一个工具，用于将 [`$crate`] 添加到语言之前编写的代码迁移到与 Rust 2018 的基于路径的宏导入兼容。不建议在新代码中使用。

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
声明宏具有*混合站点卫生性*（mixed-site hygiene）。这意味着[循环标签][loop labels]、[块标签][block labels]和局部变量在宏定义站点查找，而其他符号在宏调用站点查找。例如：

```rust
let x = 1;
fn func() {
    unreachable!("this is never called")
}

macro_rules! check {
    () => {
        assert_eq!(x, 1); // 使用定义站点的 `x`。
        func();           // 使用调用站点的 `func`。
    };
}

{
    let x = 2;
    fn func() { /* 不会 panic */ }
    check!();
}
```

宏展开中定义的标签和局部变量不会在调用之间共享，因此以下代码无法编译：

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
一个特例是 `$crate` 元变量。它引用定义该宏的 crate，可以用于路径的开头来查找在调用站点不在作用域内的项或宏。

<!-- ignore: requires external crates -->
```rust,ignore
//// `helper_macro` crate 中的定义。
#[macro_export]
macro_rules! helped {
    // () => { helper!() } // 这可能导致错误，因为 'helper' 不在作用域内。
    () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
    () => { () }
}

//// 另一个 crate 中的使用。
// 注意 `helper_macro::helper` 没有被导入！
use helper_macro::helped;

fn unit() {
    helped!();
}
```

注意，因为 `$crate` 引用当前 crate，在引用非宏项时必须使用完全限定的模块路径：

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
此外，尽管 `$crate` 允许宏在展开时引用其自身 crate 中的项，但它的使用对可见性没有影响。被引用的项或宏仍然必须从调用站点可见。在以下示例中，任何从 crate 外部调用 `call_foo!()` 的尝试都将失败，因为 `foo()` 不是公开的。

```rust
#[macro_export]
macro_rules! call_foo {
    () => { $crate::foo() };
}

fn foo() {}
```

> [!NOTE]
> 在 Rust 1.30 之前，不支持 `$crate` 和 [`local_inner_macros`][macro.decl.scope.macro_export.local_inner_macros]。它们与[基于路径的宏导入][macro.decl.scope.macro_export]一起添加，以确保辅助宏不需要由宏导出 crate 的用户手动导入。为较早版本 Rust 编写的使用辅助宏的 crate 需要修改为使用 `$crate` 或 `local_inner_macros` 才能与基于路径的导入良好配合。

r[macro.decl.follow-set]
## 后继集歧义限制

r[macro.decl.follow-set.intro]
宏系统使用的解析器相当强大，但为了防止在当前或未来语言版本中产生歧义，它受到了限制。

r[macro.decl.follow-set.token-restriction]
特别地，除了关于歧义展开的规则之外，由元变量匹配的非终结符后面必须跟一个被认为可以在该类型匹配之后安全使用的词法单元。

例如，像 `$i:expr [ , ]` 这样的宏匹配器理论上可以在今天的 Rust 中被接受，因为 `[,]` 不能是合法表达式的一部分，因此解析总是无歧义的。然而，因为 `[` 可以开始尾随表达式，所以 `[` 不是一个可以安全排除出现在表达式之后的字符。如果 `[,]` 在 Rust 的后续版本中被接受，这个匹配器将变得歧义或解析错误，破坏可运行的代码。像 `$i:expr,` 或 `$i:expr;` 这样的匹配器是合法的，因为 `,` 和 `;` 是合法的表达式分隔符。具体规则如下：

r[macro.decl.follow-set.token-expr-stmt]
  * `expr` 和 `stmt` 只能后跟以下之一：`=>`、`,` 或 `;`。

r[macro.decl.follow-set.token-pat_param]
  * `pat_param` 只能后跟以下之一：`=>`、`,`、`=`、`|`、`if` 或 `in`。

r[macro.decl.follow-set.token-pat]
  * `pat` 只能后跟以下之一：`=>`、`,`、`=`、`if` 或 `in`。

r[macro.decl.follow-set.token-path-ty]
  * `path` 和 `ty` 只能后跟以下之一：`=>`、`,`、`=`、`|`、`;`、`:`、`>`、`>>`、`[`、`{`、`as`、`where`，或者是 `block` 片段限定符的宏变量。

r[macro.decl.follow-set.token-vis]
  * `vis` 只能后跟以下之一：`,`、一个非原始 `priv` 之外的标识符、任何可以开始类型的词法单元，或者是具有 `ident`、`ty` 或 `path` 片段限定符的元变量。

r[macro.decl.follow-set.token-other]
  * 所有其他片段限定符没有限制。

r[macro.decl.follow-set.edition2021]
> [!EDITION-2021]
> 在 2021 版本之前，`pat` 也可以后跟 `|`。

r[macro.decl.follow-set.repetition]
当涉及重复时，规则适用于每种可能的展开次数，并将分隔符考虑在内。这意味着：

  * 如果重复包含分隔符，该分隔符必须能够跟在重复内容之后。
  * 如果重复可以多次（`*` 或 `+`）重复，那么内容必须能够跟在自身之后。
  * 重复的内容必须能够跟在前面的内容之后，后面的内容必须能够跟在重复的内容之后。
  * 如果重复可以匹配零次（`*` 或 `?`），那么后面的内容必须能够跟在前面的内容之后。

更多细节请参见[形式规范][formal specification]。

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
