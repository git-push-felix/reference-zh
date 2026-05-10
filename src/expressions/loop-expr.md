r[expr.loop]
# 循环和其他可中断表达式

r[expr.loop.syntax]
```grammar,expressions
LoopExpression ->
    LoopLabel? (
        InfiniteLoopExpression
      | PredicateLoopExpression
      | IteratorLoopExpression
      | LabelBlockExpression
    )
```

r[expr.loop.intro]
Rust 支持四种循环表达式：

*   [`loop` 表达式](#infinite-loops)表示无限循环。
*   [`while` 表达式](#predicate-loops)在谓词为 false 之前循环。
*   [`for` 表达式](#iterator-loops)从迭代器中提取值，循环直到迭代器为空。
*   [带标签的块表达式][expr.loop.block-labels]恰好运行循环一次，但允许使用 `break` 提前退出循环。

r[expr.loop.break-label]
所有四种循环类型都支持 [`break` 表达式](#break-expressions)和[标签](#loop-labels)。

r[expr.loop.continue-label]
除带标签的块表达式之外，所有类型都支持 [`continue` 表达式](#continue-expressions)。

r[expr.loop.explicit-result]
只有 `loop` 和带标签的块表达式支持[求值为非平凡值](#break-and-loop-values)。

r[expr.loop.infinite]
## 无限循环

r[expr.loop.infinite.syntax]
```grammar,expressions
InfiniteLoopExpression -> `loop` BlockExpression
```

r[expr.loop.infinite.intro]
`loop` 表达式持续重复执行其主体：`loop { println!("I live."); }`。

r[expr.loop.infinite.diverging]
没有关联 `break` 表达式的 `loop` 表达式是[发散][diverging]的，具有类型 [`!`]。

r[expr.loop.infinite.break]
包含关联 [`break` 表达式](#break-expressions)的 `loop` 表达式可能终止，并且必须具有与 `break` 表达式的值兼容的类型。

r[expr.loop.while]
## 谓词循环

r[expr.loop.while.syntax]
```grammar,expressions
PredicateLoopExpression -> `while` Conditions BlockExpression
```

r[expr.loop.while.intro]
`while` 循环表达式允许在一组条件保持为 true 时重复求值一个块。

r[expr.loop.while.condition]
条件操作数必须是一个具有[布尔类型][boolean type]的[表达式][Expression]或一个条件 `let` 匹配。如果所有条件操作数都求值为 `true` 且所有 `let` 模式都成功匹配其[受检者][scrutinee]，则执行循环体块。

r[expr.loop.while.repeat]
循环体成功执行后，重新求值条件操作数以确定是否应再次执行循环体。

r[expr.loop.while.exit]
如果任何条件操作数求值为 `false` 或任何 `let` 模式未匹配其受检者，循环体不被执行，执行继续到 `while` 表达式之后。

r[expr.loop.while.eval]
`while` 表达式求值为 `()`。

示例：

```rust
let mut i = 0;

while i < 10 {
    println!("hello");
    i = i + 1;
}
```

r[expr.loop.while.let]
### `while let` 模式

r[expr.loop.while.let.intro]
`while` 条件中的 `let` 模式允许在模式成功匹配时将新变量绑定到作用域中。以下示例展示了使用 `let` 模式的绑定：

```rust
let mut x = vec![1, 2, 3];

while let Some(y) = x.pop() {
    println!("y = {}", y);
}

while let _ = 5 {
    println!("Irrefutable patterns are always true");
    break;
}
```

r[expr.loop.while.let.desugar]
`while let` 循环等价于包含 [`match` 表达式][`match` expression]的 `loop` 表达式，如下所示。

<!-- ignore: expansion example -->
```rust,ignore
'label: while let PATS = EXPR {
    /* loop body */
}
```

等价于

<!-- ignore: expansion example -->
```rust,ignore
'label: loop {
    match EXPR {
        PATS => { /* loop body */ },
        _ => break,
    }
}
```

r[expr.loop.while.let.or-pattern]
可以使用 `|` 运算符指定多个模式。这与 `match` 表达式中的 `|` 具有相同的语义：

```rust
let mut vals = vec![2, 3, 1, 2, 2];
while let Some(v @ 1) | Some(v @ 2) = vals.pop() {
    // 打印 2, 2, 然后是 1
    println!("{}", v);
}
```

r[expr.loop.while.chains]
### `while` 条件链

r[expr.loop.while.chains.intro]
多个条件操作数可以用 `&&` 分隔。这些具有与 [`if` 条件链][`if` condition chains]相同的语义和限制。

以下是链式多个表达式的示例，混合了 `let` 绑定和布尔表达式，并且表达式可以引用先前表达式的模式绑定：

```rust
fn main() {
    let outer_opt = Some(Some(1i32));

    while let Some(inner_opt) = outer_opt
        && let Some(number) = inner_opt
        && number == 1
    {
        println!("Peek a boo");
        break;
    }
}
```

r[expr.loop.for]
## 迭代器循环

r[expr.loop.for.syntax]
```grammar,expressions
IteratorLoopExpression ->
    `for` Pattern `in` Expression _except [StructExpression]_ BlockExpression
```
<!-- TODO: 上面的例外不准确，参见 https://github.com/rust-lang/reference/issues/569 -->

r[expr.loop.for.intro]
`for` 表达式是一种语法结构，用于遍历由 `std::iter::IntoIterator` 实现提供的元素。

r[expr.loop.for.condition]
如果迭代器产生一个值，该值与不可反驳的模式匹配，则执行循环体，然后控制返回到 `for` 循环的头部。如果迭代器为空，`for` 表达式完成。

对数组内容进行 `for` 循环的示例：

```rust
let v = &["apples", "cake", "coffee"];

for text in v {
    println!("I like {}.", text);
}
```

对一系列整数进行 for 循环的示例：

```rust
let mut sum = 0;
for n in 1..11 {
    sum += n;
}
assert_eq!(sum, 55);
```

r[expr.loop.for.desugar]
`for` 循环等价于包含 [`match` 表达式][`match` expression]的 `loop` 表达式，如下所示：

<!-- ignore: expansion example -->
```rust,ignore
'label: for PATTERN in iter_expr {
    /* loop body */
}
```

等价于

<!-- ignore: expansion example -->
```rust,ignore
{
    let result = match IntoIterator::into_iter(iter_expr) {
        mut iter => 'label: loop {
            let mut next;
            match Iterator::next(&mut iter) {
                Option::Some(val) => next = val,
                Option::None => break,
            };
            let PATTERN = next;
            let () = { /* loop body */ };
        },
    };
    result
}
```

r[expr.loop.for.lang-items]
这里的 `IntoIterator`、`Iterator` 和 `Option` 始终是标准库中的项，而不是当前作用域中这些名称解析到的任何东西。

变量名 `next`、`iter` 和 `val` 仅用于说明，它们实际上没有用户可以输入的名字。

> [!NOTE]
> 外层的 `match` 用于确保 `iter_expr` 中的任何[临时值][temporary values]不会在循环完成之前被丢弃。`next` 在被赋值之前声明，因为这更经常地使类型被正确推断。

r[expr.loop.label]
## 循环标签

r[expr.loop.label.syntax]
```grammar,expressions
LoopLabel -> LIFETIME_OR_LABEL `:`
```

r[expr.loop.label.intro]
循环表达式可以可选地具有一个*标签*。标签写为循环表达式前的一个生命周期，如 `'foo: loop { break 'foo; }`、`'bar: while false {}`、`'humbug: for _ in 0..0 {}`。

r[expr.loop.label.control-flow]
如果存在标签，则嵌套在此循环中的带标签的 `break` 和 `continue` 表达式可以退出此循环或将控制返回到其头部。参见 [break 表达式](#break-expressions)和 [continue 表达式](#continue-expressions)。

r[expr.loop.label.ref]
标签遵循局部变量的卫生和遮蔽规则。例如，此代码将打印 "outer loop"：

```rust
'a: loop {
    'a: loop {
        break 'a;
    }
    print!("outer loop");
    break 'a;
}
```

`'_` 不是有效的循环标签。

r[expr.loop.break]
## `break` 表达式

r[expr.loop.break.syntax]
```grammar,expressions
BreakExpression -> `break` LIFETIME_OR_LABEL? Expression?
```

r[expr.loop.break.intro]
当遇到 `break` 时，关联循环体的执行立即终止，例如：

```rust
let mut last = 0;
for x in 1..100 {
    if x > 12 {
        break;
    }
    last = x;
}
assert_eq!(last, 12);
```

r[expr.loop.break.diverging]
`break` 表达式是[发散][diverging]的，具有类型 [`!`]。

r[expr.loop.break.label]
`break` 表达式通常与包围 `break` 表达式的最内层 `loop`、`for` 或 `while` 循环关联，但可以使用[标签](#loop-labels)来指定影响哪个外围循环。示例：

```rust
'outer: loop {
    while true {
        break 'outer;
    }
}
```

r[expr.loop.break.value]
`break` 表达式仅允许在循环体内部，并具有以下形式之一：`break`、`break 'label` 或（[见下文](#break-and-loop-values)）`break EXPR` 或 `break 'label EXPR`。

r[expr.loop.break-value.implicit-value]
在[带有 break 表达式的 `loop`][expr.loop.break-value] 或[带标签的块表达式][labeled block expression]中，不含表达式的 `break` 等价于 `break ()`。

r[expr.loop.block-labels]
## 带标签的块表达式

r[expr.loop.block-labels.syntax]
```grammar,expressions
LabelBlockExpression -> BlockExpression
```

r[expr.loop.block-labels.intro]
带标签的块表达式与块表达式完全相同，不同之处在于它们允许在块内使用 `break` 表达式。

r[expr.loop.block-labels.break]
与循环不同，带标签的块表达式中的 `break` 表达式*必须*具有标签（即标签不是可选的）。

r[expr.loop.block-labels.label-required]
类似地，带标签的块表达式*必须*以标签开头。

```rust
# fn do_thing() {}
# fn condition_not_met() -> bool { true }
# fn do_next_thing() {}
# fn do_last_thing() {}
let result = 'block: {
    do_thing();
    if condition_not_met() {
        break 'block 1;
    }
    do_next_thing();
    if condition_not_met() {
        break 'block 2;
    }
    do_last_thing();
    3
};
```

r[expr.loop.block-labels.type]
带标签的块表达式的类型是所有 break 操作数和最终操作数的[最小上界][least upper bound]。如果省略了最终操作数，则最终操作数的类型默认为[单元类型][unit type]，除非块[发散][expr.block.diverging]，在这种情况下它是[永不类型][never type]。

> [!EXAMPLE]
> ```rust
> fn example(condition: bool) {
>     let s = String::from("owned");
>
>     let _: &str = 'block: {
>         if condition {
>             break 'block &s;  // &String 通过 Deref 强制为 &str
>         }
>         break 'block "literal";  // &'static str 强制为 &str
>     };
> }
> ```

r[expr.loop.continue]
## `continue` 表达式

r[expr.loop.continue.syntax]
```grammar,expressions
ContinueExpression -> `continue` LIFETIME_OR_LABEL?
```

r[expr.loop.continue.intro]
当遇到 `continue` 时，关联循环体的当前迭代立即终止，将控制返回到循环*头部*。

r[expr.loop.continue.diverging]
`continue` 表达式是[发散][diverging]的，具有类型 [`!`]。

r[expr.loop.continue.while]
对于 `while` 循环，头部是控制循环的条件操作数。

r[expr.loop.continue.for]
对于 `for` 循环，头部是控制循环的调用表达式。

r[expr.loop.continue.label]
与 `break` 类似，`continue` 通常与最内层的外围循环关联，但可以使用 `continue 'label` 来指定受影响的循环。

r[expr.loop.continue.in-loop-only]
`continue` 表达式仅允许在循环体内部。

r[expr.loop.break-value]
## `break` 和循环值

r[expr.loop.break-value.intro]
当与 `loop` 关联时，可以使用 break 表达式从该循环返回值，通过 `break EXPR` 或 `break 'label EXPR` 形式，其中 `EXPR` 是其结果从 `loop` 返回的表达式。例如：

```rust
let (mut a, mut b) = (1, 1);
let result = loop {
    if b > 10 {
        break b;
    }
    let c = a + b;
    a = b;
    b = c;
};
// 斐波那契数列中第一个大于 10 的数：
assert_eq!(result, 13);
```

r[expr.loop.break-value.type]
具有关联 `break` 表达式的 `loop` 的类型是所有 break 操作数的[最小上界][least upper bound]。

> [!EXAMPLE]
> ```rust
> fn example(condition: bool) {
>     let s = String::from("owned");
>
>     let _: &str = loop {
>         if condition {
>             break &s; // &String 通过 Deref 强制为 &str
>         }
>         break "literal"; // &'static str 强制为 &str
>     };
> }
> ```

r[expr.loop.break-value.diverging]
如果所有 break 操作数中没有任何一个是不发散的，则具有关联 `break` 表达式的 `loop` 不[发散][diverge]。如果所有 `break` 操作数都发散，则 `loop` 表达式也发散。

> [!EXAMPLE]
> ```rust
> fn diverging_loop_with_break(condition: bool) -> ! {
>     // 此循环是发散的，因为所有 `break` 操作数都是发散的。
>     loop {
>         if condition {
>             break loop {};
>         } else {
>             break panic!();
>         }
>     }
> }
> ```
>
> ```rust,compile_fail,E0308
> fn loop_with_non_diverging_break(condition: bool) -> ! {
>     // 此循环的类型是 i32，即使其中一个 break 是发散的。
>     loop {
>         if condition {
>             break loop {};
>         } else {
>             break 123i32;
>         }
>     } // 错误：期望 `!`，找到 `i32`
> }
> ```

[`!`]: type.never
[`if` condition chains]: if-expr.md#chains-of-conditions
[`if` expressions]: if-expr.md
[`match` expression]: match-expr.md
[boolean type]: ../types/boolean.md
[diverge]: divergence
[diverging]: divergence
[labeled block expression]: expr.loop.block-labels
[least upper bound]: coerce.least-upper-bound
[never type]: type.never
[scrutinee]: ../glossary.md#scrutinee
[temporary values]: ../expressions.md#temporaries
[unit type]: type.tuple.unit
