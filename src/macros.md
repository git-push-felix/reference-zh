r[macro]
# 宏

r[macro.intro]
Rust 的功能和语法可以通过称为宏的自定义定义来扩展。它们被赋予名称，并通过一致的语法来调用：`some_extension!(...)`。

有两种定义新宏的方式：

* [声明宏][Macros by Example]以高阶的、声明式的方式定义新语法。
* [过程宏][Procedural Macros]使用操作输入 token 的函数来定义类函数宏、自定义派生宏和自定义属性宏。

r[macro.invocation]
## 宏调用

r[macro.invocation.syntax]
```grammar,macros
MacroInvocation ->
    SimplePath `!` DelimTokenTree

DelimTokenTree ->
      `(` TokenTree* `)`
    | `[` TokenTree* `]`
    | `{` TokenTree* `}`

TokenTree ->
    Token _except [delimiters][lex.token.delim]_ | DelimTokenTree

MacroInvocationSemi ->
      SimplePath `!` `(` TokenTree* `)` `;`
    | SimplePath `!` `[` TokenTree* `]` `;`
    | SimplePath `!` `{` TokenTree* `}`
```

r[macro.invocation.intro]
宏调用在编译时展开宏，并用宏的结果替换调用。宏可以在以下情况下调用：

r[macro.invocation.expr]
* [表达式][expressions]与[语句][statements]

r[macro.invocation.pattern]
* [模式][patterns]

r[macro.invocation.type]
* [类型][types]

r[macro.invocation.item]
* [项][items]（包括[关联项][associated items]）

r[macro.invocation.nested]
* [`macro_rules`] 转写器

r[macro.invocation.extern]
* [外部块][External blocks]

r[macro.invocation.item-statement]
当作为项或语句使用时，对于不使用花括号的情况，应使用 [MacroInvocationSemi] 形式，在末尾需要有一个分号。在宏调用或 [`macro_rules`] 定义之前绝不允许出现[可见性限定符][Visibility qualifiers]。

```rust
// 作为表达式使用。
let x = vec![1,2,3];

// 作为语句使用。
println!("Hello!");

// 在模式中使用。
macro_rules! pat {
    ($i:ident) => (Some($i))
}

if let pat!(x) = Some(1) {
    assert_eq!(x, 1);
}

// 在类型中使用。
macro_rules! Tuple {
    { $A:ty, $B:ty } => { ($A, $B) };
}

type N2 = Tuple!(i32, i32);

// 作为程序项使用。
# use std::cell::RefCell;
thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

// 作为关联程序项使用。
macro_rules! const_maker {
    ($t:ty, $v:tt) => { const CONST: $t = $v; };
}
trait T {
    const_maker!{i32, 7}
}

// 宏中嵌套宏调用。
macro_rules! example {
    () => { println!("Macro call in a macro!") };
}
// 外层宏 `example` 先展开，然后内层宏 `println` 展开。
example!();
```

r[macro.invocation.name-resolution]

宏调用可以通过两种作用域来解析：

- 文本作用域
  - [文本作用域 `macro_rules`](macros-by-example.md#r-macro.decl.scope.textual)
- 基于路径的作用域
  - [基于路径的作用域 `macro_rules`](macros-by-example.md#r-macro.decl.scope.path-based)
  - [过程宏][Procedural Macros]

[External blocks]: items/external-blocks.md
[Macros by Example]: macros-by-example.md
[Procedural Macros]: procedural-macros.md
[`macro_rules`]: macros-by-example.md
[associated items]: items/associated-items.md
[delimiters]: tokens.md#delimiters
[expressions]: expressions.md
[items]: items.md
[patterns]: patterns.md
[statements]: statements.md
[types]: types.md
[visibility qualifiers]: visibility-and-privacy.md
