r[expr.path]
# 路径表达式

r[expr.path.syntax]
```grammar,expressions
PathExpression ->
      PathInExpression
    | QualifiedPathInExpression
```

r[expr.path.intro]
用作表达式上下文的[路径][path]表示局部变量或项。

r[expr.path.place]
解析为局部变量或静态变量的路径表达式是[位置表达式]；其他路径是[值表达式]。

r[expr.path.safety]
使用 [`static mut`] 变量需要 [`unsafe` 块]。

```rust
# mod globals {
#     pub static STATIC_VAR: i32 = 5;
#     pub static mut STATIC_MUT_VAR: i32 = 7;
# }
# let local_var = 3;
local_var;
globals::STATIC_VAR;
unsafe { globals::STATIC_MUT_VAR };
let some_constructor = Some::<i32>;
let push_integer = Vec::<i32>::push;
let slice_reverse = <[i32]>::reverse;
```

r[expr.path.const]
关联常量的求值与 [`const` 块]的处理方式相同。

[place expressions]: ../expressions.md#place-expressions-and-value-expressions
[value expressions]: ../expressions.md#place-expressions-and-value-expressions
[path]: ../paths.md
[`static mut`]: ../items/static-items.md#mutable-statics
[`unsafe` block]: block-expr.md#unsafe-blocks
[`const` blocks]: block-expr.md#const-blocks
