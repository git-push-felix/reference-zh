r[divergence]
# 发散

r[divergence.intro]
*发散表达式*是永远不会完成正常执行的表达式。

```rust
fn diverges() -> ! {
    panic!("This function never returns!");
}

fn example() {
    let x: i32 = diverges(); // 这一行永远不会完成。
    println!("This is never printed: {x}");
}
```

有关特定表达式发散行为的规则，请参见：

- [expr.block.diverging] --- 块表达式。
- [expr.if.diverging] --- `if` 表达式。
- [expr.loop.block-labels.type] --- 带 `break` 的带标签块表达式。
- [expr.loop.break-value.diverging] --- 带 `break` 的 `loop` 表达式。
- [expr.loop.break.diverging] --- `break` 表达式。
- [expr.loop.continue.diverging] --- `continue` 表达式。
- [expr.loop.infinite.diverging] --- 无限 `loop` 表达式。
- [expr.match.diverging] --- `match` 表达式。
- [expr.match.empty] --- 空 `match` 表达式。
- [expr.return.diverging] --- `return` 表达式。
- [type.never.constraint] --- 返回 `!` 的函数调用。

> [!NOTE]
> [`panic!`] 宏以及相关的生成 panic 的宏（如 [`unreachable!`]）也具有类型 [`!`] 并且是发散的。

r[divergence.never]
任何类型为 [`!`] 的表达式都是发散表达式。然而，发散表达式不限于类型 [`!`]；其他类型的表达式也可能发散（例如，`Some(loop {})` 的类型是 `Option<!>`）。

> [!NOTE]
> 尽管 `!` 被视为无人居住类型，但类型是无人居住的并不足以使其发散。
>
> ```rust,compile_fail,E0308
> enum Empty {}
> fn make_never() -> ! {loop{}}
> fn make_empty() -> Empty {loop{}}
>
> fn diverging() -> ! {
>     // 此处的类型为 `!`。
>     // 因此，整个函数被认为是发散的。
>     make_never();
>     // 正确：函数体的类型是 `!`，与返回类型匹配。
> }
> fn not_diverging() -> ! {
>     // 此类型是无人居住的。
>     // 然而，整个函数不被认为是发散的。
>     make_empty();
>     // 错误：函数体的类型是 `()`，但期望的类型是 `!`。
> }
> ```

> [!NOTE]
> 发散可以传播到外围的块。请参见 [expr.block.diverging]。

r[divergence.fallback]
## 回退

如果待推断的类型仅与发散表达式统一，那么该类型将被推断为 [`!`]。

> [!EXAMPLE]
> ```rust,compile_fail,E0277
> fn foo() -> i32 { 22 }
> match foo() {
>     // 错误：trait 约束 `!: Default` 未满足。
>     4 => Default::default(),
>     _ => return,
> };
> ```

> [!EDITION-2024]
> 在 2024 版次之前，该类型被推断为 `()`。

> [!NOTE]
> 重要的是，类型统一可能以*结构化*方式发生，因此回退到的 `!` 可能是更大类型的一部分。以下代码可以编译：
>
> ```rust
> fn foo() -> i32 { 22 }
> // 此处的类型为 `Option<!>`，而非 `!`
> match foo() {
>     4 => Default::default(),
>     _ => Some(return),
> };
> ```

<!-- TODO: 这最后一点可能应该移到一个更通用的"类型推断"章节，讨论泛化和统一。 -->

[`!`]: type.never
