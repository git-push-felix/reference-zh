r[items.traits]
# trait

r[items.traits.syntax]
```grammar,items
Trait ->
    `unsafe`? `trait` IDENTIFIER GenericParams? ( `:` Bounds? )? WhereClause?
    `{`
        InnerAttribute*
        AssociatedItem*
    `}`
```

r[items.traits.intro]
*trait* 描述一个类型可以实现的抽象接口。该接口由[关联程序项][associated items]组成，这些程序项有三种类型：

- [函数](associated-items.md#associated-functions-and-methods)
- [类型](associated-items.md#associated-types)
- [常量](associated-items.md#associated-constants)

r[items.traits.namespace]
trait 声明在其所在模块或块的[类型命名空间][type namespace]中定义一个 trait。

r[items.traits.associated-item-namespaces]
关联程序项被定义为 trait 的成员，位于各自的命名空间中。关联类型在类型命名空间中定义。关联常量和关联函数在值命名空间中定义。

r[items.traits.self-param]
所有 trait 都定义一个隐式的类型参数 `Self`，它指代"实现此接口的类型"。trait 还可以包含额外的类型参数。这些类型参数（包括 `Self`）可像[通常一样][generics]受到其他 trait 等的约束。

r[items.traits.impls]
trait 通过单独的[实现][implementations]为特定类型实现。

r[items.traits.associated-item-decls]
trait 函数可以省略函数体，用分号代替。这表示实现必须定义该函数。如果 trait 函数定义了主体，则该定义作为任何未覆盖它的实现的默认实现。类似地，关联常量可以省略等号和表达式，表示实现必须定义常量值。关联类型绝不能定义类型，类型只能在实现中指定。

```rust
// 带有定义和无定义的关联 trait 程序项示例。
trait Example {
    const CONST_NO_DEFAULT: i32;
    const CONST_WITH_DEFAULT: i32 = 99;
    type TypeNoDefault;
    fn method_without_default(&self);
    fn method_with_default(&self) {}
}
```

r[items.traits.const-fn]
trait 函数不允许是 [`const`]。

r[items.traits.bounds]
## trait 约束 {#trait-bounds}

泛型程序项可以使用 trait 作为其类型参数的[约束][bounds]。

r[items.traits.generic]
## 泛型 trait {#generic-traits}

可以为 trait 指定类型参数使其成为泛型。这些参数出现在 trait 名之后，使用与[泛型函数][generic functions]相同的语法。

```rust
trait Seq<T> {
    fn len(&self) -> u32;
    fn elt_at(&self, n: u32) -> T;
    fn iter<F>(&self, f: F) where F: Fn(T);
}
```

<a id="object-safety"></a>
r[items.traits.dyn-compatible]
## Dyn 兼容性 {#dyn-compatibility}

r[items.traits.dyn-compatible.intro]
dyn 兼容的 trait 可以作为 [trait 对象][trait object]的基础 trait。如果 trait 具有以下特性，则它是*dyn 兼容*的：

r[items.traits.dyn-compatible.supertraits]
* 所有[超 trait][supertraits] 也必须是 dyn 兼容的。

r[items.traits.dyn-compatible.sized]
* `Sized` 不能是[超 trait][supertraits]。换句话说，它不能要求 `Self: Sized`。

r[items.traits.dyn-compatible.associated-consts]
* 不能有任何关联常量。

r[items.traits.dyn-compatible.associated-types]
* 不能有任何带泛型的关联类型。

r[items.traits.dyn-compatible.associated-functions]
* 所有关联函数必须要么可从 trait 对象分派，要么显式地不可分派：
    * 可分发函数必须：
        * 没有任何类型参数（允许生命周期参数）。
        * 是[方法][method]，除了在接收器类型中之外不使用 `Self`。
        * 具有以下类型之一的接收器：
            * `&Self`（即 `&self`）
            * `&mut Self`（即 `&mut self`）
            * [`Box<Self>`]
            * [`Rc<Self>`]
            * [`Arc<Self>`]
            * [`Pin<P>`]，其中 `P` 是以上类型之一
        * 没有不透明返回类型；即
            * 不是 `async fn`（它有一个隐藏的 `Future` 类型）。
            * 没有返回位置的 `impl Trait` 类型（`fn example(&self) -> impl Trait`）。
        * 没有 `where Self: Sized` 约束（`Self` 的接收器类型（即 `self`）隐含此约束）。
    * 显式不可分派函数要求：
        * 具有 `where Self: Sized` 约束（`Self` 的接收器类型（即 `self`）隐含此约束）。

r[items.traits.dyn-compatible.async-traits]
* [`AsyncFn`]、[`AsyncFnMut`] 和 [`AsyncFnOnce`] trait 不是 dyn 兼容的。

> [!NOTE]
> 此概念以前称为*对象安全性*。

```rust
# use std::rc::Rc;
# use std::sync::Arc;
# use std::pin::Pin;
// dyn 兼容方法的示例。
trait TraitMethods {
    fn by_ref(self: &Self) {}
    fn by_ref_mut(self: &mut Self) {}
    fn by_box(self: Box<Self>) {}
    fn by_rc(self: Rc<Self>) {}
    fn by_arc(self: Arc<Self>) {}
    fn by_pin(self: Pin<&Self>) {}
    fn with_lifetime<'a>(self: &'a Self) {}
    fn nested_pin(self: Pin<Arc<Self>>) {}
}
# struct S;
# impl TraitMethods for S {}
# let t: Box<dyn TraitMethods> = Box::new(S);
```

```rust,compile_fail
// 此 trait 是 dyn 兼容的，但这些方法不能在 trait 对象上分派。
trait NonDispatchable {
    // 非方法不能分派。
    fn foo() where Self: Sized {}
    // Self 类型直到运行时才知道。
    fn returns(&self) -> Self where Self: Sized;
    // `other` 可能是接收器的不同具体类型。
    fn param(&self, other: Self) where Self: Sized {}
    // 泛型与虚函数表不兼容。
    fn typed<T>(&self, x: T) where Self: Sized {}
}

struct S;
impl NonDispatchable for S {
    fn returns(&self) -> Self where Self: Sized { S }
}
let obj: Box<dyn NonDispatchable> = Box::new(S);
obj.returns(); // 错误：不能用 Self 返回值调用
obj.param(S);  // 错误：不能用 Self 参数调用
obj.typed(1);  // 错误：不能用泛型类型调用
```

```rust,compile_fail
# use std::rc::Rc;
// dyn 不兼容 trait 的示例。
trait DynIncompatible {
    const CONST: i32 = 1;  // 错误：不能有关联常量

    fn foo() {}  // 错误：不带 Sized 的关联函数
    fn returns(&self) -> Self; // 错误：返回类型中的 Self
    fn typed<T>(&self, x: T) {} // 错误：有泛型类型参数
    fn nested(self: Rc<Box<Self>>) {} // 错误：嵌套接收器不能在其上分派
}

struct S;
impl DynIncompatible for S {
    fn returns(&self) -> Self { S }
}
let obj: Box<dyn DynIncompatible> = Box::new(S); // 错误
```

```rust,compile_fail
// `Self: Sized` trait 是 dyn 不兼容的。
trait TraitWithSize where Self: Sized {}

struct S;
impl TraitWithSize for S {}
let obj: Box<dyn TraitWithSize> = Box::new(S); // 错误
```

```rust,compile_fail
// 如果 `Self` 是类型参数，则 dyn 不兼容。
trait Super<A> {}
trait WithSelf: Super<Self> where Self: Sized {}

struct S;
impl<A> Super<A> for S {}
impl WithSelf for S {}
let obj: Box<dyn WithSelf> = Box::new(S); // 错误：不能使用 `Self` 类型参数
```

r[items.traits.supertraits]
## 超 trait {#supertraits}

r[items.traits.supertraits.intro]
**超 trait** 是类型要实现特定 trait 所必须实现的 trait。此外，当[泛型][generics]或 [trait 对象][trait object]受到 trait 约束时，它可以访问其超 trait 的关联程序项。

r[items.traits.supertraits.decl]
超 trait 通过对 trait 的 `Self` 类型的 trait 约束以及这些 trait 约束中声明的 trait 的超 trait 传递地声明。trait 作为自身的超 trait 是错误的。

r[items.traits.supertraits.subtrait]
带有超 trait 的 trait 称为其超 trait 的**子 trait**。

以下是将 `Shape` 声明为 `Circle` 的超 trait 的示例。

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle: Shape { fn radius(&self) -> f64; }
```

以下是相同的示例，但使用了 [where 子句][where clauses]。

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape { fn radius(&self) -> f64; }
```

下一个示例使用 `Shape` 的 `area` 函数为 `radius` 提供了默认实现。

```rust
# trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape {
    fn radius(&self) -> f64 {
        // A = pi * r^2
        // 因此代数推导，
        // r = sqrt(A / pi)
        (self.area() / std::f64::consts::PI).sqrt()
    }
}
```

下一个示例在泛型参数上调用超 trait 方法。

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle: Shape { fn radius(&self) -> f64; }
fn print_area_and_radius<C: Circle>(c: C) {
    // 这里我们从 `Circle` 的超 trait `Shape` 调用 area 方法。
    println!("Area: {}", c.area());
    println!("Radius: {}", c.radius());
}
```

类似地，以下是在 trait 对象上调用超 trait 方法的示例。

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle: Shape { fn radius(&self) -> f64; }
# struct UnitCircle;
# impl Shape for UnitCircle { fn area(&self) -> f64 { std::f64::consts::PI } }
# impl Circle for UnitCircle { fn radius(&self) -> f64 { 1.0 } }
# let circle = UnitCircle;
let circle = Box::new(circle) as Box<dyn Circle>;
let nonsense = circle.radius() * circle.area();
```

r[items.traits.safety]
## Unsafe trait {#unsafe-traits}

r[items.traits.safety.intro]
以 `unsafe` 关键字开头的 trait 程序项表示*实现*该 trait 可能是[不安全][unsafe]的。使用正确实现的 unsafe trait 是安全的。[trait 实现][trait implementation]也必须以 `unsafe` 关键字开头。

[`Sync`] 和 [`Send`] 是 unsafe trait 的示例。

r[items.traits.params]
## 参数模式 {#parameter-patterns}

r[items.traits.params.patterns-no-body]
不带主体的关联函数中的参数只允许使用 [IDENTIFIER] 或 `_` [通配符][WildcardPattern] 模式，以及 [SelfParam] 允许的形式。`mut` [IDENTIFIER] 目前是允许的，但已被弃用，将来会变成硬错误。
<!-- https://github.com/rust-lang/rust/issues/35203 -->

```rust
trait T {
    fn f1(&self);
    fn f2(x: Self, _: i32);
}
```

```rust,compile_fail,E0642
trait T {
    fn f2(&x: &i32); // 错误：不带主体的函数中不允许使用模式
}
```

r[items.traits.params.patterns-with-body]
带主体的关联函数中的参数只允许使用不可反驳的模式。

```rust
trait T {
    fn f1((a, b): (i32, i32)) {} // OK：是不可反驳的
}
```

```rust,compile_fail,E0005
trait T {
    fn f1(123: i32) {} // 错误：模式是可反驳的
    fn f2(Some(x): Option<i32>) {} // 错误：模式是可反驳的
}
```

r[items.traits.params.pattern-required.edition2018]
> [!EDITION-2018]
> 在 2018 版本之前，关联函数参数的模式是可选的：
>
> ```rust,edition2015
> // 2015 版本
> trait T {
>     fn f(i32); // OK：参数标识符不是必需的
> }
> ```
>
> 从 2018 版本开始，模式不再是可选的。

r[items.traits.params.restriction-patterns.edition2018]
> [!EDITION-2018]
> 在 2018 版本之前，带主体的关联函数中的参数仅限于以下类型的模式：
>
> * [IDENTIFIER]
> * `mut` [IDENTIFIER]
> * [`_`][WildcardPattern]
> * `&` [IDENTIFIER]
> * `&&` [IDENTIFIER]
>
> ```rust,edition2015,compile_fail,E0642
> // 2015 版本
> trait T {
>     fn f1((a, b): (i32, i32)) {} // 错误：不允许的模式
> }
> ```
>
> 从 2018 开始，所有不可反驳的模式都允许，如 [items.traits.params.patterns-with-body] 中所述。

r[items.traits.associated-visibility]
## 程序项可见性 {#item-visibility}

r[items.traits.associated-visibility.intro]
trait 程序项在语法上允许 [Visibility] 注解，但在 trait 验证时会被拒绝。这允许在不同使用上下文中用统一的语法解析程序项。作为示例，空的 `vis` 宏片段说明符可以用于 trait 程序项，而该宏规则可能在允许可见性的其他情况下使用。

```rust
macro_rules! create_method {
    ($vis:vis $name:ident) => {
        $vis fn $name(&self) {}
    };
}

trait T1 {
    // 空的 `vis` 是允许的。
    create_method! { method_of_t1 }
}

struct S;

impl S {
    // 此处允许可见性。
    create_method! { pub method_of_s }
}

impl T1 for S {}

fn main() {
    let s = S;
    s.method_of_t1();
    s.method_of_s();
}
```

[WildcardPattern]: ../patterns.md#wildcard-pattern
[bounds]: ../trait-bounds.md
[trait object]: ../types/trait-object.md
[associated items]: associated-items.md
[method]: associated-items.md#methods
[supertraits]: #supertraits
[implementations]: implementations.md
[generics]: generics.md
[where clauses]: generics.md#where-clauses
[generic functions]: functions.md#generic-functions
[unsafe]: ../unsafety.md
[trait implementation]: implementations.md#trait-implementations
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[`async`]: functions.md#async-functions
[`const`]: functions.md#const-functions
[type namespace]: ../names/namespaces.md
