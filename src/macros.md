r[macro]
# 宏

r[macro.intro]
Rust 的功能和语法可以通过称为宏的自定义定义进行扩展。它们被赋予名称，并通过一致的语法进行调用：`some_extension!(...)`。

定义新宏有两种方式：

* [声明宏][Macros by Example]以更高层次的声明性方式定义新语法。
* [过程宏][Procedural Macros]定义类函数宏、自定义派生宏和自定义属性宏，使用对输入词法单元进行操作的函数。

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
    Token _排除 [delimiters][lex.token.delim]_ | DelimTokenTree

MacroInvocationSemi ->
      SimplePath `!` `(` TokenTree* `)` `;`
    | SimplePath `!` `[` TokenTree* `]` `;`
    | SimplePath `!` `{` TokenTree* `}`
```

r[macro.invocation.intro]
宏调用在编译期展开宏，并用宏的结果替换调用。宏可以在以下情形中调用：

r[macro.invocation.expr]
* [表达式][expressions]和[语句][statements]

r[macro.invocation.pattern]
* [模式][patterns]

r[macro.invocation.type]
* [类型][types]

r[macro.invocation.item]
* [项][items]，包括[关联项][associated items]

r[macro.invocation.nested]
* [`macro_rules`] 转录器

r[macro.invocation.extern]
* [外部块][External blocks]

r[macro.invocation.item-statement]
当用作项或语句时，使用 [MacroInvocationSemi] 形式，在不使用花括号的情况下要求在末尾带分号。[可见性限定符]绝不允许出现在宏调用或 [`macro_rules`] 定义之前。

```rust
// 用作表达式。
let x = vec![1,2,3];

// 用作语句。
println!("Hello!");

// 用于模式中。
macro_rules! pat {
    ($i:ident) => (Some($i))
}

if let pat!(x) = Some(1) {
    assert_eq!(x, 1);
}

// 用于类型中。
macro_rules! Tuple {
    { $A:ty, $B:ty } => { ($A, $B) };
}

type N2 = Tuple!(i32, i32);

// 用作项。
# use std::cell::RefCell;
thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

// 用作关联项。
macro_rules! const_maker {
    ($t:ty, $v:tt) => { const CONST: $t = $v; };
}
trait T {
    const_maker!{i32, 7}
}

// 宏中的宏调用。
macro_rules! example {
    () => { println!("宏中的宏调用！") };
}
// 先展开外部宏 `example`，再展开内部宏 `println`。
example!();
```

r[macro.invocation.name-resolution]

宏调用可以通过两种作用域解析：

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
