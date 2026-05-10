r[expr.closure]
# 闭包表达式

r[expr.closure.syntax]
```grammar,expressions
ClosureExpression ->
    `async`?[^cl-async-edition]
    `move`?
    ( `||` | `|` ClosureParameters? `|` )
    (Expression | `->` TypeNoBounds BlockExpression)

ClosureParameters -> ClosureParam (`,` ClosureParam)* `,`?

ClosureParam -> OuterAttribute* PatternNoTopAlt ( `:` Type )?
```

[^cl-async-edition]: `async` 限定符在 2015 版本中不允许使用。

r[expr.closure.intro]
*闭包表达式*（也称为 lambda 表达式或 lambda）定义一个[闭包类型][closure type]，并求值为该类型的一个值。闭包表达式的语法是可选的 `async` 关键字、可选的 `move` 关键字，然后是一组用管道符号（`|`）括起来的、以逗号分隔的[模式][patterns]列表（称为*闭包参数*，每个参数后面可选地跟有 `:` 和类型），然后是可选的 `->` 和类型（称为*返回类型*），然后是一个表达式（称为*闭包体操作数*）。

r[expr.closure.param-type]
每个模式后的可选类型是该模式的类型标注。

r[expr.closure.explicit-type-body]
如果有返回类型，闭包体必须是一个[块][block]。

r[expr.closure.parameter-restriction]
闭包表达式表示一个将参数列表映射到跟随参数的表达式的函数。与 [`let` 绑定][`let` binding]一样，闭包参数是不可反驳的[模式][patterns]，其类型标注是可选的，如果未给出将从上下文中推断。

r[expr.closure.unique-type]
每个闭包表达式都有一个唯一的匿名类型。

r[expr.closure.captures]
值得注意的是，闭包表达式*捕获其环境*，而常规[函数定义][function definitions]则不会。

r[expr.closure.capture-inference]
没有 `move` 关键字时，闭包表达式[推断如何从环境中捕获每个变量](../types/closure.md#capture-modes)，优先通过共享引用捕获，有效地借用闭包体内提到的所有外部变量。

r[expr.closure.capture-mut-ref]
如果需要，编译器将推断应该改为使用可变引用，或者应该从环境中移动或复制值（取决于它们的类型）。

r[expr.closure.capture-move]
可以通过在闭包前加上 `move` 关键字来强制闭包通过复制或移走值来捕获其环境。这通常用于确保闭包的生命周期为 `'static`。

r[expr.closure.trait-impl]
## 闭包 trait 实现

闭包类型实现哪些 trait 取决于变量的捕获方式、被捕获变量的类型以及 `async` 的存在。关于闭包如何以及何时实现 `Fn`、`FnMut` 和 `FnOnce`，请参见[调用 trait 和强制][call traits and coercions]章节。如果每个被捕获变量的类型也实现了该 trait，则闭包类型也实现 [`Send`] 和 [`Sync`]。

r[expr.closure.async]
## Async 闭包

r[expr.closure.async.intro]
标记有 `async` 关键字的闭包表示它们是异步的，其方式类似于[异步函数][items.fn.async]。

r[expr.closure.async.future]
调用 async 闭包不执行任何工作，而是求值为一个实现 [`Future`] 的值，该值对应于闭包体的计算。

```rust
async fn takes_async_callback(f: impl AsyncFn(u64)) {
    f(0).await;
    f(1).await;
}

async fn example() {
    takes_async_callback(async |i| {
        core::future::ready(i).await;
        println!("done with {i}.");
    }).await;
}
```

r[expr.closure.async.edition2018]
> [!EDITION-2018]
> Async 闭包仅从 Rust 2018 起可用。

## 示例

在此示例中，我们定义了一个接受高阶函数参数的函数 `ten_times`，然后用一个闭包表达式作为参数调用它，随后又用一个从环境中移走值的闭包表达式调用它。

```rust
fn ten_times<F>(f: F) where F: Fn(i32) {
    for index in 0..10 {
        f(index);
    }
}

ten_times(|j| println!("hello, {}", j));
// 带有类型标注
ten_times(|j: i32| -> () { println!("hello, {}", j) });

let word = "konnichiwa".to_owned();
ten_times(move |j| println!("{}, {}", word, j));
```

## 闭包参数上的属性

r[expr.closure.param-attributes]
闭包参数上的属性遵循与[常规函数参数][regular function parameters]相同的规则和限制。

[`let` binding]: ../statements.md#let-statements
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[block]: block-expr.md
[call traits and coercions]: ../types/closure.md#call-traits-and-coercions
[closure type]: ../types/closure.md
[function definitions]: ../items/functions.md
[patterns]: ../patterns.md
[regular function parameters]: ../items/functions.md#attributes-on-function-parameters
