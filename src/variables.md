r[variable]
# 变量

r[variable.intro]
*变量*是栈帧的一个组成部分，可以是具名的函数参数、匿名的[临时值](expressions.md#temporaries)，或具名的局部变量。

r[variable.local]
*局部变量*（或称*栈局部*分配）直接持有一个值，分配在栈内存中。该值是栈帧的一部分。

r[variable.local-mut]
局部变量默认是不可变的，除非另行声明。例如：`let mut x = ...`。

r[variable.param-mut]
函数参数默认是不可变的，除非用 `mut` 声明。`mut` 关键字仅作用于紧随其后的那个参数。例如：`|mut x, y|` 和 `fn f(mut x: Box<i32>, y: Box<i32>)` 声明了一个可变变量 `x` 和一个不可变变量 `y`。

r[variable.init]
局部变量在分配时并不初始化。相反，整个栈帧的全部局部变量在进入栈帧时以未初始化状态一次性分配。函数内的后续语句可以初始化这些局部变量，也可以不初始化。局部变量只有在通过所有可达的控制流路径被初始化后才能使用。

在下面的示例中，`init_after_if` 在 [`if` 表达式][`if` expression]之后被初始化，而 `uninit_after_if` 则没有，因为在 `else` 分支中它没有被初始化。

```rust
# fn random_bool() -> bool { true }
fn initialization_example() {
    let init_after_if: ();
    let uninit_after_if: ();

    if random_bool() {
        init_after_if = ();
        uninit_after_if = ();
    } else {
        init_after_if = ();
    }

    init_after_if; // ok
    // uninit_after_if; // err: use of possibly uninitialized `uninit_after_if`
}
```

[`if` expression]: expressions/if-expr.md#if-expressions
