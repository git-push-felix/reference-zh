r[macro.proc]
# 过程宏

r[macro.proc.intro]
*过程宏*允许以函数执行的方式创建语法扩展。过程宏有三种类型：

* [类函数宏][Function-like macros] - `custom!(...)`
* [派生宏][Derive macros] - `#[derive(CustomDerive)]`
* [属性宏][Attribute macros] - `#[CustomAttribute]`

过程宏允许你在编译时运行操作 Rust 语法的代码，既可以消费也可以产出 Rust 语法。你可以将过程宏看作是 AST 到另一个 AST 的函数。

r[macro.proc.def]
过程宏必须在 crate 类型为 `proc-macro` 的 crate 的根部定义。这些宏不能在定义它们的 crate 中使用，只能在另一个 crate 中导入后使用。

> [!NOTE]
> 使用 Cargo 时，过程宏 crate 在清单中通过 `proc-macro` 键定义：
>
> ```toml
> [lib]
> proc-macro = true
> ```

r[macro.proc.result]
作为函数，它们必须返回语法、panic 或无限循环。返回的语法根据过程宏的类型来替换或添加语法。panic 会被编译器捕获并转化为编译错误。无限循环不会被编译器捕获，会导致编译器挂起。

过程宏在编译期间运行，因此拥有编译器所拥有的相同资源。例如，标准输入、错误和输出与编译器可以访问的相同。同样，文件访问也是相同的。因此，过程宏具有与 [Cargo 构建脚本][Cargo's build scripts]相同的安全考虑。

r[macro.proc.error]
过程宏有两种报告错误的方式。第一种是 panic。第二种是发出 [`compile_error`] 宏调用。

r[macro.proc.proc_macro-crate]
## `proc_macro` crate

r[macro.proc.proc_macro-crate.intro]
过程宏 crate 几乎总是会链接到编译器提供的 [`proc_macro` crate]。`proc_macro` crate 提供了编写过程宏所需的类型以及便利设施。

r[macro.proc.proc_macro-crate.token-stream]
此 crate 主要包含一个 [`TokenStream`] 类型。过程宏操作*token 流*而非 AST 节点，这对编译器和过程宏来说都是一个随时间推移更稳定的接口。*token 流*大致等价于 `Vec<TokenTree>`，其中 `TokenTree` 可以粗略地视作词法 token。例如 `foo` 是一个 `Ident` token，`.` 是一个 `Punct` token，`1.2` 是一个 `Literal` token。与 `Vec<TokenTree>` 不同，`TokenStream` 类型克隆开销很低。

r[macro.proc.proc_macro-crate.span]
所有 token 都有一个关联的 `Span`。`Span` 是一个不可修改但可以构造的不透明值。`Span` 表示程序中一段源代码的范围，主要用于错误报告。虽然你不能修改 `Span` 本身，但你始终可以更改与任何 token *关联*的 `Span`，例如从另一个 token 获取 `Span`。

r[macro.proc.hygiene]
## 过程宏的卫生性

过程宏是*非卫生的*。这意味着它们的行为就好像输出的 token 流被简单地内联写入到其相邻的代码中一样。这意味着它受外部项的影响，也会影响外部导入。

宏作者需要小心确保其宏在此限制下能在尽可能多的上下文中正常工作。这通常包括使用库中项的绝对路径（例如 `::std::option::Option` 而不是 `Option`），或者确保生成的函数名称不太可能与其他函数冲突（如 `__internal_foo` 而不是 `foo`）。

<!-- TODO: rule name needs improvement -->
<!-- template:attributes -->
r[macro.proc.proc_macro]
## `proc_macro` 属性

r[macro.proc.proc_macro.intro]
*`proc_macro` [属性][attributes]* 定义了一个[类函数][macro.invocation]过程宏。

> [!EXAMPLE]
> 此宏定义忽略其输入，并在其作用域中生成一个函数 `answer`。
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
> 我们可以在二进制 crate 中使用它来将 "42" 打印到标准输出。
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
`proc_macro` 属性只能应用于具有类型 `fn(TokenStream) -> TokenStream` 的 `pub` 函数，其中 [`TokenStream`] 来自 [`proc_macro` crate]。它必须具有 ["Rust" ABI][items.fn.extern]。不允许使用其他函数限定符。它必须位于 crate 的根部。

r[macro.proc.proc_macro.duplicates]
`proc_macro` 属性在一个函数上只能指定一次。

r[macro.proc.proc_macro.namespace]
`proc_macro` 属性公开地在 crate 根部的[宏命名空间][macro namespace]中定义一个与该函数同名的宏。

r[macro.proc.proc_macro.behavior]
类函数过程宏的类函数宏调用会将宏调用分隔符内的内容作为输入 [`TokenStream`] 参数传入，并用函数的输出 [`TokenStream`] 替换整个宏调用。

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
将 *`proc_macro_derive` [属性]* 应用于一个函数可定义一个*派生宏*，该宏可以由 [`derive` 属性][`derive` attribute]调用。这些宏会接收一个 [struct]、[enum] 或 [union] 定义的 token 流，并且可以在这些定义之后生成新的[项][items]。它们还可以声明和使用[派生宏辅助属性][derive macro helper attributes]。

> [!EXAMPLE]
> 此派生宏忽略其输入，并追加定义函数的 token。
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
> 为了使用它，我们可以这样写：
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
`proc_macro_derive` 属性只能应用于在 crate 根部定义的 `pub` 函数，该函数具有 [Rust ABI][items.fn.extern]，类型为 `fn(TokenStream) -> TokenStream`，其中 [`TokenStream`] 来自 [`proc_macro` crate]。该函数可以是 `const` 的，也可以使用 `extern` 显式指定 Rust ABI，但不能使用任何其他[限定符][FunctionQualifiers]（例如不能是 `async` 或 `unsafe`）。

r[macro.proc.derive.duplicates]
`proc_macro_derive` 属性在一个函数上只能使用一次。

r[macro.proc.derive.namespace]
`proc_macro_derive` 属性在 crate 根部的[宏命名空间][macro namespace]中公开定义派生宏。

r[macro.proc.derive.output]
输入的 [`TokenStream`] 是应用了 `derive` 属性的项的 token 流。输出的 [`TokenStream`] 必须是（可能为空的）一组项。这些项被追加到同一个[模块][module]或[块][block]中的输入项之后。

r[macro.proc.derive.attributes]
### 派生宏辅助属性

r[macro.proc.derive.attributes.intro]
派生宏可以声明*派生宏辅助属性*，这些属性可以在应用了该派生宏的[项][item]的作用域内使用。这些[属性][attributes]是[惰性的][inert]。虽然它们的目的是供声明它们的宏使用，但任何宏都可以看到它们。

r[macro.proc.derive.attributes.decl]
派生宏的辅助属性通过在 `proc_macro_derive` 属性中的 `attributes` 列表中添加其标识符来声明。

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
> 为了使用它，我们可以这样写：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[derive(WithHelperAttr)]
> struct Struct {
>     #[helper] field: (),
> }
> ```

r[macro.proc.derive.attributes.scope]
当派生宏调用被应用于一个项时，该派生宏引入的辅助属性在以下情况下进入作用域：1）应用于该项且在派生宏调用之后按词法顺序出现的属性；2）应用于该项内部字段和变体的属性。

> [!NOTE]
> `rustc` 目前允许在引入它们的宏之前使用派生辅助属性。这种不按顺序使用的派生辅助属性可能不会遮蔽其他属性宏。此行为已被弃用，并计划移除。
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[helper] // 已弃用，未来将成为硬错误。
> #[derive(WithHelperAttr)]
> struct Struct {
>     field: (),
> }
> ```
>
> 更多细节请参阅 [Rust issue #79202](https://github.com/rust-lang/rust/issues/79202)。


<!-- template:attributes -->
r[macro.proc.attribute]
## `proc_macro_attribute` 属性

r[macro.proc.attribute.intro]
*`proc_macro_attribute` [属性][attributes]* 定义了一个*属性宏*，该宏可以作为[外部属性][attributes]来使用。

> [!EXAMPLE]
> 此属性宏接受输入流并原样输出，实际上是一个无操作属性。
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
> 这展示了编译器输出中属性宏所看到的字符串化的 [`TokenStream`s]。
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
> // Example: Basic function.
> #[show_streams]
> fn invoke1() {}
> // out: attr: ""
> // out: item: "fn invoke1() {}"
>
> // Example: Attribute with input.
> #[show_streams(bar)]
> fn invoke2() {}
> // out: attr: "bar"
> // out: item: "fn invoke2() {}"
>
> // Example: Multiple tokens in the input.
> #[show_streams(multiple => tokens)]
> fn invoke3() {}
> // out: attr: "multiple => tokens"
> // out: item: "fn invoke3() {}"
>
> // Example: Delimiters in the input.
> #[show_streams { delimiters }]
> fn invoke4() {}
> // out: attr: "delimiters"
> // out: item: "fn invoke4() {}"
> ```

r[macro.proc.attribute.syntax]
`proc_macro_attribute` 属性使用 [MetaWord] 语法。

r[macro.proc.attribute.allowed-positions]
`proc_macro_attribute` 属性只能应用于具有类型 `fn(TokenStream, TokenStream) -> TokenStream` 的 `pub` 函数，其中 [`TokenStream`] 来自 [`proc_macro` crate]。它必须具有 ["Rust" ABI][items.fn.extern]。不允许使用其他函数限定符。它必须位于 crate 的根部。

r[macro.proc.attribute.duplicates]
`proc_macro_attribute` 属性在一个函数上只能指定一次。

r[macro.proc.attribute.namespace]
`proc_macro_attribute` 属性在 crate 根部的[宏命名空间][macro namespace]中定义一个与该函数同名的属性。

r[macro.proc.attribute.use-positions]
属性宏只能用于：

- [项][Items]
- [`extern` 块][`extern` blocks]中的项
- 固有和 trait [实现][implementations]
- [Trait 定义][Trait definitions]

r[macro.proc.attribute.behavior]
第一个 [`TokenStream`] 参数是属性名称后的分隔 token 树，但不包括外部分隔符。如果应用的属性仅包含属性名称或属性名称后跟空分隔符，则 [`TokenStream`] 为空。

第二个 [`TokenStream`] 是[项][item]的其余部分，包括该项上的其他[属性][attributes]。

应用该属性的项将被返回的 [`TokenStream`] 中的零个或多个项替换。

r[macro.proc.token]
## 声明宏 token 与过程宏 token

r[macro.proc.token.intro]
声明式 `macro_rules` 宏和过程宏使用相似但不同的 token（或 [`TokenTree`s]）定义。

r[macro.proc.token.macro_rules]
`macro_rules` 中的 token 树（对应 `tt` 匹配器）定义为：
- 分隔组（`(...)`、`{...}` 等）
- 语言支持的所有运算符，包括单字符和多字符（`+`、`+=`）。
    - 注意，此集合不包括单引号 `'`。
- 字面量（`"string"`、`1` 等）
    - 注意，取反（例如 `-1`）永远不是此类字面量 token 的一部分，而是一个独立的运算符 token。
- 标识符，包括关键字（`ident`、`r#ident`、`fn`）
- 生命周期（`'ident`）
- `macro_rules` 中的元变量替换（例如 `macro_rules! mac { ($my_expr: expr) => { $my_expr } }` 中，在 `mac` 展开后，`$my_expr` 将被视为一个单一的 token 树，无论传递的表达式是什么）

r[macro.proc.token.tree]
过程宏中的 token 树定义为：
- 分隔组（`(...)`、`{...}` 等）
- 语言支持的运算符中使用的所有标点字符（`+`，但不包括 `+=`），以及单引号 `'` 字符（通常用于生命周期，关于生命周期的拆分和拼接行为请参见下文）
- 字面量（`"string"`、`1` 等）
    - 支持将取反（例如 `-1`）作为整数和浮点字面量的一部分。
- 标识符，包括关键字（`ident`、`r#ident`、`fn`）

r[macro.proc.token.conversion.intro]
当 token 流传入和传出过程宏时，会处理这两种定义之间的不匹配。请注意，下面的转换可能会延迟进行，因此如果 token 没有被实际检查，转换可能不会发生。

r[macro.proc.token.conversion.to-proc_macro]
当传入过程宏时：
- 所有多字符运算符被拆分为单字符。
- 生命周期被拆分为一个 `'` 字符和一个标识符。
- 关键字元变量 [`$crate`] 作为单个标识符传入。
- 所有其他元变量替换以其底层 token 流的形式表示。
    - 当需要保持解析优先级时，此类 token 流可能被包装到具有隐式分隔符（[`Delimiter::None`]）的分隔组（[`Group`]）中。
    - `tt` 和 `ident` 替换永远不会被包装到此类组中，总是以其底层 token 树的形式表示。

r[macro.proc.token.conversion.from-proc_macro]
当从过程宏发出时：
- 标点字符在适用时会被粘合为多字符运算符。
- 与标识符连接的单引号 `'` 会被粘合为生命周期。
- 负数字面量会被转换为两个 token（`-` 和字面量），可能被包装进具有隐式分隔符（[`Delimiter::None`]）的分隔组（[`Group`]）中，以保持解析优先级。

r[macro.proc.token.doc-comment]
请注意，声明宏和过程宏都不支持文档注释 token（例如 `/// Doc`），因此它们在传递给宏时始终被转换为表示等效的 `#[doc = r"str"]` 属性的 token 流。

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
