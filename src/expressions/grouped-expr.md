r[expr.paren]
# 分组表达式

r[expr.paren.syntax]
```grammar,expressions
GroupedExpression -> `(` Expression `)`
```

r[expr.paren.intro]
*括号表达式*包裹一个单独的表达式，求值为该表达式。括号表达式的语法是 `(`，然后是一个表达式（称为*被括操作数*），然后是 `)`。

r[expr.paren.evaluation]
括号表达式求值为被括操作数的值。

r[expr.paren.place-or-value]
如果被括操作数是位置表达式，则括号表达式也是[位置表达式][place]；如果被括操作数是值表达式，则括号表达式也是值表达式。

r[expr.paren.override-precedence]
括号可用于显式修改表达式中子表达式的优先级顺序。

括号表达式的示例：

```rust
let x: i32 = 2 + 3 * 4; // 未加括号
let y: i32 = (2 + 3) * 4; // 加了括号
assert_eq!(x, 14);
assert_eq!(y, 20);
```

必须使用括号的一个示例是当调用作为结构体成员的函数指针时：

```rust
# struct A {
#    f: fn() -> &'static str
# }
# impl A {
#    fn f(&self) -> &'static str {
#        "The method f"
#    }
# }
# let a = A{f: || "The field f"};
#
assert_eq!( a.f (), "The method f");
assert_eq!((a.f)(), "The field f");
```

[place]: ../expressions.md#place-expressions-and-value-expressions
