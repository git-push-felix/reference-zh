r[attributes.limits]
# 限制

以下[属性][attributes]影响编译时限制。

r[attributes.limits.recursion_limit]
## `recursion_limit` 属性 {#the-recursion_limit-attribute}

r[attributes.limits.recursion_limit.intro]
*`recursion_limit` 属性*可以应用于 [crate] 级别，以设置可能无限递归的编译时操作（如宏展开或自动解引用）的最大深度。

r[attributes.limits.recursion_limit.syntax]
它使用 [MetaNameValueStr] 语法来指定递归深度。

> [!NOTE]
> `rustc` 中的默认值是 128。

```rust,compile_fail
#![recursion_limit = "4"]

macro_rules! a {
    () => { a!(1); };
    (1) => { a!(2); };
    (2) => { a!(3); };
    (3) => { a!(4); };
    (4) => { };
}

// 这无法展开，因为它需要大于 4 的递归深度。
a!{}
```

```rust,compile_fail
#![recursion_limit = "1"]

// 这失败了，因为自动解引用需要两个递归步骤。
(|_: &u8| {})(&&&1);
```

<!-- template:attributes -->
r[attributes.limits.type_length_limit]
## `type_length_limit` 属性 {#the-type_length_limit-attribute}

r[attributes.limits.type_length_limit.intro]
*`type_length_limit` [属性][attributes]* 设置在单态化期间构造具体类型时允许的最大类型替换次数。

> [!NOTE]
> `rustc` 仅在 nightly 的 `-Zenforce-type-length-limit` 标志激活时强制执行该限制。
>
> 更多信息请参见 [Rust PR #127670](https://github.com/rust-lang/rust/pull/127670)。

> [!EXAMPLE]
> <!-- ignore: not enforced without nightly flag -->
> ```rust,ignore
> #![type_length_limit = "4"]
>
> fn f<T>(x: T) {}
>
> // 这无法编译，因为单态化为
> // `f::<((((i32,), i32), i32), i32)>` 需要超过 4 个类型元素。
> f(((((1,), 2), 3), 4));
> ```

> [!NOTE]
> `rustc` 中的默认值是 `1048576`。

r[attributes.limits.type_length_limit.syntax]
`type_length_limit` 属性使用 [MetaNameValueStr] 语法。字符串中的值必须是非负数。

r[attributes.limits.type_length_limit.allowed-positions]
`type_length_limit` 属性只能应用于 crate 根。

> [!NOTE]
> `rustc` 忽略其他位置的用法但会发出 lint 警告。这可能在将来成为错误。

r[attributes.limits.type_length_limit.duplicates]
只有第一次在项上使用 `type_length_limit` 才有效。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。这可能在将来成为错误。

[attributes]: ../attributes.md
[crate]: ../crates-and-source-files.md
