r[expr.return]
# `return` 表达式

r[expr.return.syntax]
```grammar,expressions
ReturnExpression -> `return` Expression?
```

r[expr.return.intro]
`return` 表达式用关键字 `return` 表示。

r[expr.return.behavior]
求值 `return` 表达式将其参数移动到当前函数调用的指定输出位置，销毁当前函数激活帧，并将控制转移到调用者帧。

r[expr.return.diverging]
`return` 表达式是[发散][diverging]的，具有类型 [`!`]。

`return` 表达式示例：

```rust
fn max(a: i32, b: i32) -> i32 {
    if a > b {
        return a;
    }
    return b;
}
```

[`!`]: type.never
[diverging]: divergence
