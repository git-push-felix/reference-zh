r[expr.block]
# 块表达式

r[expr.block.syntax]
```grammar,expressions
BlockExpression ->
    `{`
        InnerAttribute*
        Statements?
    `}`

Statements ->
      Statement+
    | Statement+ ExpressionWithoutBlock
    | ExpressionWithoutBlock
```

r[expr.block.intro]
*块表达式*（或称*块*）是一种控制流表达式，也是项和变量声明的匿名命名空间作用域。

r[expr.block.sequential-evaluation]
作为控制流表达式，块按顺序执行其组成的非项声明语句，然后是其可选的最终表达式。

r[expr.block.namespace]
作为匿名命名空间作用域，项声明仅在块自身内部有效，由 `let` 语句声明的变量从下一条语句开始直到块结束都有效。更多细节参见[作用域][scopes]章节。

r[expr.block.inner-attributes]
块的语法是 `{`，然后是一些[内部属性][inner attributes]，然后是一些[语句][statements]，然后是一个可选的表达式（称为最终操作数），最后是 `}`。

r[expr.block.statements]
语句通常要求后跟分号，有两个例外：

1. 项声明语句不需要后跟分号。
2. 表达式语句通常要求后跟分号，除非其外围表达式是控制流表达式。

r[expr.block.null-statement]
此外，语句之间允许额外的分号，但这些分号不影响语义。

r[expr.block.evaluation]
在求值块表达式时，每条语句（除了项声明语句）按顺序执行。

r[expr.block.result]
然后执行最终操作数（如果有给出的话）。

r[expr.block.value-trailing-expr]
当块包含[最终操作数][final operand]时，块具有该最终操作数的类型和值。

```rust
let x: u8 = { 0u8 }; // `0u8` 是最终操作数。
assert_eq!(x, 0);
let x: u8 = { (); 0u8 }; // 同上。
assert_eq!(x, 0);
```

r[expr.block.value-no-trailing-expr]
当块不包含[最终操作数][final operand]且块不发散时，块具有[单元类型][unit type]和[单元值][unit value]。

```rust
let x: () = {}; // 没有最终操作数。
assert_eq!(x, ());
let x: () = { 0u8; }; // 同上。
assert_eq!(x, ());
```

r[expr.block.value-diverges-no-trailing-expr]
当块不包含[最终操作数][final operand]且块[发散][diverges]时，块具有[永不类型][never type]且没有最终值（因为其类型是[无人居住][uninhabited]的）。

```rust,no_run
fn f() -> ! { loop {}; } // 发散且没有最终操作数。
//          ^^^^^^^^^^^^
// 函数体是块表达式。
```

> [!NOTE]
> 注意，块没有最终操作数与有显式的单元类型最终操作数是不同的。例如，即使此块发散，块的类型也是[单元][unit]而非[永不][never]。
>
> ```rust,compile_fail,E0308
> fn f() -> ! { loop {}; () } // 错误：类型不匹配。
> //          ^^^^^^^^^^^^^^^ 此块具有单元类型。
> ```

> [!NOTE]
> 作为控制流表达式，如果块表达式是表达式语句的外围表达式，则期望类型为 `()`，除非其后紧跟分号。

r[expr.block.diverging]
如果一个块的所有可达控制流路径都包含一个发散表达式，则该块被认为是[发散的][divergence]，除非该表达式是未被读取的[位置表达式][place expression]。

```rust,no_run
# #![ feature(never_type) ]
fn no_control_flow() -> ! {
    // 没有条件语句，因此整个函数体是发散的。
    loop {}
}

fn control_flow_diverging() -> ! {
    // 所有路径都是发散的，因此整个函数体是发散的。
    if true {
        loop {}
    } else {
        loop {}
    }
}

fn control_flow_not_diverging() -> () {
    // 某些路径不是发散的，因此整个块不是发散的。
    if true {
        ()
    } else {
        loop {}
    }
}

// 注意：这里使用了不稳定的 never 类型，该类型仅在
// Rust 的 nightly 通道上可用。这只是为了说明目的。
// 在稳定版 Rust 中也可能遇到这种情况，但需要更
// 复杂的示例。
struct Foo {
    x: !,
}

fn make<T>() -> T { loop {} }

fn diverging_place_read() -> ! {
    let foo = Foo { x: make() };
    // 读取位置表达式产生一个发散块。
    let _x = foo.x;
}
```

```rust,compile_fail,E0308
# #![ feature(never_type) ]
# fn make<T>() -> T { loop {} }
# struct Foo {
#     x: !,
# }
fn diverging_place_not_read() -> ! {
    let foo = Foo { x: make() };
    // 赋值给 `_` 意味着该位置未被读取。
    let _ = foo.x;
} // 错误：类型不匹配。
```

r[expr.block.value]
块始终是[值表达式][value expressions]，并在值表达式上下文中求值最后一个操作数。

> [!NOTE]
> 这可以用于在确实需要时强制移动值。例如，下面的示例在调用 `consume_self` 时失败，因为结构体已在块表达式中从 `s` 移出。
>
> ```rust,compile_fail
> struct Struct;
>
> impl Struct {
>     fn consume_self(self) {}
>     fn borrow_self(&self) {}
> }
>
> fn move_by_block_expression() {
>     let s = Struct;
>
>     // 在块表达式中将值从 `s` 移出。
>     (&{ s }).borrow_self();
>
>     // 因为 `s` 已被移出，所以执行失败。
>     s.consume_self();
> }
> ```

r[expr.block.async]
## `async` 块 {#async-blocks}

r[expr.block.async.syntax]
```grammar,expressions
AsyncBlockExpression -> `async` `move`? BlockExpression
```

r[expr.block.async.intro]
*async 块*是块表达式的一种变体，它求值为一个 future。

r[expr.block.async.future-result]
块的最终表达式（如果有的话）决定 future 的结果值。

r[expr.block.async.anonymous-type]
执行 async 块类似于执行闭包表达式：其直接效果是产生并返回一个匿名类型。

r[expr.block.async.future]
然而，闭包返回一个实现 [`std::ops::Fn`] trait 族之一的类型，而 async 块返回的类型实现 [`std::future::Future`] trait。

r[expr.block.async.layout-unspecified]
此类型的具体数据格式未作规定。

> [!NOTE]
> rustc 生成的 future 类型大致等价于一个枚举，每个 `await` 点对应一个变体，每个变体存储从其对应点恢复所需的数据。

r[expr.block.async.edition2018]
> [!EDITION-2018]
> Async 块仅从 Rust 2018 起可用。

r[expr.block.async.capture]
### 捕获模式

Async 块使用与闭包相同的[捕获模式][capture modes]从环境中捕获变量。与闭包类似，当写为 `async { .. }` 时，每个变量的捕获模式将根据块的内容推断。而 `async move { .. }` 块则会将所有引用的变量移动到生成的 future 中。

r[expr.block.async.context]
### 异步上下文 {#async-context}

因为 async 块构造一个 future，它们定义了一个**异步上下文**，其中又可以包含 [`await` 表达式][`await` expressions]。异步上下文由 async 块以及异步函数体建立，异步函数的语义是通过 async 块来定义的。

r[expr.block.async.function]
### 控制流运算符

r[expr.block.async.function.intro]
Async 块像函数边界一样运作，很像闭包。

r[expr.block.async.function.return-try]
因此，`?` 运算符和 `return` 表达式都影响 future 的输出，而不是外围函数或其他上下文。也就是说，async 块内的 `return <expr>` 将返回 `<expr>` 的结果作为 future 的输出。类似地，如果 `<expr>?` 传播错误，该错误将作为 future 的结果传播。

r[expr.block.async.function.control-flow]
最后，`break` 和 `continue` 关键字不能用于从 async 块中跳出。因此以下代码是非法的：

```rust,compile_fail
loop {
    async move {
        break; // 错误[E0267]：`async` 块内的 `break`
    }
}
```

r[expr.block.const]
## `const` 块 {#const-blocks}

r[expr.block.const.syntax]
```grammar,expressions
ConstBlockExpression -> `const` BlockExpression
```

r[expr.block.const.intro]
*const 块*是块表达式的一种变体，其主体在编译时求值而不是在运行时求值。

r[expr.block.const.context]
Const 块允许你定义常量值而无需定义新的[常量项][constant items]，因此有时也称为*内联常量*。它还支持类型推断，因此无需像[常量项][constant items]那样指定类型。

r[expr.block.const.generic-params]
与[自由项][free item]常量项不同，Const 块能够引用作用域内的泛型参数。它们被脱糖为作用域内有泛型参数的常量项（类似于关联常量，但没有与之关联的 trait 或类型）。例如，以下代码：

```rust
fn foo<T>() -> usize {
    const { std::mem::size_of::<T>() + 1 }
}
```

等价于：

```rust
fn foo<T>() -> usize {
    {
        struct Const<T>(T);
        impl<T> Const<T> {
            const CONST: usize = std::mem::size_of::<T>() + 1;
        }
        Const::<T>::CONST
    }
}
```

r[expr.block.const.evaluation]

如果 const 块表达式在运行时被执行，则常量保证被求值，即使其返回值被忽略：

```rust
fn foo<T>() -> usize {
    // 如果此代码曾经被执行，则断言肯定已在编译时被求值。
    const { assert!(std::mem::size_of::<T>() > 0); }
    // 此处我们可以有依赖于类型非零大小的 unsafe 代码。
    /* ... */
    42
}
```

r[expr.block.const.not-executed]

如果 const 块表达式不在运行时被执行，它可能被求值也可能不被求值：
```rust,compile_fail
if false {
    // 当程序构建时，panic 可能发生也可能不发生。
    const { panic!(); }
}
```

r[expr.block.unsafe]
## `unsafe` 块 {#unsafe-blocks}

r[expr.block.unsafe.syntax]
```grammar,expressions
UnsafeBlockExpression -> `unsafe` BlockExpression
```

r[expr.block.unsafe.intro]
*有关何时使用 `unsafe` 的更多信息，请参见 [`unsafe` 块][`unsafe` blocks]。*

代码块可以用 `unsafe` 关键字作为前缀，以允许[不安全操作][unsafe operations]。示例：

```rust
unsafe {
    let b = [13u8, 17u8];
    let a = &b[0] as *const u8;
    assert_eq!(*a, 13);
    assert_eq!(*a.offset(1), 17);
}

# unsafe fn an_unsafe_fn() -> i32 { 10 }
let a = unsafe { an_unsafe_fn() };
```

r[expr.block.label]
## 带标签的块表达式 {#labeled-block-expressions}

带标签的块表达式在[循环和其他可中断表达式][Loops and other breakable expressions]一节中描述。

r[expr.block.attributes]
## 块表达式上的属性 {#attributes-on-block-expressions}

r[expr.block.attributes.inner-attributes]
在以下情况下，允许在块表达式的开花括号后直接放置[内部属性][inner attributes]：

* [函数][Function]和[方法][method]体。
* 循环体（[`loop`]、[`while`] 和 [`for`]）。
* 用作[语句][statement]的块表达式。
* 作为[数组表达式][array expressions]、[元组表达式][tuple expressions]、[调用表达式][call expressions]和元组式[结构体][struct]表达式元素的块表达式。
* 作为另一个块表达式的尾部表达式的块表达式。
<!-- Keep list in sync with expressions.md -->

r[expr.block.attributes.valid]
在块表达式上有意义的属性是 [`cfg`] 和 [lint 检查属性][the lint check attributes]。

例如，此函数在 unix 平台上返回 `true`，在其他平台上返回 `false`。

```rust
fn is_unix_platform() -> bool {
    #[cfg(unix)] { true }
    #[cfg(not(unix))] { false }
}
```

[`await` expressions]: await-expr.md
[`cfg`]: ../conditional-compilation.md
[`for`]: loop-expr.md#iterator-loops
[`loop`]: loop-expr.md#infinite-loops
[`unsafe` blocks]: ../unsafe-keyword.md#unsafe-blocks-unsafe-
[`while`]: loop-expr.md#predicate-loops
[array expressions]: array-expr.md
[call expressions]: call-expr.md
[capture modes]: ../types/closure.md#capture-modes
[constant items]: ../items/constant-items.md
[diverges]: expr.block.diverging
[final operand]: expr.block.inner-attributes
[free item]: ../glossary.md#free-item
[function]: ../items/functions.md
[inner attributes]: ../attributes.md
[method]: ../items/associated-items.md#methods
[mutable reference]: ../types/pointer.md#mutables-references-
[never type]: type.never
[never]: type.never
[place expression]: expr.place-value.place-memory-location
[scopes]: ../names/scopes.md
[shared references]: ../types/pointer.md#shared-references-
[statement]: ../statements.md
[statements]: ../statements.md
[struct]: struct-expr.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[tuple expressions]: tuple-expr.md
[uninhabited]: glossary.uninhabited
[unit type]: type.tuple.unit
[unit value]: type.tuple.unit
[unit]: type.tuple.unit
[unsafe operations]: ../unsafety.md
[value expressions]: ../expressions.md#place-expressions-and-value-expressions
[Loops and other breakable expressions]: expr.loop.block-labels
