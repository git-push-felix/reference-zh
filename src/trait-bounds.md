r[bound]
# trait 约束与生命周期约束

r[bound.syntax]
```grammar,miscellaneous
Bounds -> Bound ( `+` Bound )* `+`?

Bound -> Lifetime | TraitBound | UseBound

TraitBound ->
      ( `?` | ForLifetimes )? TypePath
    | `(` ( `?` | ForLifetimes )? TypePath `)`

LifetimeBounds -> ( Lifetime `+` )* Lifetime?

Lifetime ->
      LIFETIME_OR_LABEL
    | `'static`
    | `'_`

UseBound -> `use` UseBoundGenericArgs

UseBoundGenericArgs ->
      `<` `>`
    | `<` ( UseBoundGenericArg `,`)* UseBoundGenericArg `,`? `>`

UseBoundGenericArg ->
      Lifetime
    | IDENTIFIER
    | `Self`
```

r[bound.intro]
[Trait] 约束和生命周期约束为[泛型项][generic]提供了一种方式来限制哪些类型和生命周期可以用作它们的参数。约束可以在 [where 子句]中的任何类型上提供。对于某些常见情况，还有更简短的形式：

* 在声明[泛型参数][generic]之后写出的约束：`fn f<A: Copy>() {}` 等同于 `fn f<A>() where A: Copy {}`。
* 在 trait 声明中作为[超 trait][supertraits]：`trait Circle : Shape {}` 等价于 `trait Circle where Self : Shape {}`。
* 在 trait 声明中作为对[关联类型][associated types]的约束：`trait A { type B: Copy; }` 等价于 `trait A where Self::B: Copy { type B; }`。

r[bound.satisfaction]
项上的约束在使用该项时必须被满足。在对泛型项进行类型检查和借用检查时，约束可用于确定某个 trait 已为某个类型实现。例如，给定 `Ty: Trait`：

* 在泛型函数体内，可以对 `Ty` 值调用 `Trait` 的方法。同样，可以使用 `Trait` 上的关联常量。
* 可以使用 `Trait` 的关联类型。
* 具有 `T: Trait` 约束的泛型函数和类型可以使用，其中 `Ty` 被用于 `T`。

```rust
# type Surface = i32;
trait Shape {
    fn draw(&self, surface: Surface);
    fn name() -> &'static str;
}

fn draw_twice<T: Shape>(surface: Surface, sh: T) {
    sh.draw(surface);           // 可以调用方法，因为 T: Shape
    sh.draw(surface);
}

fn copy_and_draw_twice<T: Copy>(surface: Surface, sh: T) where T: Shape {
    let shape_copy = sh;        // 不会移动 sh，因为 T: Copy
    draw_twice(surface, sh);    // 可以使用泛型函数，因为 T: Shape
}

struct Figure<S: Shape>(S, S);

fn name_figure<U: Shape>(
    figure: Figure<U>,          // 类型 Figure<U> 是良构的，因为 U: Shape
) {
    println!(
        "Figure of two {}",
        U::name(),              // 可以使用关联函数
    );
}
```

r[bound.trivial]
不使用项的参数或[高阶生命周期][higher-ranked lifetimes]的约束会在项被定义时检查。如果这样的约束为假，则报错。

r[bound.special]
[`Copy`]、[`Clone`] 和 [`Sized`] 约束也会在调用项时对某些泛型类型进行检查，即使调用时没有提供具体类型。将 `Copy` 或 `Clone` 作为可变引用、[trait 对象][trait object]或[切片][slice]的约束是错误的。将 `Sized` 作为 trait 对象或切片的约束是错误的。

```rust,compile_fail
struct A<'a, T>
where
    i32: Default,           // 允许，但没有用处
    i32: Iterator,          // 错误：`i32` 不是迭代器
    &'a mut T: Copy,        // （使用时）错误：trait 约束未满足
    [T]: Sized,             // （使用时）错误：编译时无法确定大小
{
    f: &'a T,
}
struct UsesA<'a, T>(A<'a, T>);
```

r[bound.trait-object]
trait 约束和生命周期约束也用于命名 [trait 对象][trait objects]。

r[bound.sized]
## `?Sized`

`?` 仅用于放宽对[类型参数][type parameters]或[关联类型][associated types]的隐式 [`Sized`] trait 约束。`?Sized` 不能用于对其他类型的约束。

r[bound.lifetime]
## 生命周期约束

r[bound.lifetime.intro]
生命周期约束可以应用于类型或其他生命周期。

r[bound.lifetime.outlive-lifetime]
`'a: 'b` 通常读作 `'a` *存活不短于* `'b`。`'a: 'b` 意味着 `'a` 至少和 `'b` 一样长，因此引用 `&'a ()` 在 `&'b ()` 有效的任何地方都有效。

```rust
fn f<'a, 'b>(x: &'a i32, mut y: &'b i32) where 'a: 'b {
    y = x;                      // &'a i32 是 &'b i32 的子类型，因为 'a: 'b
    let r: &'b &'a i32 = &&0;   // &'b &'a i32 是良构的，因为 'a: 'b
}
```

r[bound.lifetime.outlive-type]
`T: 'a` 意味着 `T` 的所有生命周期参数都存活不短于 `'a`。例如，如果 `'a` 是一个无约束的生命周期参数，那么 `i32: 'static` 和 `&'static str: 'a` 是满足的，但 `Vec<&'a ()>: 'static` 则不满足。

r[bound.higher-ranked]
## 高阶 trait 约束

r[bound.higher-ranked.syntax]
```grammar,miscellaneous
ForLifetimes -> `for` GenericParams
```

r[bound.higher-ranked.intro]
trait 约束可以是*高阶*的（higher ranked），即对生命周期进行量化。这些约束指定了一个*对所有*生命周期都成立的约束。例如，形如 `for<'a> &'a T: PartialEq<i32>` 的约束将要求如下的实现：

```rust
# struct T;
impl<'a> PartialEq<i32> for &'a T {
    // ...
#    fn eq(&self, other: &i32) -> bool {true}
}
```

然后就可以用它将具有任意生命周期的 `&'a T` 与 `i32` 进行比较。

只有高阶约束才能在这里使用，因为引用的生命周期比函数上任何可能的生命周期参数都短：

```rust
fn call_on_ref_zero<F>(f: F) where for<'a> F: Fn(&'a i32) {
    let zero = 0;
    f(&zero);
}
```

r[bound.higher-ranked.trait]
高阶生命周期也可以直接写在 trait 之前：唯一的区别是生命周期参数的[作用域][hrtb-scopes]，它仅延伸到紧随其后的 trait 结束，而非整个约束。下面这个函数与上一个等价。

```rust
fn call_on_ref_zero<F>(f: F) where F: for<'a> Fn(&'a i32) {
    let zero = 0;
    f(&zero);
}
```

r[bound.implied]
## 隐含约束

r[bound.implied.intro]
类型要良构所需满足的生命周期约束有时会被推断出来。

```rust
fn requires_t_outlives_a<'a, T>(x: &'a T) {}
```

类型参数 `T` 需要存活不短于 `'a` 才能使类型 `&'a T` 良构。这会被推断出来，因为函数签名中包含类型 `&'a T`，而该类型仅在 `T: 'a` 成立时才有效。

r[bound.implied.context]
隐含约束会被添加到函数的所有参数和返回值上。在 `requires_t_outlives_a` 内部，可以假定 `T: 'a` 成立，即使你没有显式指定：

```rust
fn requires_t_outlives_a_not_implied<'a, T: 'a>() {}

fn requires_t_outlives_a<'a, T>(x: &'a T) {
    // 这段代码可以编译，因为 `T: 'a` 由
    // 引用类型 `&'a T` 隐含。
    requires_t_outlives_a_not_implied::<'a, T>();
}
```

```rust,compile_fail,E0309
# fn requires_t_outlives_a_not_implied<'a, T: 'a>() {}
fn not_implied<'a, T>() {
    // 这会报错，因为 `T: 'a` 没有由
    // 函数签名隐含。
    requires_t_outlives_a_not_implied::<'a, T>();
}
```

r[bound.implied.trait]
只有生命周期约束会被隐含，trait 约束仍然必须显式添加。因此下面的示例会引发错误：

```rust,compile_fail,E0277
use std::fmt::Debug;
struct IsDebug<T: Debug>(T);
// error[E0277]: `T` 没有实现 `Debug`
fn doesnt_specify_t_debug<T>(x: IsDebug<T>) {}
```

r[bound.implied.def]
生命周期约束也会为类型定义和任何类型的 impl 块进行推断：

```rust
struct Struct<'a, T> {
    // 这要求 `T: 'a` 以使其良构，
    // 这由编译器推断。
    field: &'a T,
}

enum Enum<'a, T> {
    // 这要求 `T: 'a` 以使其良构，
    // 这由编译器推断。
    //
    // 注意，即使只使用 `Enum::OtherVariant`，
    // 也要求 `T: 'a` 成立。
    SomeVariant(&'a T),
    OtherVariant,
}

trait Trait<'a, T: 'a> {}

// 这会报错，因为 impl 头部中没有任何类型隐含 `T: 'a`。
//     impl<'a, T> Trait<'a, T> for () {}

// 这段可以编译，因为 `T: 'a` 由 self 类型 `&'a T` 隐含。
impl<'a, T> Trait<'a, T> for &'a T {}
```

r[bound.use]
## Use 约束

某些约束列表可以包含 `use<..>` 约束来控制哪些泛型参数被 `impl Trait` [抽象返回类型][abstract return type]所捕获。更多细节请参阅[精确捕获][precise capturing]。

[抽象返回类型]: types/impl-trait.md#abstract-return-types
[arrays]: types/array.md
[associated types]: items/associated-items.md#associated-types
[hrtb-scopes]: names/scopes.md#higher-ranked-trait-bound-scopes
[supertraits]: items/traits.md#supertraits
[generic]: items/generics.md
[higher-ranked lifetimes]: #higher-ranked-trait-bounds
[precise capturing]: types/impl-trait.md#precise-capturing
[slice]: types/slice.md
[Trait]: items/traits.md#trait-bounds
[trait object]: types/trait-object.md
[trait objects]: types/trait-object.md
[type parameters]: types/parameters.md
[where clause]: items/generics.md#where-clauses
