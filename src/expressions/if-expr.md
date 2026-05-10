r[expr.if]
# `if` 表达式

r[expr.if.syntax]
```grammar,expressions
IfExpression ->
    `if` Conditions BlockExpression
    (`else` ( BlockExpression | IfExpression ) )?

Conditions ->
      Expression _except [StructExpression]_
    | LetChain

LetChain -> LetChainCondition ( `&&` LetChainCondition )*

LetChainCondition ->
      Expression _except [ExcludedConditions]_
    | OuterAttribute* `let` Pattern `=` Scrutinee _except [ExcludedConditions]_

@root ExcludedConditions ->
      StructExpression
    | LazyBooleanExpression
    | RangeExpr
    | RangeFromExpr
    | RangeInclusiveExpr
    | AssignmentExpression
    | CompoundAssignmentExpression
```
<!-- TODO: 上面的结构体例外需要澄清，参见 https://github.com/rust-lang/reference/issues/1808
     链式语法可能需要改进，参见 https://github.com/rust-lang/reference/issues/1811
-->

r[expr.if.intro]
`if` 表达式的语法是由 `&&` 分隔的一个或多个条件操作数的序列，后跟一个结果块，任意数量的 `else if` 条件和块，以及一个可选的尾部 `else` 块。

r[expr.if.condition]
条件操作数必须是一个具有[布尔类型][boolean type]的[表达式][Expression]或一个条件 `let` 匹配。

r[expr.if.condition-true]
如果所有条件操作数都求值为 `true` 且所有 `let` 模式都成功匹配其[受检者][scrutinee]，则执行结果块，并跳过任何后续的 `else if` 或 `else` 块。

r[expr.if.else-if]
如果任何条件操作数求值为 `false` 或任何 `let` 模式未匹配其受检者，则跳过结果块，并求值任何后续的 `else if` 条件。

r[expr.if.else]
如果所有 `if` 和 `else if` 条件都求值为 `false`，则执行 `else` 块（如果有的话）。

r[expr.if.result]
`if` 表达式求值为与执行的块相同的值，如果没有块被执行则求值为 `()`。

r[expr.if.type]
`if` 表达式在所有情况下必须具有相同的类型。

```rust
# let x = 3;
if x == 4 {
    println!("x is four");
} else if x == 3 {
    println!("x is three");
} else {
    println!("x is something else");
}

// `if` 可以用作表达式。
let y = if 12 * 15 > 150 {
    "Bigger"
} else {
    "Smaller"
};
assert_eq!(y, "Bigger");
```

r[expr.if.diverging]
如果条件表达式发散或所有分支都发散，则 `if` 表达式[发散][diverges]。

```rust,no_run
fn diverging_condition() -> ! {
    // 因为条件表达式发散而发散
    if loop {} {
        ()
    } else {
        ()
    };
    // 上面的分号很重要：`if` 表达式的类型是 `()`，
    // 尽管它发散了。当最终体表达式被省略时，体的类型
    // 被推断为 !，因为函数体发散。如果没有分号，
    // `if` 将是类型为 `()` 的尾部表达式，这将无法匹配返回类型 `!`。
}

fn diverging_arms() -> ! {
    // 因为所有分支都发散而发散
    if true {
        loop {}
    } else {
        loop {}
    }
}
```

r[expr.if.let]
## `if let` 模式

r[expr.if.let.intro]
`if` 条件中的 `let` 模式允许在模式成功匹配时将新变量绑定到作用域中。

以下示例展示了使用 `let` 模式的绑定：

```rust
let dish = ("Ham", "Eggs");

// 因为模式被反驳，此体将被跳过。
if let ("Bacon", b) = dish {
    println!("Bacon is served with {}", b);
} else {
    // 改为执行此块。
    println!("No bacon will be served");
}

// 此体将执行。
if let ("Ham", b) = dish {
    println!("Ham is served with {}", b);
}

if let _ = 5 {
    println!("Irrefutable patterns are always true");
}
```

r[expr.if.let.or-pattern]
可以使用 `|` 运算符指定多个模式。这与 [`match` 表达式][`match` expressions]中的 `|` 具有相同的语义：

```rust
enum E {
    X(u8),
    Y(u8),
    Z(u8),
}
let v = E::Y(12);
if let E::X(n) | E::Y(n) = v {
    assert_eq!(n, 12);
}
```

r[expr.if.chains]
## 条件链

r[expr.if.chains.intro]
多个条件操作数可以用 `&&` 分隔。

r[expr.if.chains.order]
类似于 `&&` [LazyBooleanExpression]，每个操作数从左到右求值，直到某个操作数求值为 `false` 或 `let` 匹配失败，在这种情况下后续操作数不会被求值。

r[expr.if.chains.bindings]
每个模式的绑定被放入作用域，以供下一个条件操作数和结果块使用。

以下是链式多个表达式的示例，混合了 `let` 绑定和布尔表达式，并且表达式可以引用先前表达式的模式绑定：

```rust
fn single() {
    let outer_opt = Some(Some(1i32));

    if let Some(inner_opt) = outer_opt
        && let Some(number) = inner_opt
        && number == 1
    {
        println!("Peek a boo");
    }
}
```

上面的代码等价于以下不使用条件链的代码：

```rust
fn nested() {
    let outer_opt = Some(Some(1i32));

    if let Some(inner_opt) = outer_opt {
        if let Some(number) = inner_opt {
            if number == 1 {
                println!("Peek a boo");
            }
        }
    }
}
```

r[expr.if.chains.or]
如果任何条件操作数是 `let` 模式，则由于与 `let` 受检者的歧义和优先级，所有条件操作数都不能是 `||` [惰性布尔运算符表达式][expr.bool-logic]。如果需要 `||` 表达式，可以使用括号。例如：

```rust
# let foo = Some(123);
# let condition1 = true;
# let condition2 = false;
// 此处需要括号。
if let Some(x) = foo && (condition1 || condition2) { /*...*/ }
```

r[expr.if.edition2024]
> [!EDITION-2024]
> 在 2024 版本之前，不支持 let 链。即 [LetChain] 语法在 `if` 表达式中是不允许的。

[`match` expressions]: match-expr.md
[boolean type]: ../types/boolean.md
[diverges]: divergence
[scrutinee]: ../glossary.md#scrutinee
