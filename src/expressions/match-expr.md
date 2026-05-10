r[expr.match]
# `match` 表达式

r[expr.match.syntax]
```grammar,expressions
MatchExpression ->
    `match` Scrutinee `{`
        InnerAttribute*
        MatchArms?
    `}`

Scrutinee -> Expression _except [StructExpression]_

MatchArms ->
    ( MatchArm `=>` ( ExpressionWithoutBlock `,` | ExpressionWithBlock `,`? ) )*
    MatchArm `=>` Expression `,`?

MatchArm -> OuterAttribute* Pattern MatchArmGuard?

MatchArmGuard -> `if` MatchConditions

MatchConditions ->
     MatchGuardChain
   | Expression

MatchGuardChain -> MatchGuardCondition ( `&&` MatchGuardCondition )*

MatchGuardCondition ->
     Expression _except [ExcludedMatchConditions]_
   | OuterAttribute* `let` Pattern `=` MatchGuardScrutinee

MatchGuardScrutinee -> Expression _except [ExcludedMatchConditions]_

@root ExcludedMatchConditions ->
      LazyBooleanExpression
    | RangeExpr
    | RangeFromExpr
    | RangeInclusiveExpr
    | AssignmentExpression
    | CompoundAssignmentExpression
```
<!-- TODO: 上面的例外不准确，参见 https://github.com/rust-lang/reference/issues/569 -->

r[expr.match.intro]
*`match` 表达式*基于模式进行分支。所发生的匹配的确切形式取决于[模式][pattern]。

r[expr.match.scrutinee]
`match` 表达式有一个*[受检者]表达式*，即要与模式进行比较的值。

r[expr.match.scrutinee-constraint]
受检者表达式和模式必须具有相同的类型。

r[expr.match.scrutinee-behavior]
`match` 的行为取决于受检者表达式是[位置表达式还是值表达式][place expression]。

r[expr.match.scrutinee-value]
如果受检者表达式是[值表达式]，它首先被求值到一个临时位置，然后结果值按顺序与分支中的模式进行比较，直到找到匹配。第一个具有匹配模式的分支被选为 `match` 的分支目标，模式绑定的任何变量被赋值给分支块中的局部变量，控制进入该块。

r[expr.match.scrutinee-place]
当受检者表达式是[位置表达式]时，`match` 不分配临时位置；然而，按值绑定可能会从内存位置复制或移动。如果可能，最好对位置表达式进行匹配，因为这类匹配的生命周期继承位置表达式的生命周期，而不是被限制在 match 内部。

`match` 表达式示例：

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    4 => println!("four"),
    5 => println!("five"),
    _ => println!("something else"),
}
```

r[expr.match.pattern-vars]
模式内绑定的变量的作用域是匹配守卫和分支表达式。

r[expr.match.pattern-var-binding]
[绑定模式]（移动、复制或引用）取决于模式。

r[expr.match.or-pattern]
可以使用 `|` 运算符连接多个匹配模式。每个模式将按从左到右的顺序进行测试，直到找到成功的匹配。

```rust
let x = 9;
let message = match x {
    0 | 1  => "not many",
    2 ..= 9 => "a few",
    _      => "lots"
};

assert_eq!(message, "a few");

// 模式匹配顺序的演示。
struct S(i32, i32);

match S(1, 2) {
    S(z @ 1, _) | S(_, z @ 2) => assert_eq!(z, 1),
    _ => panic!(),
}
```

> [!NOTE]
> `2..=9` 是一个[区间模式]，而不是[区间表达式]。因此，只有区间模式支持的那些类型的区间才能用于匹配分支中。

r[expr.match.or-patterns-restriction]
每个 `|` 分隔模式中的每个绑定都必须出现在该分支的所有模式中。

r[expr.match.binding-restriction]
每个同名的绑定必须具有相同的类型和相同的绑定模式。

r[expr.match.type]
整个 `match` 表达式的类型是各个匹配分支的[最小上界]。

r[expr.match.empty]
如果没有匹配分支，则 `match` 表达式是[发散][diverging]的，类型为 [`!`]。

> [!EXAMPLE]
> ```rust
> # fn make<T>() -> T { loop {} }
> enum Empty {}
>
> fn diverging_match_no_arms() -> ! {
>     let e: Empty = make();
>     match e {}
> }
> ```


r[expr.match.diverging]
如果受检者表达式或所有匹配分支发散，则整个 `match` 表达式也发散。

r[expr.match.guard]
## 匹配守卫

r[expr.match.guard.intro]
匹配分支可以接受*匹配守卫*以进一步细化匹配某个情况的条件。

r[expr.match.guard.condition]
模式守卫出现在 `if` 关键字之后的模式之后，由一个具有[布尔类型][type.bool]的[表达式][Expression]或一个条件 `let` 匹配组成。

r[expr.match.guard.behavior]
当模式成功匹配时，模式守卫被执行。如果所有守卫条件操作数都求值为 `true` 且所有 `let` 模式都成功匹配其[受检者]，则该匹配分支被成功匹配，并执行分支体。

r[expr.match.guard.next]
否则，测试下一个模式，包括同一分支中使用 `|` 运算符的其他匹配。

```rust
# let maybe_digit = Some(0);
# fn process_digit(i: i32) { }
# fn process_other(i: i32) { }
let message = match maybe_digit {
    Some(x) if x < 10 => process_digit(x),
    Some(x) => process_other(x),
    None => panic!(),
};
```

> [!NOTE]
> 使用 `|` 运算符的多重匹配可能导致模式守卫及其副作用被多次执行。例如：
>
> ```rust
> # use std::cell::Cell;
> let i : Cell<i32> = Cell::new(0);
> match 1 {
>     1 | _ if { i.set(i.get() + 1); false } => {}
>     _ => {}
> }
> assert_eq!(i.get(), 2);
> ```

r[expr.match.guard.bound-variables]
模式守卫可以引用在其后模式内绑定的变量。

r[expr.match.guard.shared-ref]
在求值守卫之前，会对受检者中变量匹配的部分获取共享引用。在求值守卫期间，访问变量时使用此共享引用。

r[expr.match.guard.value]
只有当守卫成功求值时，值才从受检者移动或复制到变量中。这允许在守卫内部使用共享借用，而如果在守卫未匹配的情况下不必从受检者中移出。

r[expr.match.guard.no-mutation]
此外，通过在求值守卫时持有共享引用，也防止了守卫内部的修改。

r[expr.match.guard.let]
守卫可以使用 `let` 模式来有条件地匹配受检者，并在模式成功匹配时将新变量绑定到作用域中。

> [!EXAMPLE]
> 在此示例中，守卫条件 `let Some(first_char) = name.chars().next()` 被求值。如果 `let` 模式成功匹配（即字符串至少有一个字符），则执行分支体。否则，模式匹配继续到下一个分支。
>
> `let` 模式创建一个新绑定（`first_char`），它可以在分支体中与原始模式绑定（`name`）一起使用。
> ```rust
> # enum Command {
> #     Run(String),
> #     Stop,
> # }
> let cmd = Command::Run("example".to_string());
>
> match cmd {
>     Command::Run(name) if let Some(first_char) = name.chars().next() => {
>         // 此处 `name` 和 `first_char` 都可用
>         println!("Running: {name} (starts with '{first_char}')");
>     }
>     Command::Run(name) => {
>         println!("{name} is empty");
>     }
>     _ => {}
> }
> ```

r[expr.match.guard.chains]
## 匹配守卫链

r[expr.match.guard.chains.intro]
多个守卫条件操作数可以用 `&&` 分隔。

> [!EXAMPLE]
> ```rust
> # let foo = Some([123]);
> # let already_checked = false;
> match foo {
>     Some(xs) if let [single] = xs && !already_checked => { dbg!(single); }
>     _ => {}
> }
> ```

r[expr.match.guard.chains.order]
类似于 `&&` [LazyBooleanExpression]，每个操作数从左到右求值，直到某个操作数求值为 `false` 或 `let` 匹配失败，在这种情况下后续操作数不会被求值。

r[expr.match.guard.chains.bindings]
每个 `let` 模式的绑定被放入作用域，以供下一个条件操作数和匹配分支体使用。

r[expr.match.guard.chains.or]
如果任何守卫条件操作数是 `let` 模式，则由于与 `let` 受检者的歧义和优先级，所有条件操作数都不能是 `||` [惰性布尔运算符表达式][expr.bool-logic]。

> [!EXAMPLE]
> 如果需要 `||` 表达式，可以使用括号。例如：
>
> ```rust
> # let foo = Some([123]);
> match foo {
>     // 此处需要括号。
>     Some(xs) if let [x] = xs && (x < -100 || x > 20) => {}
>     _ => {}
> }
> ```

r[expr.match.attributes]
## 匹配分支上的属性

r[expr.match.attributes.outer]
匹配分支上允许外部属性。在匹配分支上有意义的唯一属性是 [`cfg`] 和 [lint 检查属性]。

r[expr.match.attributes.inner]
在匹配表达式开花括号后直接允许[内部属性]，其所在表达式上下文与[块表达式上的属性]相同。

[`!`]: type.never
[`cfg`]: ../conditional-compilation.md
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
[binding mode]: ../patterns.md#binding-modes
[diverging]: divergence
[Inner attributes]: ../attributes.md
[least upper bound]: coerce.least-upper-bound
[lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[pattern]: ../patterns.md
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[Range Expression]: range-expr.md
[Range Pattern]: ../patterns.md#range-patterns
[scrutinee]: ../glossary.md#scrutinee
[value expression]: ../expressions.md#place-expressions-and-value-expressions
