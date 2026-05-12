r[type.impl-trait]
# impl Trait

r[type.impl-trait.syntax]
```grammar,types
ImplTraitType -> `impl` Bounds

ImplTraitTypeOneBound -> `impl` TraitBound
```

r[type.impl-trait.intro]
`impl Trait` 提供了指定实现了特定 trait 的未命名但具体类型的方式。它可以出现在两类位置：参数位置（可以充当函数的匿名类型参数）和返回位置（可以充当抽象返回类型）。

```rust
trait Trait {}
# impl Trait for () {}

// 参数位置：匿名类型参数
fn foo(arg: impl Trait) {
}

// 返回位置：抽象返回类型
fn bar() -> impl Trait {
}
```
r[type.impl-trait.param]
## 匿名类型参数

> [!NOTE]
> 这通常被称为"参数位置的 impl Trait"。（"参数"在此是更准确的用语，但"参数位置的 impl Trait"是该特性开发期间使用的措辞，并且仍保留在实现的某些部分。）

r[type.impl-trait.param.intro]
函数可以使用 `impl` 后跟一组 trait 边界来声明参数具有匿名类型。调用者必须提供一个满足匿名类型参数所声明边界的类型，函数也只能使用通过匿名类型参数的 trait 边界可用的方法。

例如，以下两种形式几乎是等价的：

```rust
trait Trait {}

// 泛型类型参数
fn with_generic_type<T: Trait>(arg: T) {
}

// 参数位置的 impl Trait
fn with_impl_trait(arg: impl Trait) {
}
```

r[type.impl-trait.param.generic]
也就是说，参数位置的 `impl Trait` 是类似 `<T: Trait>` 的泛型类型参数的语法糖，不同之处在于该类型是匿名的，并且不出现在 [GenericParams] 列表中。

> [!NOTE]
> 对于函数参数，泛型类型参数和 `impl Trait` 并不完全等价。对于泛型参数如 `<T: Trait>`，调用者可以选择在调用处使用 [GenericArgs] 显式指定 `T` 的泛型参数，例如 `foo::<usize>(1)`。将参数从一种形式更改为另一种会构成函数的调用者的破坏性变更，因为这改变了泛型参数的数量。

r[type.impl-trait.return]
## 抽象返回类型 {#abstract-return-types}

> [!NOTE]
> 这通常被称为"返回位置的 impl Trait"。

r[type.impl-trait.return.intro]
函数可以使用 `impl Trait` 返回一个抽象返回类型。这些类型代表另一个具体类型，调用者只能使用指定的 `Trait` 所声明的方法。

r[type.impl-trait.return.constraint-body]
函数的每个可能返回值必须解析为*相同*的具体类型。

返回位置的 `impl Trait` 允许函数返回未装箱的抽象类型。这对[闭包][closures]和迭代器特别有用。例如，闭包具有唯一的、不可写入的类型。以前，从函数返回闭包的唯一方式是使用 [trait 对象][trait object]：

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

这可能因堆分配和动态分发带来性能损失。以前无法完全指定闭包的类型，只能使用 `Fn` trait。这意味着 trait 对象是必需的。然而，使用 `impl Trait`，可以更简单地编写：

```rust
fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}
```

这也避免了使用装箱 trait 对象的缺点。

类似地，迭代器的具体类型可能变得非常复杂，包含了链中所有先前迭代器的类型。返回 `impl Iterator` 意味着函数仅将 `Iterator` trait 暴露为其返回类型的边界，而不是显式指定所有涉及到的其他迭代器类型。

r[type.impl-trait.return-in-trait]
## trait 和 trait 实现中的返回位置 `impl Trait`

r[type.impl-trait.return-in-trait.intro]
trait 中的函数也可以使用 `impl Trait` 作为匿名关联类型的语法。

r[type.impl-trait.return-in-trait.desugaring]
trait 的关联函数返回类型中的每个 `impl Trait` 都被脱糖为一个匿名关联类型。实现函数签名中出现的返回类型用于确定该关联类型的值。

r[type.impl-trait.generic-captures]
## 捕获 {#capturing}

每个返回位置 `impl Trait` 抽象类型背后都隐藏着某个具体类型。为了让这个具体类型使用泛型参数，该泛型参数必须被抽象类型*捕获*。

r[type.impl-trait.generic-capture.auto]
## 自动捕获

r[type.impl-trait.generic-capture.auto.intro]
返回位置 `impl Trait` 抽象类型自动捕获所有作用域内泛型参数，包括泛型类型、const 和生命周期参数（包括高阶参数）。

r[type.impl-trait.generic-capture.edition2024]
> [!EDITION-2024]
> 在 2024 版本之前，对于自由函数以及固有 impl 的关联函数和方法，未出现在抽象返回类型边界中的泛型生命周期参数不会被自动捕获。

r[type.impl-trait.generic-capture.precise]
## 精确捕获 {#precise-capturing}

r[type.impl-trait.generic-capture.precise.use]
返回位置 `impl Trait` 抽象类型所捕获的泛型参数集合可以通过 [`use<..>` 边界][`use<..>` bound]显式控制。如果存在，则只有在 `use<..>` 边界中列出的泛型参数才会被捕获。例如：

```rust
fn capture<'a, 'b, T>(x: &'a (), y: T) -> impl Sized + use<'a, T> {
  //                                      ~~~~~~~~~~~~~~~~~~~~~~~
  //                                     仅捕获 `'a` 和 `T`。
  (x, y)
}
```

r[type.impl-trait.generic-capture.precise.constraint-single]
目前，边界列表中只能出现一个 `use<..>` 边界，所有作用域内的类型和 const 泛型参数都必须被包含，并且出现在抽象类型其他边界中的所有生命周期参数都必须被包含。

r[type.impl-trait.generic-capture.precise.constraint-lifetime]
在 `use<..>` 边界内，出现的任何生命周期参数必须位于所有类型和 const 泛型参数之前，如果被省略的生命周期（`'_`）本来可以在 `impl Trait` 返回类型中出现，则它也可以出现。

r[type.impl-trait.generic-capture.precise.constraint-param-impl-trait]
由于所有作用域内类型参数必须按名称包含，`use<..>` 边界不能用于使用参数位置 `impl Trait` 的项签名中，因为这些项的作用域中有匿名类型参数。

r[type.impl-trait.generic-capture.precise.constraint-in-trait]
在 trait 定义的关联函数中出现的任何 `use<..>` 边界都必须包含该 trait 的所有泛型参数，包括 trait 的隐式 `Self` 泛型类型参数。

## 泛型与返回位置 `impl Trait` 的区别

在参数位置，`impl Trait` 在语义上与泛型类型参数非常相似。然而，在返回位置上两者有显著区别。使用 `impl Trait` 时，与泛型类型参数不同，是函数选择返回类型，调用者不能选择返回类型。

以下函数：

```rust
# trait Trait {}
fn foo<T: Trait>() -> T {
    // ...
# panic!()
}
```

允许调用者决定返回类型 `T`，函数返回该类型。

以下函数：

```rust
# trait Trait {}
# impl Trait for () {}
fn foo() -> impl Trait {
    // ...
}
```

不允许调用者决定返回类型。相反，函数选择返回类型，但只承诺它将实现 `Trait`。

r[type.impl-trait.constraint]
## 限制

`impl Trait` 只能作为非 `extern` 函数的参数或返回类型出现。它不能是 `let` 绑定的类型、字段类型，也不能出现在类型别名中。

[`use<..>` bound]: ../trait-bounds.md#use-bounds
[closures]: closure.md
[trait object]: trait-object.md
