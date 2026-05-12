r[lang-types]
# 特殊类型和 trait

r[lang-types.intro]
[标准库][the standard library]中的某些类型和 trait 为 Rust 编译器所知。本章记录了这些类型和 trait 的特殊特性。

r[lang-types.box]
## `Box<T>`

r[lang-types.box.intro]
[`Box<T>`] 具有一些 Rust 目前不允许用户定义类型使用的特殊特性。

r[lang-types.box.deref]
* `Box<T>` 的[解引用运算符][dereference operator]产生一个可以[移出][moved from]的位置。这意味着 `*` 运算符和 `Box<T>` 的析构函数是语言内置的。

r[lang-types.box.receiver]
* [方法][Methods]可以将 `Box<Self>` 作为接收者（receiver）。

r[lang-types.box.fundamental]
* 可以在与 `T` 相同的 crate 中为 `Box<T>` 实现 trait，而[孤儿规则][orphan rules]禁止其他泛型类型这样做。

<!-- Editor Note: 这远非详尽列表 -->

r[lang-types.rc]
## `Rc<T>`

r[lang-types.rc.receiver]
[方法][Methods]可以将 [`Rc<Self>`] 作为接收者。

r[lang-types.arc]
## `Arc<T>`

r[lang-types.arc.receiver]
[方法][Methods]可以将 [`Arc<Self>`] 作为接收者。

r[lang-types.pin]
## `Pin<P>`

r[lang-types.pin.receiver]
[方法][Methods]可以将 [`Pin<P>`] 作为接收者。

r[lang-types.unsafe-cell]
## `UnsafeCell<T>`

r[lang-types.unsafe-cell.interior-mut]
[`std::cell::UnsafeCell<T>`] 用于[内部可变性][interior mutability]。它确保编译器不会对此类类型执行不正确的优化。

r[lang-types.unsafe-cell.read-only-alloc]
它还确保具有内部可变性类型的 [`static` 项][`static` items]不会被放置在标记为只读的内存中。

r[lang-types.phantom-data]
## `PhantomData<T>`

[`std::marker::PhantomData<T>`] 是一个[零大小][zero-sized]、最小对齐的类型，对于[变型][variance]、[丢弃检查][drop check]和[自动 trait](#自动-trait)的目的，它被认为是拥有一个 `T`。

r[lang-types.ops]
## 运算符 trait

[`std::ops`] 和 [`std::cmp`] 中的 trait 用于重载[运算符][operators]、[索引表达式][indexing expressions]和[调用表达式][call expressions]。

r[lang-types.deref]
## `Deref` 和 `DerefMut` {#deref-and-derefmut}

除了重载一元 `*` 运算符外，[`Deref`] 和 [`DerefMut`] 还用于[方法解析][method resolution]和[解引用强制转换][deref coercions]。

r[lang-types.drop]
## `Drop`

[`Drop`] trait 提供了一个[析构函数][destructor]，在此类型的值将被销毁时运行。

r[lang-types.copy]
## `Copy`

r[lang-types.copy.intro]
[`Copy`] trait 改变了实现它的类型的语义。

r[lang-types.copy.behavior]
类型实现 `Copy` 的值在赋值时被复制而不是移动。

r[lang-types.copy.constraint]
`Copy` 只能为不实现 `Drop` 且其所有字段都是 `Copy` 的类型实现。对于枚举，这意味着所有变体的所有字段都必须是 `Copy`。对于联合体，这意味着所有变体都必须是 `Copy`。

r[lang-types.copy.builtin-types]
编译器为以下类型实现 `Copy`：

r[lang-types.copy.tuple]
* `Copy` 类型的[元组][Tuples]

r[lang-types.copy.fn-pointer]
* [函数指针][Function pointers]

r[lang-types.copy.fn-item]
* [函数项][Function items]

r[lang-types.copy.closure]
* 不捕获值或仅捕获 `Copy` 类型值的[闭包][Closures]

r[lang-types.clone]
## `Clone`

r[lang-types.clone.intro]
[`Clone`] trait 是 `Copy` 的超 trait，因此它也需要编译器生成的实现。

r[lang-types.clone.builtin-types]
编译器为以下类型实现它：

r[lang-types.clone.builtin-copy]
* 具有内置 `Copy` 实现的类型（见上文）

r[lang-types.clone.tuple]
* `Clone` 类型的[元组][Tuples]

r[lang-types.clone.closure]
* 仅捕获 `Clone` 类型值或不从环境捕获值的[闭包][Closures]

r[lang-types.send]
## `Send`

[`Send`] trait 表示此类型的值可以安全地从一个线程发送到另一个线程。

r[lang-types.sync]
## `Sync`

r[lang-types.sync.intro]
[`Sync`] trait 表示此类型的值可以安全地在多个线程之间共享。

r[lang-types.sync.static-constraint]
所有在不可变 [`static` 项][`static` items]中使用的类型都必须实现此 trait。

r[lang-types.termination]
## `Termination`

[`Termination`] trait 指示 [main 函数][main function]和[测试函数][test functions]可接受的返回类型。

r[lang-types.auto-traits]
## 自动 trait {#auto-traits}

[`Send`]、[`Sync`]、[`Unpin`]、[`UnwindSafe`] 和 [`RefUnwindSafe`] trait 是*自动 trait*。自动 trait 具有特殊属性。

r[lang-types.auto-traits.auto-impl]
如果没有为给定类型的自动 trait 编写显式实现或否定实现，则编译器根据以下规则自动实现它：

r[lang-types.auto-traits.builtin-composite]
* 如果 `T` 实现了该 trait，则 `&T`、`&mut T`、`*const T`、`*mut T`、`[T; n]` 和 `[T]` 也实现该 trait。

r[lang-types.auto-traits.fn-item-pointer]
* 函数项类型和函数指针自动实现该 trait。

r[lang-types.auto-traits.aggregate]
* 如果结构体、枚举、联合体和元组的所有字段都实现了该 trait，则它们也实现该 trait。

r[lang-types.auto-traits.closure]
* 如果闭包的所有捕获的类型都实现了该 trait，则闭包也实现该 trait。通过共享引用捕获 `T` 和通过值捕获 `U` 的闭包实现了 `&T` 和 `U` 都实现的任何自动 trait。

r[lang-types.auto-traits.generic-impl]
对于泛型类型（将上述内置类型视为对 `T` 泛型），如果存在泛型实现，则编译器不会自动为那些本可以使用该实现但不满足所需 trait 约束的类型实现它。例如，标准库为所有 `T` 是 `Sync` 的 `&T` 实现了 `Send`；这意味着如果 `T` 是 `Send` 但不是 `Sync`，编译器将不会为 `&T` 实现 `Send`。

r[lang-types.auto-traits.negative]
自动 trait 也可以有否定实现，在标准库文档中显示为 `impl !AutoTrait for T`，覆盖自动实现。例如 `*mut T` 具有 `Send` 的否定实现，因此 `*mut T` 不是 `Send`，即使 `T` 是。目前没有稳定方式来指定额外的否定实现；它们仅存在于标准库中。

r[lang-types.auto-traits.trait-object-marker]
自动 trait 可以作为附加约束添加到任何 [trait 对象][trait object]中，即使通常只允许一个 trait。例如，`Box<dyn Debug + Send + UnwindSafe>` 是有效类型。

r[lang-types.sized]
## `Sized`

r[lang-types.sized.intro]
[`Sized`] trait 表示此类型的大小在编译时已知；也就是说，它不是[动态大小类型][dynamically sized type]。

r[lang-types.sized.implicit-sized]
[类型参数][Type parameters]（trait 中的 `Self` 除外）默认是 `Sized`，[关联类型][associated types]也是如此。

r[lang-types.sized.implicit-impl]
`Sized` 始终由编译器自动实现，而不是通过[实现项][implementation items]。

r[lang-types.sized.relaxation]
这些隐式 `Sized` 约束可以通过使用特殊的 `?Sized` 约束来放宽。

[`Arc<Self>`]: std::sync::Arc
[`Deref`]: std::ops::Deref
[`DerefMut`]: std::ops::DerefMut
[`Pin<P>`]: std::pin::Pin
[`Rc<Self>`]: std::rc::Rc
[`RefUnwindSafe`]: std::panic::RefUnwindSafe
[`Termination`]: std::process::Termination
[`UnwindSafe`]: std::panic::UnwindSafe
[`Unpin`]: std::marker::Unpin

[Arrays]: types/array.md
[associated types]: items/associated-items.md#associated-types
[call expressions]: expressions/call-expr.md
[deref coercions]: type-coercions.md#coercion-types
[dereference operator]: expressions/operator-expr.md#the-dereference-operator
[destructor]: destructors.md
[drop check]: ../nomicon/dropck.html
[dynamically sized type]: dynamically-sized-types.md
[Function pointers]: types/function-pointer.md
[Function items]: types/function-item.md
[implementation items]: items/implementations.md
[indexing expressions]: expressions/array-expr.md#array-and-slice-indexing-expressions
[interior mutability]: interior-mutability.md
[main function]: crates-and-source-files.md#main-functions
[Methods]: items/associated-items.md#associated-functions-and-methods
[method resolution]: expressions/method-call-expr.md
[moved from]: expr.move.movable-place
[operators]: expressions/operator-expr.md
[orphan rules]: items/implementations.md#trait-implementation-coherence
[`static` items]: items/static-items.md
[test functions]: attributes/testing.md#the-test-attribute
[the standard library]: std
[trait object]: types/trait-object.md
[Tuples]: types/tuple.md
[Type parameters]: types/parameters.md
[variance]: subtyping.md#variance
[zero-sized]: glossary.zst
[Closures]: types/closure.md
