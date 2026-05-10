r[statement]
# 语句

r[statement.syntax]
```grammar,statements
Statement ->
      `;`
    | Item
    | LetStatement
    | ExpressionStatement
    | OuterAttribute* MacroInvocationSemi
```

r[statement.intro]
*语句*是[块][block]的一个组成部分，而块又是外部[表达式][expression]或[函数][function]的一个组成部分。

r[statement.kind]
Rust 有两种语句：[声明语句](#declaration-statements)和[表达式语句](#expression-statements)。

r[statement.decl]
## 声明语句

*声明语句*是在所包含的语句块中引入一个或多个*名称*的语句。声明的名称可以表示新的变量或新的[项][item]。

声明语句有两种：项声明和 `let` 语句。

r[statement.item]
### 项声明

r[statement.item.intro]
*项声明语句*的语法形式与[模块][module]中的[项声明][item]完全相同。

r[statement.item.scope]
在语句块中声明一个项会将其[作用域][scope]限制在包含该语句的块中。该项不会被赋予[规范路径][canonical path]，其声明的任何子项也不会。

r[statement.item.associated-scope]
例外情况是，由[实现][implementations]定义的关联项在外部作用域中仍然可以访问，只要该项以及（如果适用）trait 是可访问的。在其他方面，其含义与在模块中声明该项完全相同。

r[statement.item.outer-generics]
不会隐式捕获包含该函数的泛型参数、参数和局部变量。例如，`inner` 不能访问 `outer_var`。

```rust
fn outer() {
  let outer_var = true;

  fn inner() { /* outer_var 在此作用域中不可见 */ }

  inner();
}
```

r[statement.let]
### `let` 语句

r[statement.let.syntax]
```grammar,statements
LetStatement ->
    OuterAttribute* `let` PatternNoTopAlt ( `:` Type )?
    (
          `=` Expression
        | `=` Expression _except [LazyBooleanExpression] or end with a `}`_ `else` BlockExpression
    )? `;`
```

r[statement.let.intro]
*`let` 语句*通过[模式][pattern]引入一组新的[变量][variables]。模式后面可以选择跟上类型标注，然后要么结束，要么跟上一个初始化表达式以及可选的 `else` 块。

r[statement.let.inference]
当没有给出类型标注时，编译器会推断类型，或者在没有足够类型信息进行确定推断时报错。

r[statement.let.scope]
变量声明引入的任何变量从声明点开始可见，直到封闭块作用域结束，除非被另一个变量声明遮蔽。

r[statement.let.constraint]
如果没有 `else` 块，模式必须是不可反驳的。如果存在 `else` 块，模式可以是可反驳的。

r[statement.let.behavior]
如果模式不匹配（这要求模式是可反驳的），则执行 `else` 块。`else` 块必须总是发散（求值为[永不类型][never type]）。

```rust
let (mut v, w) = (vec![1, 2, 3], 42); // 绑定可以是 mut 或 const
let Some(t) = v.pop() else { // 可反驳的模式需要 else 块
    panic!(); // else 块必须发散
};
let [u, v] = [v[0], v[1]] else { // 此模式是不可反驳的，因此编译器
                                 // 会发出 lint 警告，因为 else 块是多余的。
    panic!();
};
```

r[statement.expr]
## 表达式语句

r[statement.expr.syntax]
```grammar,statements
ExpressionStatement ->
      ExpressionWithoutBlock `;`
    | ExpressionWithBlock `;`?
```

r[statement.expr.intro]
*表达式语句*是对[表达式][expression]求值并忽略其结果的语句。通常，表达式语句的目的是触发对表达式求值产生的效果。

r[statement.expr.restriction-semicolon]
仅由[块表达式][block]或控制流表达式组成的表达式，如果在允许语句的上下文中使用，可以省略尾部的分号。这可能导致歧义，因为可能被解析为独立语句或另一个表达式的一部分；在这种情况下，它会被解析为语句。

r[statement.expr.constraint-block]
作为语句使用时，[ExpressionWithBlock] 表达式的类型必须是单元类型。

```rust
# let mut v = vec![1, 2, 3];
v.pop();          // 忽略 pop 返回的元素
if v.is_empty() {
    v.push(5);
} else {
    v.remove(0);
}                 // 分号可以省略。
[1];              // 独立的表达式语句，不是索引表达式。
```

当尾部省略分号时，结果必须是类型 `()`。

```rust
// bad: 块的类型是 i32，不是 ()
// Error: expected `()` because of default return type
// if true {
//   1
// }

// good: 块的类型是 i32
if true {
  1
} else {
  2
};
```

r[statement.attribute]
## 语句上的属性

语句接受[外部属性][outer attributes]。对语句有意义的属性包括 [`cfg`] 和 [lint 检查属性][the lint check attributes]。

[block]: expressions/block-expr.md
[expression]: expressions.md
[function]: items/functions.md
[item]: items.md
[module]: items/modules.md
[never type]: types/never.md
[canonical path]: paths.md#canonical-paths
[implementations]: items/implementations.md
[variables]: variables.md
[outer attributes]: attributes.md
[`cfg`]: conditional-compilation.md
[the lint check attributes]: attributes/diagnostics.md#lint-check-attributes
[pattern]: patterns.md
[scope]: names/scopes.md
