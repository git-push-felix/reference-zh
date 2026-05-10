r[expr.await]
# Await 表达式

r[expr.await.syntax]
```grammar,expressions
AwaitExpression -> Expression `.` `await`
```

r[expr.await.intro]
`await` 表达式是一种语法结构，用于挂起由 `std::future::IntoFuture` 实现提供的计算，直到给定的 future 准备好产生一个值。

r[expr.await.construct]
await 表达式的语法是一个类型实现了 [`IntoFuture`] trait 的表达式（称为*future 操作数*），然后是 `.` 记号，再然后是 `await` 关键字。

r[expr.await.allowed-positions]
Await 表达式仅在[异步上下文][async context]（如 [`async fn`]、[`async` 闭包][`async` closure]或 [`async` 块][`async` block]）中合法。

r[expr.await.effects]
更具体地说，await 表达式具有以下效果。

1. 通过在 future 操作数上调用 [`IntoFuture::into_future`] 来创建 future。
2. 将该 future 求值为一个 [future] `tmp`；
3. 使用 [`Pin::new_unchecked`] 固定 `tmp`；
4. 然后通过调用 [`Future::poll`] 方法并将当前的[任务上下文](#task-context)传入来对此固定的 future 进行轮询；
5. 如果对 `poll` 的调用返回 [`Poll::Pending`]，则 future 返回 `Poll::Pending`，挂起其状态，以便当外围异步上下文被重新轮询时，执行回到第 3 步；
6. 否则对 `poll` 的调用必定返回了 [`Poll::Ready`]，在这种情况下，包含在 [`Poll::Ready`] 变体中的值被用作 `await` 表达式本身的结果。

r[expr.await.edition2018]
> [!EDITION-2018]
> Await 表达式仅从 Rust 2018 起可用。

r[expr.await.task]
## 任务上下文

任务上下文指的是当异步上下文本身被轮询时提供给当前[异步上下文][async context]的 [`Context`]。因为 `await` 表达式仅在异步上下文中合法，所以必须有某个任务上下文可用。

r[expr.await.desugar]
## 近似脱糖

实际上，await 表达式大致等价于以下非规范性脱糖：

<!-- ignore: example expansion -->
```rust,ignore
match operand.into_future() {
    mut pinned => loop {
        let mut pin = unsafe { Pin::new_unchecked(&mut pinned) };
        match Pin::future::poll(Pin::borrow(&mut pin), &mut current_context) {
            Poll::Ready(r) => break r,
            Poll::Pending => yield Poll::Pending,
        }
    }
}
```

其中 `yield` 伪代码返回 `Poll::Pending`，当被重新调用时，从该点恢复执行。变量 `current_context` 引用从异步环境中获取的上下文。

[`async fn`]: ../items/functions.md#async-functions
[`async` closure]: closure-expr.md#async-closures
[`async` block]: block-expr.md#async-blocks
[`Context`]: std::task::Context
[`future::poll`]: std::future::Future::poll
[`pin::new_unchecked`]: std::pin::Pin::new_unchecked
[`poll::Pending`]: std::task::Poll::Pending
[`poll::Ready`]: std::task::Poll::Ready
[async context]: ../expressions/block-expr.md#async-context
[future]: std::future::Future
[`IntoFuture`]: std::future::IntoFuture
[`IntoFuture::into_future`]: std::future::IntoFuture::into_future
