r[destructors]
# 析构器

r[destructors.intro]
当[已初始化][initialized]&#32;[变量][variable]或[临时值][temporary]离开[作用域](#drop-scopes)时，其*析构器*会被运行，或者说它被*丢弃*。[赋值][Assignment]也会运行其左操作数的析构器（如果它已被初始化）。如果变量已被部分初始化，则仅丢弃其已初始化的字段。

r[destructors.operation]
类型 `T` 的析构器包括：

1. 如果 `T: Drop`，则调用 [`<T as core::ops::Drop>::drop`](core::ops::Drop::drop)
2. 递归地运行其所有字段的析构器。
    * [结构体][struct]的字段按声明顺序丢弃。
    * 活动[枚举变体][enum variant]的字段按声明顺序丢弃。
    * [元组][tuple]的字段按顺序丢弃。
    * [数组][array]或拥有所有权的[切片][slice]的元素从第一个元素到最后一个按顺序丢弃。
    * [闭包][closure]通过 move 捕获的变量按未指定顺序丢弃。
    * [Trait 对象][Trait objects]运行底层类型的析构器。
    * 其他类型不会导致任何进一步的丢弃。

r[destructors.drop_in_place]
如果必须手动运行析构器（例如在实现你自己的智能指针时），可以使用 [`core::ptr::drop_in_place`]。

一些示例：

```rust
struct PrintOnDrop(&'static str);

impl Drop for PrintOnDrop {
    fn drop(&mut self) {
        println!("{}", self.0);
    }
}

let mut overwritten = PrintOnDrop("drops when overwritten");
overwritten = PrintOnDrop("drops when scope ends");

let tuple = (PrintOnDrop("Tuple first"), PrintOnDrop("Tuple second"));

let moved;
// 赋值时不会运行析构器。
moved = PrintOnDrop("Drops when moved");
// 现在丢弃，但之后变为未初始化状态。
moved;

// 未初始化的值不会丢弃。
let uninitialized: PrintOnDrop;

// 部分移动后，只有剩余字段被丢弃。
let mut partial_move = (PrintOnDrop("first"), PrintOnDrop("forgotten"));
// 执行部分移动，仅保留 `partial_move.0` 初始化状态。
core::mem::forget(partial_move.1);
// 当 partial_move 的作用域结束时，只有第一个字段被丢弃。
```

r[destructors.scope]
## 丢弃作用域 {#drop-scopes}

r[destructors.scope.intro]
每个变量或临时值都关联到一个*丢弃作用域*。当控制流离开丢弃作用域时，与该作用域关联的所有变量按声明（对于变量）或创建（对于临时值）的逆序被丢弃。

r[destructors.scope.desugaring]
丢弃作用域可以通过将 [`for`]、[`if`] 和 [`while`] 表达式替换为使用 [`match`]、[`loop`] 和 `break` 的等效表达式来确定。

r[destructors.scope.operators]
重载运算符与内置运算符不做区分，且不考虑[绑定模式][binding modes]。

r[destructors.scope.list]
对于函数或闭包，存在以下丢弃作用域：

r[destructors.scope.function]
* 整个函数

r[destructors.scope.statement]
* 每个[语句][statement]

r[destructors.scope.expression]
* 每个[表达式][expression]

r[destructors.scope.block]
* 每个块，包括函数体
    * 对于[块表达式][block expression]，该块的作用域和该表达式的作用域是同一个作用域。

r[destructors.scope.match-arm]
* `match` 表达式的每个分支

r[destructors.scope.nesting]
丢弃作用域按如下方式嵌套。当同时离开多个作用域时（例如从函数返回时），变量从内到外被丢弃。

r[destructors.scope.nesting.function]
* 整个函数作用域是最外层作用域。

r[destructors.scope.nesting.function-body]
* 函数体块包含在整个函数的作用域中。

r[destructors.scope.nesting.expr-statement]
* 表达式语句中的表达式的父作用域是该语句的作用域。

r[destructors.scope.nesting.let-initializer]
* [`let` 语句][`let` statement]的初始化器的父作用域是该 `let` 语句的作用域。

r[destructors.scope.nesting.statement]
* 语句作用域的父作用域是包含该语句的块的作用域。

r[destructors.scope.nesting.match-guard]
* `match` 守卫的表达式的父作用域是该守卫所在分支的作用域。

r[destructors.scope.nesting.match-arm]
* `match` 表达式中 `=>` 之后的表达式的父作用域是它所在分支的作用域。

r[destructors.scope.nesting.match]
* 分支作用域的父作用域是该分支所属的 `match` 表达式的作用域。

r[destructors.scope.nesting.other]
* 所有其他作用域的父作用域是其直接包含的表达式的的作用域。

r[destructors.scope.params]
### 函数参数的作用域

所有函数参数都在整个函数体的作用域中，因此在计算函数时最后丢弃。每个实际函数参数在该参数模式中引入的绑定之后被丢弃。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
// 丢弃 `y`，然后丢弃第二个参数，然后丢弃 `x`，然后丢弃第一个参数
fn patterns_in_parameters(
    (x, _): (PrintOnDrop, PrintOnDrop),
    (_, y): (PrintOnDrop, PrintOnDrop),
) {}

// 丢弃顺序为 3 2 0 1
patterns_in_parameters(
    (PrintOnDrop("0"), PrintOnDrop("1")),
    (PrintOnDrop("2"), PrintOnDrop("3")),
);
```

r[destructors.scope.bindings]
### 局部变量的作用域

r[destructors.scope.bindings.let]
在 `let` 语句中声明的局部变量关联到包含该 `let` 语句的块的作用域。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let declared_first = PrintOnDrop("Dropped last in outer scope");
{
    let declared_in_block = PrintOnDrop("Dropped in inner scope");
}
let declared_last = PrintOnDrop("Dropped first in outer scope");
```

r[destructors.scope.bindings.match-arm]
在 `match` 表达式或模式匹配的 `match` 守卫中声明的局部变量关联到它们所在 `match` 分支的分支作用域。

```rust
# #![allow(irrefutable_let_patterns)]
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
match PrintOnDrop("Dropped last in the first arm's scope") {
    // 当守卫求值成功时，控制流保留在分支中，
    // 值可以从被检查值移动到分支的绑定中，
    // 导致它们在分支作用域中被丢弃。
    x if let y = PrintOnDrop("Dropped second in the first arm's scope")
        && let z = PrintOnDrop("Dropped first in the first arm's scope") =>
    {
        let declared_in_block = PrintOnDrop("Dropped in inner scope");
        // 模式匹配守卫的绑定和临时值按逆序丢弃，
        // 在丢弃每个守卫条件操作数的临时值之前先丢弃其绑定。
        // 最后，丢弃由分支模式绑定的变量。
    }
    _ => unreachable!(),
}

match PrintOnDrop("Dropped in the enclosing temporary scope") {
    // 当守卫求值失败时，控制流离开分支作用域，
    // 导致更早的模式匹配守卫条件操作数的绑定和临时值被丢弃。
    // 这发生在求值下一个分支的守卫或主体之前。
    _ if let y = PrintOnDrop("Dropped in the first arm's scope")
        && false => unreachable!(),
    // 当由于自重叠或模式导致守卫多次执行时，
    // 控制流在守卫失败时离开分支作用域，
    // 并在再次执行守卫之前重新进入分支作用域。
    _ | _ if let y = PrintOnDrop("Dropped in the second arm's scope twice")
        && false => unreachable!(),
    _ => {},
}
```

r[destructors.scope.bindings.patterns]
模式中的变量按模式内声明的逆序丢弃。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let (declared_first, declared_last) = (
    PrintOnDrop("Dropped last"),
    PrintOnDrop("Dropped first"),
);
```

r[destructors.scope.bindings.or-patterns]
对于丢弃顺序，[或模式][or-patterns]按第一个子模式中给出的顺序声明绑定。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
// 在 `y` 之前丢弃 `x`。
fn or_pattern_drop_order<T>(
    (Ok([x, y]) | Err([y, x])): Result<[T; 2], [T; 2]>
//   ^^^^^^^^^^   ^^^^^^^^^^^ 这是第二个子模式。
//   |
//   这是第一个子模式。
//
//   在第一个子模式中，`x` 在 `y` 之前声明。由于它是
//   第一个子模式，即使匹配到绑定顺序相反的
//   第二个子模式时，也使用该顺序。
) {}

// 这里我们匹配第一个子模式，丢弃顺序按照
// 第一个子模式中的声明顺序。
or_pattern_drop_order(Ok([
    PrintOnDrop("Declared first, dropped last"),
    PrintOnDrop("Declared last, dropped first"),
]));

// 这里我们匹配第二个子模式，丢弃顺序仍然
// 按照第一个子模式中的声明顺序。
or_pattern_drop_order(Err([
    PrintOnDrop("Declared last, dropped first"),
    PrintOnDrop("Declared first, dropped last"),
]));
```

r[destructors.scope.temporary]
### 临时值作用域 {#temporary-scopes}

r[destructors.scope.temporary.intro]
表达式的*临时值作用域*是用于在[位置上下文][place context]中使用该表达式时保存该表达式结果的临时变量的作用域，除非它被[提升][promoted]。

r[destructors.scope.temporary.enclosing]
除了生命周期延长的情况外，表达式的临时值作用域是包含该表达式且符合以下条件之一的最小作用域：

* 整个函数。
* 一个语句。
* 一个 [`if`]、[`while`] 或 [`loop`] 表达式的主体。
* 一个 `if` 表达式的 `else` 块。
* `if` 或 `while` 表达式中非模式匹配的条件表达式，或非模式匹配的 `match` [守卫条件操作数][guard condition operand]。
* `match` 分支的模式匹配守卫（如果存在）和主体表达式。
* [惰性布尔表达式][lazy boolean expression]的每个操作数。
* [`if`] 的模式匹配条件和随之的主体（[destructors.scope.temporary.edition2024]）。
* [`while`] 的模式匹配条件和循环体。
* 块的尾部表达式的整体（[destructors.scope.temporary.edition2024]）。

> [!NOTE]
> `match` 表达式的[被检查值][scrutinee]不是一个临时值作用域，因此被检查值中的临时值可以在 `match` 表达式之后被丢弃。例如，在 `match 1 { ref mut z => z };` 中，`1` 的临时值存活到语句结束。

> [!NOTE]
> [解构赋值][destructuring assignment]的脱糖限制了其所赋值操作数（RHS）的临时值作用域。详情请参见 [expr.assign.destructure.tmp-scopes]。

r[destructors.scope.temporary.edition2024]
> [!EDITION-2024]
> 2024 版添加了两条新的临时值作用域收窄规则：`if let` 的临时值在 `else` 块之前丢弃，块尾部表达式的临时值在尾部表达式求值后立即丢弃。

一些示例：

```rust
# #![allow(irrefutable_let_patterns)]
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let local_var = PrintOnDrop("local var");

// 条件求值后立即丢弃
if PrintOnDrop("If condition").0 == "If condition" {
    // 在块末尾丢弃
    PrintOnDrop("If body").0
} else {
    unreachable!()
};

if let "if let scrutinee" = PrintOnDrop("if let scrutinee").0 {
    PrintOnDrop("if let consequent").0
    // `if let consequent` 在这里丢弃
}
// `if let scrutinee` 在这里丢弃
else {
    PrintOnDrop("if let else").0
    // `if let else` 在这里丢弃
};

while let x = PrintOnDrop("while let scrutinee").0 {
    PrintOnDrop("while let loop body").0;
    break;
    // `while let loop body` 在这里丢弃。
    // `while let scrutinee` 在这里丢弃。
}

// 在第一个 || 之前丢弃
(PrintOnDrop("first operand").0 == ""
// 在 ) 之前丢弃
|| PrintOnDrop("second operand").0 == "")
// 在分号之前丢弃
|| PrintOnDrop("third operand").0 == "";

// 被检查值在函数末尾丢弃，早于局部变量
// （因为这是函数体块的尾部表达式）。
match PrintOnDrop("Matched value in final expression") {
    // 非模式匹配守卫的临时值在条件求值后丢弃
    _ if PrintOnDrop("guard condition").0 == "" => (),
    // 模式匹配守卫的临时值在离开分支作用域时丢弃
    _ if let "guard scrutinee" = PrintOnDrop("guard scrutinee").0 => {
        let _ = &PrintOnDrop("lifetime-extended temporary in inner scope");
        // `lifetime-extended temporary in inner scope` 在这里丢弃
    }
    // `guard scrutinee` 在这里丢弃
    _ => (),
}
```

r[destructors.scope.operands]
### 操作数

临时值也会被创建以在计算其他操作数时保存表达式操作数的结果。这些临时值关联到具有该操作数的表达式的作用域。由于一旦表达式求值完成，这些临时值就被移走，因此丢弃它们没有效果，除非某个表达式操作数中断、返回或 [panic] 导致了提前退出。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
loop {
    // 元组表达式未完成求值，因此操作数以逆序丢弃
    (
        PrintOnDrop("Outer tuple first"),
        PrintOnDrop("Outer tuple second"),
        (
            PrintOnDrop("Inner tuple first"),
            PrintOnDrop("Inner tuple second"),
            break,
        ),
        PrintOnDrop("Never created"),
    );
}
```

r[destructors.scope.const-promotion]
### 常量提升 {#constant-promotion}

当值表达式可以写为常量并被借用，且该借用可以在原本书写表达式的位置被解引用而不改变运行时行为时，该表达式可以被提升到 `'static` 槽位。也就是说，提升后的表达式可以在编译时求值，其结果值不包含[内部可变性][interior mutability]或[析构器][destructors]（这些属性在可能的情况下根据值确定，例如 `&None` 始终具有类型 `&'static Option<_>`，因为它不包含任何禁止的内容）。

r[destructors.scope.lifetime-extension]
### 临时值生命周期延长 {#temporary-lifetime-extension}

> [!NOTE]
> 临时值生命周期延长的确切规则可能会变更。这里仅描述当前行为。

r[destructors.scope.lifetime-extension.let]
`let` 语句中表达式的临时值作用域有时被*延长*到包含该 `let` 语句的块的作用域。当通常的临时值作用域太小时，基于某些语法规则会执行此延长操作。例如：

```rust
let x = &mut 0;
// 通常临时值现在已经丢弃，但 `0` 的临时值存活到块的末尾。
println!("{}", x);
```

r[destructors.scope.lifetime-extension.static]
生命周期延长也适用于 `static` 和 `const` 项，使临时值存活到程序结束。例如：

```rust
const C: &Vec<i32> = &Vec::new();
// 通常这会是一个悬垂引用，因为 `Vec` 只存在于
// `C` 的初始化器表达式中，但借用的生命周期被延长，
// 因此它实际上具有 `'static` 生命周期。
println!("{:?}", C);
```

r[destructors.scope.lifetime-extension.sub-expressions]
如果[借用][borrow]、[解引用][dereference expression]、[字段][field expression]或[元组索引表达式][tuple indexing expression]具有延长的临时值作用域，则其操作数也具有延长的临时值作用域。如果[索引表达式][indexing expression]具有延长的临时值作用域，则被索引的表达式也具有延长的临时值作用域。

r[destructors.scope.lifetime-extension.patterns]
#### 基于模式的延长

r[destructors.scope.lifetime-extension.patterns.extending]
*扩展模式*是以下之一：

* 通过引用或可变引用进行绑定的[标识符模式][identifier pattern]。

  ```rust
  # fn temp() {}
  let ref x = temp(); // 通过引用绑定。
  # x;
  let ref mut x = temp(); // 通过可变引用绑定。
  # x;
  ```

* 其中至少一个直接子模式是扩展模式的[结构体][struct pattern]、[元组][tuple pattern]、[元组结构体][tuple struct pattern]、[切片][slice pattern]或[或模式][or-patterns]。

  ```rust
  # use core::sync::atomic::{AtomicU64, Ordering::Relaxed};
  # static X: AtomicU64 = AtomicU64::new(0);
  struct W<T>(T);
  # impl<T> Drop for W<T> { fn drop(&mut self) { X.fetch_add(1, Relaxed); } }
  let W { 0: ref x } = W(()); // 结构体模式。
  # x;
  let W(ref x) = W(()); // 元组结构体模式。
  # x;
  let (W(ref x),) = (W(()),); // 元组模式。
  # x;
  let [W(ref x), ..] = [W(())]; // 切片模式。
  # x;
  let (Ok(W(ref x)) | Err(&ref x)) = Ok(W(())); // 或模式。
  # x;
  //
  // 以上所有临时值在这里仍然存活。
  # assert_eq!(0, X.load(Relaxed));
  ```

因此 `ref x`、`V(ref x)` 和 `[ref x, y]` 都是扩展模式，但 `x`、`&ref x` 和 `&(ref x,)` 不是。

r[destructors.scope.lifetime-extension.patterns.let]
如果 `let` 语句中的模式是扩展模式，则初始化器表达式的临时值作用域被延长。

```rust
# fn temp() {}
// 这是一个扩展模式，因此临时值作用域被延长。
let ref x = *&temp(); // OK
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
// 这既不是扩展模式也不是扩展表达式，
// 因此临时值在分号处被丢弃。
let &ref x = *&&temp(); // 错误
# x;
```

```rust
# fn temp() {}
// 这不是扩展模式，但它是扩展表达式，
// 因此临时值存活到 `let` 语句之后。
let &ref x = &*&temp(); // OK
# x;
```

r[destructors.scope.lifetime-extension.exprs]
#### 基于表达式的延长 {#extending-based-on-expressions}

r[destructors.scope.lifetime-extension.exprs.extending]
对于带有初始化器的 let 语句，*扩展表达式*是以下之一的表达式：

* 初始化器表达式。
* 扩展[借用][borrow]表达式的操作数。
* 扩展[超级宏调用][super macro call]表达式的[超级操作数][super operands]。
* 扩展[数组][array expression]、[cast][cast expression]、[花括号结构体][struct expression]或[元组][tuple expression]表达式的操作数。
* 扩展[元组结构体][tuple struct]或[元组枚举变体][tuple enum variant]构造器表达式的参数。
* 扩展[块表达式][block expression]（[异步块表达式][async block expression]除外）的最终表达式。
* 扩展 [`if`] 表达式的后续分支、`else if` 或 `else` 块的最终表达式。
* 扩展 [`match`] 表达式的一个分支表达式。

> [!NOTE]
> [解构赋值][destructuring assignment]的脱糖使其所赋值操作数（RHS）成为新引入块中的扩展表达式。详情请参见 [expr.assign.destructure.tmp-ext]。

因此 `&mut 0`、`(&1, &mut 2)` 和 `Some(&mut 3)` 中的借用表达式都是扩展表达式。`&0 + &1` 和 `f(&mut 0)` 中的借用则不是。

r[destructors.scope.lifetime-extension.exprs.borrows]
扩展[借用][borrow]表达式的操作数的[临时值作用域][temporary scope]被[延长][extended]。

r[destructors.scope.lifetime-extension.exprs.super-macros]
扩展[超级宏调用][super macro call]表达式的[超级操作数][super temporaries]的[作用域][temporary scopes]被[延长][extended]。

> [!NOTE]
> `rustc` 不将扩展[数组][array]表达式的[数组重复操作数][array repeat operands]视为扩展表达式。是否应该这样处理是一个开放性问题。
>
> 详情请参见 [Rust issue #146092](https://github.com/rust-lang/rust/issues/146092)。

#### 示例

以下是表达式具有延长临时值作用域的一些示例：

```rust,edition2024
# use core::pin::pin;
# use core::sync::atomic::{AtomicU64, Ordering::Relaxed};
# static X: AtomicU64 = AtomicU64::new(0);
# #[derive(Debug)] struct S;
# impl Drop for S { fn drop(&mut self) { X.fetch_add(1, Relaxed); } }
# const fn temp() -> S { S }
let x = &temp(); // 借用的操作数。
# x;
let x = &raw const *&temp(); // 裸借用的操作数。
# assert_eq!(X.load(Relaxed), 0);
let x = &temp() as &dyn Send; // cast 的操作数。
# x;
let x = (&*&temp(),); // 元组构造器的操作数。
# x;
struct W<T>(T);
let x = W(&temp()); // 元组结构体构造器的参数。
# x;
let x = Some(&temp()); // 元组枚举变体构造器的参数。
# x;
let x = { [Some(&temp())] }; // 块的最终表达式。
# x;
let x = const { &temp() }; // `const` 块的最终表达式。
# x;
let x = unsafe { &temp() }; // `unsafe` 块的最终表达式。
# x;
let x = if true { &temp() } else { &temp() };
//              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//           `if`/`else` 块的最终表达式。
# x;
let x = match () { _ => &temp() }; // `match` 分支表达式。
# x;
let x = pin!(temp()); // 超级宏调用表达式的超级操作数。
# x;
let x = pin!({ &mut temp() }); // 同上。
# x;
let x = format_args!("{:?}", temp()); // 同上。
# x;
//
// 以上所有临时值在这里仍然存活。
# assert_eq!(0, X.load(Relaxed));
```

以下是没有延长临时值作用域的表达式的一些示例：

```rust,compile_fail,E0716
# fn temp() {}
// 函数调用的参数不是扩展表达式。临时值在分号处丢弃。
let x = core::convert::identity(&temp()); // 错误
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
# trait Use { fn use_temp(&self) -> &Self { self } }
# impl Use for () {}
// 方法调用的接收者不是扩展表达式。
let x = (&temp()).use_temp(); // 错误
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
// match 表达式的被检查值不是扩展表达式。
let x = match &temp() { x => x }; // 错误
# x;
```

```rust,compile_fail,E0515
# fn temp() {}
// `async` 块的最终表达式不是扩展表达式。
let x = async { &temp() }; // 错误
# x;
```

```rust,compile_fail,E0515
# fn temp() {}
// 闭包的最终表达式不是扩展表达式。
let x = || &temp(); // 错误
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
// 循环 break 的操作数不是扩展表达式。
let x = loop { break &temp() }; // 错误
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
// break 到标签的操作数不是扩展表达式。
let x = 'a: { break 'a &temp() }; // 错误
# x;
```

```rust,edition2024,compile_fail,E0716
# use core::pin::pin;
# fn temp() {}
// `pin!` 的参数仅在调用是扩展表达式时才被作为扩展表达式。
// 由于它不是，因此内部块不是扩展表达式，所以其尾部
// 表达式中的临时值被立即丢弃。
pin!({ &temp() }); // 错误
```

```rust,edition2024,compile_fail,E0716
# fn temp() {}
// 同上。
format_args!("{:?}", { &temp() }); // 错误
```

r[destructors.forget]
## 不运行析构器

r[destructors.manually-suppressing]
### 手动阻止析构器

[`core::mem::forget`] 可用于阻止变量的析构器运行，[`core::mem::ManuallyDrop`] 提供了一个包装器来防止变量或字段被自动丢弃。

> [!NOTE]
> 通过 [`core::mem::forget`] 或其他方式阻止析构器运行是安全的，即使变量的类型不是 `'static`。除了本文档定义的保证运行析构器的地方之外，类型*不能*安全地依赖析构器的运行来保证健全性。

r[destructors.process-termination]
### 不展开的进程终止

有一些终止进程的方式不会进行[展开][unwinding]，在这种情况下析构器不会运行。

标准库提供了 [`std::process::exit`] 和 [`std::process::abort`] 来显式执行此操作。此外，如果 [panic 处理器][panic.panic_handler.std]被设置为 `abort`，则 panic 将始终终止进程而不运行析构器。

还有一个需要知晓的额外情况：当 panic 到达[不可展开的 ABI 边界][non-unwinding ABI boundary]时，要么没有析构器运行，要么直到 ABI 边界的所有析构器都运行。

[Assignment]: expressions/operator-expr.md#assignment-expressions
[binding modes]: patterns.md#binding-modes
[closure]: types/closure.md
[destructors]: destructors.md
[destructuring assignment]: expr.assign.destructure
[expression]: expressions.md
[guard condition operand]: expressions/match-expr.md#match-guard-chains
[identifier pattern]: patterns.md#identifier-patterns
[initialized]: glossary.md#initialized
[interior mutability]: interior-mutability.md
[lazy boolean expression]: expressions/operator-expr.md#lazy-boolean-operators
[non-unwinding ABI boundary]: items/functions.md#unwinding
[panic]: panic.md
[place context]: expressions.md#place-expressions-and-value-expressions
[promoted]: destructors.md#constant-promotion
[scrutinee]: glossary.md#scrutinee
[statement]: statements.md
[temporary]: expressions.md#temporaries
[unwinding]: panic.md#unwinding
[variable]: variables.md

[array]: types/array.md
[enum variant]: types/enum.md
[slice]: types/slice.md
[struct]: types/struct.md
[Trait objects]: types/trait-object.md
[tuple]: types/tuple.md

[or-patterns]: patterns.md#or-patterns
[slice pattern]: patterns.md#slice-patterns
[struct pattern]: patterns.md#struct-patterns
[tuple pattern]: patterns.md#tuple-patterns
[tuple struct pattern]: patterns.md#tuple-struct-patterns
[tuple struct]: type.struct.tuple
[tuple enum variant]: type.enum.declaration

[array expression]: expressions/array-expr.md#array-expressions
[array repeat operands]: expr.array.repeat-operand
[async block expression]: expr.block.async
[block expression]: expressions/block-expr.md
[borrow]: expr.operator.borrow
[cast expression]: expressions/operator-expr.md#type-cast-expressions
[dereference expression]: expressions/operator-expr.md#the-dereference-operator
[extended]: destructors.scope.lifetime-extension
[field expression]: expressions/field-expr.md
[indexing expression]: expressions/array-expr.md#array-and-slice-indexing-expressions
[struct expression]: expressions/struct-expr.md
[super macro call]: expr.super-macros
[super operands]: expr.super-macros
[super temporaries]: expr.super-macros
[temporary scope]: destructors.scope.temporary
[temporary scopes]: destructors.scope.temporary
[tuple expression]: expressions/tuple-expr.md#tuple-expressions
[tuple indexing expression]: expressions/tuple-expr.md#tuple-indexing-expressions

[`for`]: expressions/loop-expr.md#iterator-loops
[`if let`]: expressions/if-expr.md#if-let-patterns
[`if`]: expressions/if-expr.md#if-expressions
[`let` statement]: statements.md#let-statements
[`loop`]: expressions/loop-expr.md#infinite-loops
[`match`]: expressions/match-expr.md
[`while let`]: expressions/loop-expr.md#while-let-patterns
[`while`]: expressions/loop-expr.md#predicate-loops
