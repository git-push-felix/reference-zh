r[type.never]
# Never 类型

r[type.never.syntax]
```grammar,types
NeverType -> `!`
```

r[type.never.intro]
never 类型 `!` 是一种没有值的类型，表示永远不会完成的求值结果。

r[type.never.coercion]
类型为 `!` 的表达式可以被强制转换为任何其他类型。

r[type.never.constraint]
`!` 类型目前**只能**出现在函数返回类型中，表示它是一个永不返回的发散函数。

```rust
fn foo() -> ! {
    panic!("此调用永远不会返回。");
}
```

```rust
unsafe extern "C" {
    pub safe fn no_return_extern_func() -> !;
}
```
