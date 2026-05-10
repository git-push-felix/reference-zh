r[const-eval]
# 常量求值

r[const-eval.intro]
常量求值是在编译期间计算[表达式][expressions]结果的过程。只有所有表达式的一个子集可以在编译时求值。

r[const-eval.const-expr]
## 常量表达式

r[const-eval.const-expr.intro]
某些形式的表达式（称为常量表达式）可以在编译时求值。

r[const-eval.const-expr.const-context]
[const 上下文][const context]中的表达式必须是常量表达式。

r[const-eval.const-expr.evaluation]
const 上下文中的表达式始终在编译时求值。

r[const-eval.const-expr.runtime-context]
在 const 上下文之外，常量表达式*可能*在编译时求值，但不保证。

r[const-eval.const-expr.error]
如果值必须在编译时求值（即在 const 上下文中），诸如越界[数组索引][array indexing]或[溢出][overflow]等行为是编译器错误。否则，这些行为是警告，但在运行时可能会导致 panic。

r[const-eval.const-expr.list]
以下表达式是常量表达式，只要任何操作数也是常量表达式，并且不会导致运行任何 [`Drop::drop`][destructors] 调用。

r[const-eval.const-expr.literal]
* [字面量][Literals]。

r[const-eval.const-expr.parameter]
* [常量参数][Const parameters]。

r[const-eval.const-expr.path-item]
* 指向[函数][functions]和[常量][constants]的[路径][Paths]。不允许递归定义常量。

r[const-eval.const-expr.path-static]
* 指向[静态项][statics]的路径，有以下限制：
  * 在任何常量求值上下文中不允许写入 `static` 项。
  * 在任何常量求值上下文中不允许读取 `extern` 静态项。
  * 如果求值*不*在 `static` 项的初始化器中进行，则不允许读取任何可变 `static`。可变 `static` 是 `static mut` 项，或具有内部可变类型的 `static` 项。

  这些要求仅在常量被求值时检查。换句话说，在 const 上下文中语法上存在此类访问是允许的，只要它们从未被执行。

r[const-eval.const-expr.tuple]
* [元组表达式][Tuple expressions]。

r[const-eval.const-expr.array]
* [数组表达式][Array expressions]。

r[const-eval.const-expr.constructor]
* [结构体表达式][Struct expressions]。

r[const-eval.const-expr.block]
* [块表达式][Block expressions]，包括 `unsafe` 和 `const` 块。
    * [let 语句][let statements]及不可反驳的[模式][patterns]，包括可变绑定
    * [赋值表达式][assignment expressions]
    * [复合赋值表达式][compound assignment expressions]
    * [表达式语句][expression statements]

r[const-eval.const-expr.field]
* [字段表达式][Field expressions]。

r[const-eval.const-expr.index]
* [数组和切片索引表达式][array indexing]，其中索引是 `usize`。

r[const-eval.const-expr.range]
* [范围表达式][Range expressions]。

r[const-eval.const-expr.closure]
* 不从环境中捕获变量的[闭包表达式][Closure expressions]。

r[const-eval.const-expr.builtin-arith-logic]
* 用于整数和浮点类型、`bool` 和 `char` 的内置[取反][negation]、[算术][arithmetic]、[逻辑][logical]、[比较][comparison]或[惰性布尔][lazy boolean]运算符。

r[const-eval.const-expr.borrows]
* 所有形式的[借用][borrow]，包括原始借用，除了那些临时作用域会被延长到程序结束（见[临时生命周期延长][temporary lifetime extension]）的表达式的借用，并且这些表达式要么是：
  * 可变借用。
  * 对产生具有[内部可变性][interior mutability]值的表达式的共享借用。

  ```rust,compile_fail,E0764
  // 由于处于尾部位置，此借用将临时值的作用域延长到程序结束。
  // 由于借用是可变的，这在 const 表达式中不允许。
  const C: &u8 = &mut 0; // 错误，不允许
  ```

  ```rust,compile_fail,E0764
  // Const 块类似于 `const` 项的初始化器。
  let _: &u8 = const { &mut 0 }; // 错误，不允许
  ```

  ```rust,compile_fail,E0492
  # use core::sync::atomic::AtomicU8;
  // 这是不允许的，因为 1) 临时作用域被延长到程序结束，
  // 且 2) 临时值具有内部可变性。
  const C: &AtomicU8 = &AtomicU8::new(0); // 错误，不允许
  ```

  ```rust,compile_fail,E0492
  # use core::sync::atomic::AtomicU8;
  // 同上。
  let _: &_ = const { &AtomicU8::new(0) }; // 错误，不允许
  ```

  ```rust
  # #![allow(static_mut_refs)]
  // 即使此借用是可变的，它也不是临时值的借用，因此允许。
  const C: &u8 = unsafe { static mut S: u8 = 0; &mut S }; // 正确
  ```

  ```rust
  # use core::sync::atomic::AtomicU8;
  // 即使此借用是对具有内部可变性值的借用，
  // 它不是临时值的借用，因此允许。
  const C: &AtomicU8 = {
      static S: AtomicU8 = AtomicU8::new(0); &S // 正确
  };
  ```

  ```rust
  # use core::sync::atomic::AtomicU8;
  // 此对内部可变临时值的共享借用是允许的，
  // 因为其作用域未被延长。
  const C: () = { _ = &AtomicU8::new(0); }; // 正确
  ```

  ```rust
  // 即使借用是可变的且临时值由于提升而存活到程序结束，这也是允许的，
  // 因为借用不在尾部位置，因此临时值的作用域
  // 不会通过临时生命周期延长而扩展。
  const C: () = { let _: &'static mut [u8] = &mut []; }; // 正确
  //                                              ~~
  //                                     已提升的临时值。
  ```

  > [!NOTE]
  > 换句话说------关注允许什么而不是不允许什么------对内部可变数据的共享借用和可变借用仅在 [const 上下文][const context]中被借用的[位置表达式][place expression]是*瞬态的*、*间接的*或*静态的*时才允许。
  >
  > 如果位置表达式是当前 const 上下文局部的变量或其临时作用域包含在当前 const 上下文内的表达式，则该位置表达式是*瞬态的*。
  >
  > ```rust
  > // 借用是对初始化器局部变量的借用，因此此位置表达式是瞬态的。
  > const C: () = { let mut x = 0; _ = &mut x; };
  > ```
  >
  > ```rust
  > // 借用是对其作用域未被延长的临时值的借用，因此此位置表达式是瞬态的。
  > const C: () = { _ = &mut 0u8; };
  > ```
  >
  > ```rust
  > // 当临时值被提升但生命周期未被延长时，其位置表达式仍被视为瞬态的。
  > const C: () = { let _: &'static mut [u8] = &mut []; };
  > ```
  >
  > 如果位置表达式是[解引用表达式][dereference expression]，则该位置表达式是*间接的*。
  >
  > ```rust
  > const C: () = { _ = &mut *(&mut 0); };
  > ```
  >
  > 如果位置表达式是 `static` 项，则该位置表达式是*静态的*。
  >
  > ```rust
  > # #![allow(static_mut_refs)]
  > const C: &u8 = unsafe { static mut S: u8 = 0; &mut S };
  > ```

  > [!NOTE]
  > 这些规则的一个令人惊讶的后果是我们允许这种写法，
  >
  > ```rust
  > const C: &[u8] = { let x: &mut [u8] = &mut []; x }; // 正确
  > //                                    ~~~~~~~
  > // 空数组即使在可变借用之后也会被提升。
  > ```
  >
  > 但我们不允许这段类似的代码：
  >
  > ```rust,compile_fail,E0764
  > const C: &[u8] = &mut []; // 错误
  > //               ~~~~~~~
  > //           尾部表达式。
  > ```
  >
  > 这两者之间的区别在于，在第一种情况下，空数组被[提升][promoted]但其作用域不经历[临时生命周期延长][temporary lifetime extension]，因此我们认为[位置表达式][place expression]是瞬态的（即使在提升之后该位置确实存活到程序结束）。在第二种情况下，空数组临时值的作用域确实经历了生命周期延长，因此由于是对生命周期延长临时值的可变借用而被拒绝（因此借用了非瞬态位置表达式）。
  >
  > 这种效果令人惊讶，因为在这种情况下，临时生命周期延长导致比没有它更少的代码可以编译。
  >
  > 更多细节请参见 [issue #143129](https://github.com/rust-lang/rust/issues/143129)。

r[const-eval.const-expr.deref]
* [解引用表达式][Dereference expressions]。

  ```rust,no_run
  # use core::cell::UnsafeCell;
  const _: u8 = unsafe {
      let x: *mut u8 = &raw mut *&mut 0;
      //                        ^^^^^^^
      //             可变引用的解引用。
      *x = 1; // 可变指针的解引用。
      *(x as *const u8) // 常量指针的解引用。
  };
  const _: u8 = unsafe {
      let x = &UnsafeCell::new(0);
      *x.get() = 1; // 内部可变值的修改。
      *x.get()
  };
  ```

r[const-eval.const-expr.group]

* [分组][Grouped]表达式。

r[const-eval.const-expr.cast]
* [强制转换][Cast]表达式，除了
  * 指针到地址的强制转换和
  * 函数指针到地址的强制转换。

r[const-eval.const-expr.const-fn]
* [const 函数][const functions]和 const 方法的调用。

r[const-eval.const-expr.loop]
* [loop] 和 [while] 表达式。

r[const-eval.const-expr.if-match]
* [if] 和 [match] 表达式。

r[const-eval.const-context]
## Const 上下文
[const context]: #const-context

r[const-eval.const-context.def]
*const 上下文*是以下之一：

r[const-eval.const-context.array-length]
* [数组类型长度表达式][Array type length expressions]

r[const-eval.const-context.repeat-length]
* [数组重复长度表达式][array expressions]

r[const-eval.const-context.init]
* 以下内容的初始化器
  * [常量][constants]
  * [静态项][statics]
  * [枚举判别值][enum discriminants]

r[const-eval.const-context.generic]
* [const 泛型参数][const generic argument]

r[const-eval.const-context.block]
* [const 块][const block]

r[const-eval.const-context.outer-generics]
数组类型长度表达式、数组重复长度表达式和 const 泛型参数在使用外部泛型参数时受到限制：此类表达式必须是单个 const 泛型参数，或者是不引用任何泛型参数的表达式。

r[const-eval.const-fn]
## Const 函数

r[const-eval.const-fn.intro]
*const 函数*是可以从 const 上下文调用的函数。它使用 `const` 限定符定义，也包括[元组结构体][tuple struct]和[元组枚举变体][tuple enum variant]构造函数。

> [!EXAMPLE]
> ```rust
> const fn square(x: i32) -> i32 { x * x }
>
> const VALUE: i32 = square(12);
> ```

r[const-eval.const-fn.const-context]
从 const 上下文调用时，const 函数由编译器在编译时解释。解释发生在编译目标的环境中，而不是主机环境中。因此，如果你针对 `32` 位系统编译，`usize` 是 `32` 位，无论你是在 `64` 位还是 `32` 位系统上构建。

r[const-eval.const-fn.outside-context]
当 const 函数在 const 上下文之外调用时，其行为与没有 `const` 限定符时相同。

r[const-eval.const-fn.body-restriction]
const 函数的主体只能使用[常量表达式][constant expressions]。

r[const-eval.const-fn.async]
const 函数不允许是 [async]。

r[const-eval.const-fn.type-restrictions]
const 函数的参数和返回类型的类型限制为与 const 上下文兼容的类型。
<!-- TODO: Define the type restrictions. -->

[arithmetic]:           expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[array expressions]:    expressions/array-expr.md
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions
[array type length expressions]: types/array.md
[assignment expressions]: expressions/operator-expr.md#assignment-expressions
[async]:                items/functions.md#async-functions
[compound assignment expressions]: expressions/operator-expr.md#compound-assignment-expressions
[block expressions]:    expressions/block-expr.md
[borrow]:               expressions/operator-expr.md#borrow-operators
[cast]:                 expressions/operator-expr.md#type-cast-expressions
[closure expressions]:  expressions/closure-expr.md
[comparison]:           expressions/operator-expr.md#comparison-operators
[const block]:          expressions/block-expr.md#const-blocks
[const functions]:      items/functions.md#const-functions
[const generic argument]: items/generics.md#const-generics
[const generic parameters]: items/generics.md#const-generics
[constant expressions]: #constant-expressions
[constants]:            items/constant-items.md
[Const parameters]:     items/generics.md
[dereference expression]: expr.deref
[dereference expressions]: expr.deref
[destructors]:          destructors.md
[enum discriminants]:   items/enumerations.md#discriminants
[expression statements]: statements.md#expression-statements
[expressions]:          expressions.md
[`extern` statics]:     items/external-blocks.md#statics
[field expressions]:    expressions/field-expr.md
[functions]:            items/functions.md
[grouped]:              expressions/grouped-expr.md
[interior mutability]:  interior-mutability.md
[if]:                   expressions/if-expr.md#if-expressions
[lazy boolean]:         expressions/operator-expr.md#lazy-boolean-operators
[let statements]:       statements.md#let-statements
[literals]:             expressions/literal-expr.md
[logical]:              expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[loop]:                 expressions/loop-expr.md#infinite-loops
[match]:                expressions/match-expr.md
[negation]:             expressions/operator-expr.md#negation-operators
[overflow]:             expressions/operator-expr.md#overflow
[paths]:                expressions/path-expr.md
[patterns]:             patterns.md
[place expression]:     expr.place-value.place-memory-location
[promoted expression]:  destructors.md#constant-promotion
[promoted]:             destructors.md#constant-promotion
[range expressions]:    expressions/range-expr.md
[statics]:              items/static-items.md
[Struct expressions]:   expressions/struct-expr.md
[temporary lifetime extension]: destructors.scope.lifetime-extension
[tuple enum variant]:   items/enumerations.md
[tuple expressions]:    expressions/tuple-expr.md
[tuple struct]:         items/structs.md
[while]:                expressions/loop-expr.md#predicate-loops
