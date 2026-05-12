r[coerce]
# 类型自动强转

r[coerce.intro]
**类型自动强转**是更改值的类型的隐式操作。它们在特定位置自动发生，并且对实际强转的类型有高度限制。

r[coerce.as]
自动强转允许的任何转换也可以通过[类型转换运算符][type cast operator] `as` 显式执行。

自动强转最初在 [RFC 401] 中定义，并在 [RFC 1558] 中扩展。

r[coerce.site]
## 强转位置 {#coercion-sites}

r[coerce.site.intro]
自动强转只能在程序的特定强转位置发生；这些通常是期望的类型是显式的或可以通过显式类型推导出来的位置（不需要类型推断）。可能的强转位置包括：

r[coerce.site.let]
* 给出了显式类型的 `let` 语句。

   例如，在以下代码中 `&mut 42` 被强转为类型 `&i8`：

   ```rust
   let _: &i8 = &mut 42;
   ```

r[coerce.site.value]
* `static` 和 `const` 项声明（类似于 `let` 语句）。

r[coerce.site.argument]
* 函数调用的参数

  被强转的值是实际参数，它被强转为形式参数的类型。

  例如，在以下代码中 `&mut 42` 被强转为类型 `&i8`：

  ```rust
  fn bar(_: &i8) { }

  fn main() {
      bar(&mut 42);
  }
  ```

  对于方法调用，接收者（`self` 参数）类型的强转方式不同，详情请参阅[方法调用表达式][method-call expressions]的文档。

r[coerce.site.constructor]
* 结构体、联合体或枚举变体字段的实例化

  例如，在以下代码中 `&mut 42` 被强转为类型 `&i8`：

  ```rust
  struct Foo<'a> { x: &'a i8 }

  fn main() {
      Foo { x: &mut 42 };
  }
  ```

r[coerce.site.return]
* 函数结果------要么是块的最后一行（如果没有用分号结束），要么是 `return` 语句中的任何表达式

  例如，在以下代码中 `x` 被强转为类型 `&dyn Display`：

  ```rust
  use std::fmt::Display;
  fn foo(x: &u32) -> &dyn Display {
      x
  }
  ```

r[coerce.site.assignment]
* 赋值表达式中被赋值的操作数

  例如，在以下代码中 `y` 被强转为类型 `&i8`：
  ```rust
  let mut x = &0i8;
  let y = &mut 42i8;
  x = y;
  ```

r[coerce.site.subexpr]
如果这些强转位置之一的表达式是强转传播表达式，则该表达式中的相关子表达式也是强转位置。从这些新的强转位置开始递归传播。传播表达式及其相关子表达式包括：

r[coerce.site.array]
* 数组字面量，其中数组的类型为 `[U; n]`。数组字面量中的每个子表达式都是到类型 `U` 的强转位置。

r[coerce.site.repeat]
* 带有重复语法的数组字面量，其中数组的类型为 `[U; n]`。重复的子表达式是到类型 `U` 的强转位置。

r[coerce.site.tuple]
* 元组，其中元组是到类型 `(U_0, U_1, ..., U_n)` 的强转位置。每个子表达式是到各自类型的强转位置，例如第零个子表达式是到类型 `U_0` 的强转位置。

r[coerce.site.parenthesis]
* 括号子表达式 (`(e)`)：如果表达式的类型为 `U`，则子表达式是到 `U` 的强转位置。

r[coerce.site.block]
* 块：如果块的类型为 `U`，则块中的最后一个表达式（如果没有用分号结束）是到 `U` 的强转位置。这包括作为控制流语句一部分的块，如 `if`/`else`，如果该块具有已知类型的话。

r[coerce.types]
## 强转类型 {#coercion-types}

r[coerce.types.intro]
允许在以下类型之间进行自动强转：

r[coerce.types.reflexive]
* `T` 到 `U`，如果 `T` 是 `U` 的[子类型][subtype]（*自反情况*）

r[coerce.types.transitive]
* `T_1` 到 `T_3`，其中 `T_1` 可以强转为 `T_2`，且 `T_2` 可以强转为 `T_3`（*传递情况*）

    注意，这尚未完全支持。

r[coerce.types.mut-reborrow]
* `&mut T` 到 `&T`

r[coerce.types.mut-pointer]
* `*mut T` 到 `*const T`

r[coerce.types.ref-to-pointer]
* `&T` 到 `*const T`

r[coerce.types.mut-to-pointer]
* `&mut T` 到 `*mut T`

r[coerce.types.deref]
* `&T` 或 `&mut T` 到 `&U`，如果 `T` 实现了 `Deref<Target = U>`。例如：

  ```rust
  use std::ops::Deref;

  struct CharContainer {
      value: char,
  }

  impl Deref for CharContainer {
      type Target = char;

      fn deref<'a>(&'a self) -> &'a char {
          &self.value
      }
  }

  fn foo(arg: &char) {}

  fn main() {
      let x = &mut CharContainer { value: 'y' };
      foo(x); // &mut CharContainer 被强转为 &char。
  }
  ```

r[coerce.types.deref-mut]
* `&mut T` 到 `&mut U`，如果 `T` 实现了 `DerefMut<Target = U>`。

r[coerce.types.unsize]
* TyCtor(`T`) 到 TyCtor(`U`)，其中 TyCtor(`T`) 是以下之一
    - `&T`
    - `&mut T`
    - `*const T`
    - `*mut T`
    - `Box<T>`

    并且 `U` 可以通过[非固定大小强转](#unsized-coercions)由 `T` 获得。

    <!--未来，coerce_inner 将递归扩展到元组和结构体。此外，从子 trait 到超 trait 的强转也将被添加。更多细节请参见 [RFC 401]。-->

r[coerce.types.fn]
* 函数项类型到 `fn` 指针

r[coerce.types.closure]
* 非捕获闭包到 `fn` 指针

r[coerce.types.never]
* `!` 到任何 `T`

r[coerce.unsize]
### 非固定大小强转 {#unsized-coercions}

r[coerce.unsize.intro]
以下强转称为*非固定大小强转*，因为它们涉及将类型转换为非固定大小类型（unsized types），并且在上文描述的其他强转不被允许的少数情况下也是被允许的。它们仍然可以在任何允许强转的地方发生。

r[coerce.unsize.trait]
两个 trait，[`Unsize`] 和 [`CoerceUnsized`]，用于辅助此过程并在库使用中暴露它。以下强转是内置的，如果 `T` 可以通过其中之一强转为 `U`，则将提供 `T` 对 `Unsize<U>` 的实现：

r[coerce.unsize.slice]
* `[T; n]` 到 `[T]`。

r[coerce.unsize.trait-object]
* `T` 到 `dyn U`，当 `T` 实现 `U + Sized`，且 `U` 是 [dyn 兼容][dyn compatible]的。

r[coerce.unsize.trait-upcast]
* `dyn T` 到 `dyn U`，当 `U` 是 `T` 的[超 trait][supertraits]之一时。
    * 这允许丢弃 auto trait，即 `dyn T + Auto` 到 `dyn U` 是允许的。
    * 如果主 trait 具有 auto trait 作为超 trait，这允许添加 auto trait，即给定 `trait T: U + Send {}`，则允许 `dyn T` 到 `dyn T + Send` 或到 `dyn U + Send` 的强转。

r[coerce.unsize.composite]
* `Foo<..., T, ...>` 到 `Foo<..., U, ...>`，当：
    * `Foo` 是一个结构体。
    * `T` 实现了 `Unsize<U>`。
    * `Foo` 的最后一个字段具有涉及 `T` 的类型。
    * 如果该字段的类型为 `Bar<T>`，则 `Bar<T>` 实现了 `Unsize<Bar<U>>`。
    * T 不是任何其他字段类型的一部分。

r[coerce.unsize.pointer]
此外，当 `T` 实现 `Unsize<U>` 或 `CoerceUnsized<Foo<U>>` 时，类型 `Foo<T>` 可以实现 `CoerceUnsized<Foo<U>>`。这允许其提供到 `Foo<U>` 的非固定大小强转。

> [!NOTE]
> 虽然非固定大小强转的定义及其实现已经稳定，但这些 trait 本身尚未稳定，因此不能在稳定的 Rust 中直接使用。

r[coerce.least-upper-bound]
## 最小上界强转

r[coerce.least-upper-bound.intro]
在某些上下文中，编译器必须将多个类型一起强转以尝试找到最通用的类型。这称为"最小上界"（Least Upper Bound）强转。LUB 强转仅用于以下情况：

+ 为一系列 if 分支找到公共类型。
+ 为一系列 match 分支找到公共类型。
+ 为数组元素找到公共类型。
+ 为[带标签块表达式][labeled block expression]在 break 操作数和最终块操作数之间找到公共类型。
+ 为[带有 break 表达式的 `loop` 表达式][`loop` expression with break expressions]在 break 操作数之间找到公共类型。
+ 为具有多个 return 语句的闭包找到返回类型。
+ 检查具有多个 return 语句的函数的返回类型。

r[coerce.least-upper-bound.target]
在每种情况下，有一组类型 `T0..Tn` 需要相互强转到某个目标类型 `T_t`，该目标类型开始时是未知的。

r[coerce.least-upper-bound.computation]
LUB 强转的计算是迭代进行的。目标类型 `T_t` 从类型 `T0` 开始。对于每个新类型 `Ti`，我们考虑：

r[coerce.least-upper-bound.computation-identity]
+ 如果 `Ti` 可以强转到当前目标类型 `T_t`，则不做更改。

r[coerce.least-upper-bound.computation-replace]
+ 否则，检查 `T_t` 是否可以强转到 `Ti`；如果可以，则 `T_t` 被更改为 `Ti`。（此检查还取决于到目前为止考虑的所有源表达式是否具有隐式强转。）

r[coerce.least-upper-bound.computation-unify]
+ 如果不能，则尝试计算 `T_t` 和 `Ti` 的公共超类型，该类型将成为新的目标类型。

### 示例：

```rust
# let (a, b, c) = (0, 1, 2);
// 对于 if 分支
let bar = if true {
    a
} else if false {
    b
} else {
    c
};

// 对于 match 分支
let baw = match 42 {
    0 => a,
    1 => b,
    _ => c,
};

// 对于数组元素
let bax = [a, b, c];

// 对于具有多个 return 语句的闭包
let clo = || {
    if true {
        a
    } else if false {
        b
    } else {
        c
    }
};
let baz = clo();

// 对于具有多个 return 语句的函数的类型检查
fn foo() -> i32 {
    let (a, b, c) = (0, 1, 2);
    match 42 {
        0 => a,
        1 => b,
        _ => c,
    }
}
```

在这些示例中，`ba*` 的类型是通过 LUB 强转找到的。编译器在处理函数 `foo` 时检查 `a`、`b`、`c` 的 LUB 强转结果是否为 `i32`。

### 注意事项

此描述显然是非形式化的。使其更精确的工作预期将作为更精确地规范化 Rust 类型检查器的一般努力的一部分进行。

[RFC 401]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md
[RFC 1558]: https://github.com/rust-lang/rfcs/blob/master/text/1558-closure-to-fn-coercion.md
[subtype]: subtyping.md
[dyn compatible]: items/traits.md#dyn-compatibility
[type cast operator]: expressions/operator-expr.md#type-cast-expressions
[`Unsize`]: std::marker::Unsize
[`CoerceUnsized`]: std::ops::CoerceUnsized
[labeled block expression]: expr.loop.block-labels
[`loop` expression with break expressions]: expr.loop.break-value
[method-call expressions]: expressions/method-call-expr.md
[supertraits]: items/traits.md#supertraits
