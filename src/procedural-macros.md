r[macro.proc]
# 过程宏

r[macro.proc.intro]
*过程宏*允许通过执行函数来创建语法扩展。过程宏有三种形式：

* [类函数宏][Function-like macros] - `custom!(...)`
* [派生宏][Derive macros] - `#[derive(CustomDerive)]`
* [属性宏][Attribute macros] - `#[CustomAttribute]`

过程宏允许你在编译期运行代码来操作 Rust 语法，既消费也产生 Rust 语法。你可以将过程宏视为从一个 AST 到另一个 AST 的函数。

r[macro.proc.def]
过程宏必须定义在具有 `proc-macro` [crate 类型][crate type]的 crate 的根中。这些宏不能在定义它们的 crate 中使用，只能在被导入到其他 crate 时使用。

> [!NOTE]
> 使用 Cargo 时，过程宏 crate 通过在清单中设置 `proc-macro` 键来定义：
>
> ```toml
> [lib]
> proc-macro = true
> ```

r[macro.proc.result]
作为函数，它们必须返回语法、panic 或无限循环。返回的语法根据过程宏的类型替换或添加语法。Panic 会被编译器捕获并转换为编译器错误。无限循环不会被编译器捕获，会导致编译器挂起。

过程宏在编译期间运行，因此拥有与编译器相同的资源。例如，标准输入、错误和输出与编译器可以访问的相同。同样，文件访问也相同。因此，过程宏具有与 [Cargo 构建脚本][Cargo's build scripts]相同的安全考量。

r[macro.proc.error]
过程宏有两种报告错误的方式。第一种是 panic。第二种是发出 [`compile_error`] 宏调用。

r[macro.proc.proc_macro-crate]
## `proc_macro` crate

r[macro.proc.proc_macro-crate.intro]
过程宏 crate 几乎总是会链接到编译器提供的 [`proc_macro` crate]。`proc_macro` crate 提供了编写过程宏所需的类型以及使其更便捷的工具。

r[macro.proc.proc_macro-crate.token-stream]
此 crate 主要包含一个 [`TokenStream`] 类型。过程宏操作的是*词法单元流*而非 AST 节点，这对于编译器和过程宏来说都是更稳定的接口。*词法单元流*大致等价于 `Vec<TokenTree>`，其中 `TokenTree` 可以粗略地视为词法词法单元。例如，`foo` 是一个 `Ident` 词法单元，`.` 是一个 `Punct` 词法单元，而 `1.2` 是一个 `Literal` 词法单元。与 `Vec<TokenTree>` 不同，`TokenStream` 类型的克隆成本很低。

r[macro.proc.proc_macro-crate.span]
所有词法单元都有一个关联的 `Span`。`Span` 是一个不透明的值，不能修改但可以创建。`Span` 表示程序中源代码的某个范围，主要用于错误报告。虽然你无法修改 `Span` 本身，但你始终可以更改与任何词法单元*关联*的 `Span`，例如通过从另一个词法单元获取 `Span`。

r[macro.proc.hygiene]
## 过程宏的卫生性

过程宏是*非卫生性的*。这意味着它们的行为就像输出的词法单元流被简单地内联写入到其旁边的代码中一样。这意味着它会受到外部项的影响，也会影响外部导入。

宏作者需要注意确保其宏在此限制下尽可能在多种上下文中工作。这通常包括对库中的项使用绝对路径（例如，`::std::option::Option` 而不是 `Option`），或确保生成的函数具有不太可能与其他函数冲突的名称（如 `__internal_foo` 而不是 `foo`）。

<!-- TODO: rule name needs improvement -->
<!-- template:attributes -->
r[macro.proc.proc_macro]
## `proc_macro` 属性

r[macro.proc.proc_macro.intro]
*`proc_macro` [属性][attributes]* 定义了一个[类函数][macro.invocation]过程宏。

> [!EXAMPLE]
> 此宏定义忽略其输入并发出一个函数 `answer` 到其作用域中。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> # #![crate_type = "proc-macro"]
> extern crate proc_macro;
> use proc_macro::TokenStream;
>
> #[proc_macro]
> pub fn make_answer(_item: TokenStream) -> TokenStream {
>     "fn answer() -> u32 { 42 }".parse().unwrap()
> }
> ```
>
> 我们可以在二进制 crate 中使用它来向标准输出打印 "42"。
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> extern crate proc_macro_examples;
> use proc_macro_examples::make_answer;
>
> make_answer!();
>
> fn main() {
>     println!("{}", answer());
> }
> ```

r[macro.proc.proc_macro.syntax]
`proc_macro` 属性使用 [MetaWord] 语法。

r[macro.proc.proc_macro.allowed-positions]
`proc_macro` 属性只能应用于类型为 `fn(TokenStream) -> TokenStream` 的 `pub` 函数，其中 [`TokenStream`] 来自 [`proc_macro` crate]。它必须具有 ["Rust" ABI][items.fn.extern]。不允许使用其他函数限定符。它必须位于 crate 的根中。

r[macro.proc.proc_macro.duplicates]
`proc_macro` 属性在函数上只能指定一次。

r[macro.proc.proc_macro.namespace]
`proc_macro` 属性在 crate 根的[宏命名空间][macro namespace]中公开定义与函数同名的宏。

r[macro.proc.proc_macro.behavior]
类函数过程宏的类函数宏调用会将宏调用定界符内的内容作为输入 [`TokenStream`] 参数传递，并用函数的输出 [`TokenStream`] 替换整个宏调用。

r[macro.proc.proc_macro.invocation]
类函数过程宏可以在任何宏调用位置被调用，包括：

- [语句][Statements]
- [表达式][Expressions]
- [模式][Patterns]
- [类型表达式][Type expressions]
- [项][Item]位置，包括 [`extern` 块][`extern` blocks]中的项
- 固有和 trait [实现][implementations]
- [Trait 定义][Trait definitions]

<!-- template:attributes -->
r[macro.proc.derive]
## `proc_macro_derive` 属性

r[macro.proc.derive.intro]
将 *`proc_macro_derive` [属性][attribute]* 应用于函数定义了一个*派生宏*，可以通过 [`derive` 属性][`derive` attribute]调用。这些宏接收[结构体][struct]、[枚举][enum]或[联合体][union]定义的词法单元流，并可以在其后发出新的[项][items]。它们还可以声明和使用[派生宏辅助属性][derive macro helper attributes]。

> [!EXAMPLE]
> 此派生宏忽略其输入并追加定义函数的词法单元。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> # #![crate_type = "proc-macro"]
> extern crate proc_macro;
> use proc_macro::TokenStream;
>
> #[proc_macro_derive(AnswerFn)]
> pub fn derive_answer_fn(_item: TokenStream) -> TokenStream {
>     "fn answer() -> u32 { 42 }".parse().unwrap()
> }
> ```
>
> 要使用它，我们可以这样写：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> extern crate proc_macro_examples;
> use proc_macro_examples::AnswerFn;
>
> #[derive(AnswerFn)]
> struct Struct;
>
> fn main() {
>     assert_eq!(42, answer());
> }
> ```

r[macro.proc.derive.syntax]
`proc_macro_derive` 属性的语法是：

```grammar,attributes
@root ProcMacroDeriveAttribute ->
    `proc_macro_derive` `(` DeriveMacroName ( `,` DeriveMacroAttributes )? `,`? `)`

DeriveMacroName -> IDENTIFIER

DeriveMacroAttributes ->
    `attributes` `(` ( IDENTIFIER (`,` IDENTIFIER)* `,`?)? `)`
```

派生宏的名称由 [DeriveMacroName] 给出。可选的 `attributes` 参数在 [macro.proc.derive.attributes] 中描述。

r[macro.proc.derive.allowed-positions]
`proc_macro_derive` 属性只能应用于在 crate 根中定义的具有 [Rust ABI][items.fn.extern] 的类型为 `fn(TokenStream) -> TokenStream` 的 `pub` 函数，其中 [`TokenStream`] 来自 [`proc_macro` crate]。函数可以是 `const` 的，可以使用 `extern` 显式指定 Rust ABI，但不能使用任何其他[限定符][FunctionQualifiers]（例如，不能是 `async` 或 `unsafe`）。

r[macro.proc.derive.duplicates]
`proc_macro_derive` 属性在函数上只能使用一次。

r[macro.proc.derive.namespace]
`proc_macro_derive` 属性在 crate 根的[宏命名空间][macro namespace]中公开定义该派生宏。

r[macro.proc.derive.output]
输入 [`TokenStream`] 是应用 `derive` 属性的项的 token 流。输出 [`TokenStream`] 必须是（可能为空的）一组项。这些项追加在输入项之后，位于同一[模块][module]或[块][block]中。

r[macro.proc.derive.attributes]
### 派生宏辅助属性

r[macro.proc.derive.attributes.intro]
派生宏可以声明*派生宏辅助属性*，以便在应用该派生宏的[项][item]作用域中使用。这些[属性][attributes]是[惰性的][inert]。虽然它们的目的是供声明它们的宏使用，但它们可以被任何宏看到。

r[macro.proc.derive.attributes.decl]
派生宏的辅助属性通过在 `proc_macro_derive` 属性的 `attributes` 列表中添加其标识符来声明。

> [!EXAMPLE]
> 这里声明了一个辅助属性，然后忽略它。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> # #![crate_type="proc-macro"]
> # extern crate proc_macro;
> # use proc_macro::TokenStream;
> #
> #[proc_macro_derive(WithHelperAttr, attributes(helper))]
> pub fn derive_with_helper_attr(_item: TokenStream) -> TokenStream {
>     TokenStream::new()
> }
> ```
>
> 要使用它，我们可以这样写：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[derive(WithHelperAttr)]
> struct Struct {
>     #[helper] field: (),
> }
> ```

r[macro.proc.derive.attributes.scope]
当派生宏调用应用于一个项时，该派生宏引入的辅助属性进入作用域：1）适用于应用于该项的属性，且在派生宏调用之后按词法应用；2）适用于该项内部的字段和变体的属性。

> [!NOTE]
> rustc 目前允许在引入它们的宏之前使用派生辅助属性。这些乱序使用的派生辅助属性不会遮蔽其他属性宏。此行为已弃用并计划移除。
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[helper] // 已弃用，将来将成为硬错误。
> #[derive(WithHelperAttr)]
> struct Struct {
>     field: (),
> }
> ```
>
> 更多细节请参见 [Rust 问题 #79202](https://github.com/rust-lang/rust/issues/79202)。


<!-- template:attributes -->
r[macro.proc.attribute]
## `proc_macro_attribute` 属性

r[macro.proc.attribute.intro]
*`proc_macro_attribute` [属性][attributes]* 定义一个*属性宏*，它可以用作[外部属性][attributes]。

> [!EXAMPLE]
> 此属性宏接收输入流并按原样发出，实际上是一个无操作属性。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> # #![crate_type = "proc-macro"]
> # extern crate proc_macro;
> # use proc_macro::TokenStream;
>
> #[proc_macro_attribute]
> pub fn return_as_is(_attr: TokenStream, item: TokenStream) -> TokenStream {
>     item
> }
> ```

> [!EXAMPLE]
> 这里展示了编译器输出中属性宏看到的字符串化 [`TokenStream`s]。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> // my-macro/src/lib.rs
> # extern crate proc_macro;
> # use proc_macro::TokenStream;
> #[proc_macro_attribute]
> pub fn show_streams(attr: TokenStream, item: TokenStream) -> TokenStream {
>     println!("attr: \"{attr}\"");
>     println!("item: \"{item}\"");
>     item
> }
> ```
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> // src/lib.rs
> extern crate my_macro;
>
> use my_macro::show_streams;
>
> // 示例：基本函数。
> #[show_streams]
> fn invoke1() {}
> // out: attr: ""
> // out: item: "fn invoke1() {}"
>
> // 示例：带输入的属性。
> #[show_streams(bar)]
> fn invoke2() {}
> // out: attr: "bar"
> // out: item: "fn invoke2() {}"
>
> // 示例：输入中的多个词法单元。
> #[show_streams(multiple => tokens)]
> fn invoke3() {}
> // out: attr: "multiple => tokens"
> // out: item: "fn invoke3() {}"
>
> // 示例：输入中的定界符。
> #[show_streams { delimiters }]
> fn invoke4() {}
> // out: attr: "delimiters"
> // out: item: "fn invoke4() {}"
> ```

r[macro.proc.attribute.syntax]
`proc_macro_attribute` 属性使用 [MetaWord] 语法。

r[macro.proc.attribute.allowed-positions]
`proc_macro_attribute` 属性只能应用于类型为 `fn(TokenStream, TokenStream) -> TokenStream` 的 `pub` 函数，其中 [`TokenStream`] 来自 [`proc_macro` crate]。它必须具有 ["Rust" ABI][items.fn.extern]。不允许使用其他函数限定符。它必须位于 crate 的根中。

r[macro.proc.attribute.duplicates]
`proc_macro_attribute` 属性在函数上只能指定一次。

r[macro.proc.attribute.namespace]
`proc_macro_attribute` 属性在 crate 根的[宏命名空间][macro namespace]中定义与函数同名的属性。

r[macro.proc.attribute.use-positions]
属性宏只能用于：

- [项][Items]
- [`extern` 块][`extern` blocks]中的项
- 固有和 trait [实现][implementations]
- [Trait 定义][Trait definitions]

r[macro.proc.attribute.behavior]
第一个 [`TokenStream`] 参数是属性名称后面但不包括外部定界符的定界词法单元树。如果应用的属性只包含属性名称，或属性名称后跟空的定界符，则 [`TokenStream`] 为空。

第二个 [`TokenStream`] 是[项][item]的其余部分，包括[项][item]上的其他[属性][attributes]。

应用该属性的项被返回的 [`TokenStream`] 中的零个或多个项替换。

r[macro.proc.token]
## 声明宏词法单元与过程宏词法单元

r[macro.proc.token.intro]
声明式 `macro_rules` 宏和过程宏对词法单元（或者说 [`TokenTree`s]）使用相似但不同的定义。

r[macro.proc.token.macro_rules]
`macro_rules` 中的词法单元树（对应 `tt` 匹配器）定义为：
- 定界组（`(...)`、`{...}` 等）
- 语言支持的所有运算符，包括单字符和多字符运算符（`+`、`+=`）。
    - 注意此集合不包括单引号 `'`。
- 字面量（`"string"`、`1` 等）
    - 注意取反（如 `-1`）从来不是此类字面量词法单元的一部分，而是一个单独的运算符词法单元。
- 标识符，包括关键字（`ident`、`r#ident`、`fn`）
- 生命周期（`'ident`）
- `macro_rules` 中的元变量替换（例如，`macro_rules! mac { ($my_expr: expr) => { $my_expr } }` 中 `mac` 展开后的 `$my_expr`，无论传递的表达式是什么，它都将被视为单个词法单元树）

r[macro.proc.token.tree]
过程宏中的词法单元树定义为：
- 定界组（`(...)`、`{...}` 等）
- 语言支持的运算符中使用的所有标点字符（`+`，但不包括 `+=`），以及单引号 `'` 字符（通常用于生命周期，参见下文关于生命周期拆分和合并行为的说明）
- 字面量（`"string"`、`1` 等）
    - 支持将取反（如 `-1`）作为整数和浮点数字面量的一部分。
- 标识符，包括关键字（`ident`、`r#ident`、`fn`）

r[macro.proc.token.conversion.intro]
当词法单元流传入和传出过程宏时，这两种定义之间的不匹配会得到处理。注意，以下转换可能会延迟进行，因此如果词法单元实际上未被检查，它们可能不会发生。

r[macro.proc.token.conversion.to-proc_macro]
当传递给过程宏时：
- 所有多字符运算符被拆分为单字符。
- 生命周期被拆分为一个 `'` 字符和一个标识符。
- 关键字元变量 [`$crate`] 作为单个标识符传递。
- 所有其他元变量替换表示为其底层的词法单元流。
    - 当为了保持解析优先级而有必要时，此类词法单元流可能会被包装到具有隐式定界符（[`Delimiter::None`]）的定界组（[`Group`]）中。
    - `tt` 和 `ident` 替换永远不会被包装到这样的组中，始终表示为其底层的词法单元树。

r[macro.proc.token.conversion.from-proc_macro]
当从过程宏发出时：
- 标点字符在适用时合并为多字符运算符。
- 与标识符连接的单引号 `'` 合并为生命周期。
- 负数字面量转换为两个词法单元（`-` 和字面量），可能在需要保持解析优先级时包装到具有隐式定界符（[`Delimiter::None`]）的定界组（[`Group`]）中。

r[macro.proc.token.doc-comment]
注意，声明宏和过程宏都不支持文档注释词法单元（例如 `/// Doc`），因此它们在传递给宏时总是被转换为表示其等效的 `#[doc = r"str"]` 属性的词法单元流。

[Attribute macros]: #the-proc_macro_attribute-attribute
[Cargo's build scripts]: ../cargo/reference/build-scripts.html
[Derive macros]: macro.proc.derive
[Function-like macros]: #the-proc_macro-attribute
[`$crate`]: macro.decl.hygiene.crate
[`Delimiter::None`]: proc_macro::Delimiter::None
[`Group`]: proc_macro::Group
[`TokenStream`]: proc_macro::TokenStream
[`TokenStream`s]: proc_macro::TokenStream
[`TokenTree`s]: proc_macro::TokenTree
[`derive` attribute]: attributes/derive.md
[`extern` blocks]: items/external-blocks.md
[`macro_rules`]: macros-by-example.md
[`proc_macro` crate]: proc_macro
[attribute]: attributes.md
[attributes]: attributes.md
[block]: expressions/block-expr.md
[crate type]: linkage.md
[derive macro helper attributes]: #derive-macro-helper-attributes
[enum]: items/enumerations.md
[expressions]: expressions.md
[function]: items/functions.md
[implementations]: items/implementations.md
[inert]: attributes.md#active-and-inert-attributes
[item]: items.md
[items]: items.md
[macro namespace]: names/namespaces.md
[module]: items/modules.md
[patterns]: patterns.md
[public]: visibility-and-privacy.md
[statements]: statements.md
[struct]: items/structs.md
[trait definitions]: items/traits.md
[type expressions]: types.md#type-expressions
[type]: types.md
[union]: items/unions.md
