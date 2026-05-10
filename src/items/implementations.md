r[items.impl]
# 实现

r[items.impl.syntax]
```grammar,items
Implementation -> InherentImpl | TraitImpl

InherentImpl ->
    `impl` GenericParams? Type WhereClause? `{`
        InnerAttribute*
        AssociatedItem*
    `}`

TraitImpl ->
    `unsafe`? `impl` GenericParams? `!`? TypePath `for` Type
    WhereClause?
    `{`
        InnerAttribute*
        AssociatedItem*
    `}`
```

r[items.impl.intro]
*实现*是将程序项与*实现类型*关联起来的程序项。实现用关键字 `impl` 定义，包含属于被实现类型的实例或静态地属于该类型的函数。

r[items.impl.kinds]
有两种类型的实现：

- 固有实现
- [trait] 实现

r[items.impl.inherent]
## 固有实现 {#inherent-implementations}

r[items.impl.inherent.intro]
固有实现的定义是：`impl` 关键字、泛型类型声明、指向具名类型的路径、where 子句以及一对花括号内的一组可关联程序项。

r[items.impl.inherent.implementing-type]
具名类型称为*实现类型*，可关联程序项是实现类型的*关联程序项*。

r[items.impl.inherent.associated-items]
固有实现将包含的程序项与实现类型关联起来。

r[items.impl.inherent.associated-items.allowed-items]
固有实现可以包含[关联函数][associated functions]（包括[方法][methods]）和[关联常量][associated constants]。

r[items.impl.inherent.type-alias]
它们不能包含关联类型别名。

r[items.impl.inherent.associated-item-path]
关联程序项的[路径][path]是指向实现类型的任意路径，后跟关联程序项的标识符作为最后的路径组件。

r[items.impl.inherent.coherence]
一个类型也可以有多个固有实现。实现类型必须与原始类型定义在同一个 crate 中定义。

``` rust
pub mod color {
    pub struct Color(pub u8, pub u8, pub u8);

    impl Color {
        pub const WHITE: Color = Color(255, 255, 255);
    }
}

mod values {
    use super::color::Color;
    impl Color {
        pub fn red() -> Color {
            Color(255, 0, 0)
        }
    }
}

pub use self::color::Color;
fn main() {
    // 实际路径到同一模块中的实现类型和 impl。
    color::Color::WHITE;

    // 不同模块中的 impl 块仍然通过指向类型的路径来访问。
    color::Color::red();

    // 指向实现类型的重导出路径也有效。
    Color::red();

    // 无效，因为 `values` 中的 use 不是 pub。
    // values::Color::red();
}
```

r[items.impl.trait]
## trait 实现 {#trait-implementations}

r[items.impl.trait.intro]
*trait 实现*的定义方式与固有实现类似，不同之处在于可选的泛型类型声明后跟一个 [trait]，再后跟关键字 `for`，再后跟指向具名类型的路径。

<!-- 要理解这一点，你必须回顾前一小节。:( -->

r[items.impl.trait.implemented-trait]
trait 称为*被实现的 trait*。实现类型实现被实现的 trait。

r[items.impl.trait.def-requirement]
trait 实现必须定义被实现的 trait 声明的所有非默认关联程序项，可以重新定义被实现的 trait 定义的默认关联程序项，并且不能定义任何其他程序项。

r[items.impl.trait.associated-item-path]
关联程序项的路径是 `<` 后跟指向实现类型的路径，再后跟 `as`，再后跟指向 trait 的路径，再后跟 `>` 作为路径组件，再后跟关联程序项的路径组件。

r[items.impl.trait.safety]
[Unsafe trait][Unsafe traits] 要求 trait 实现以 `unsafe` 关键字开头。

```rust
# #[derive(Copy, Clone)]
# struct Point {x: f64, y: f64};
# type Surface = i32;
# struct BoundingBox {x: f64, y: f64, width: f64, height: f64};
# trait Shape { fn draw(&self, s: Surface); fn bounding_box(&self) -> BoundingBox; }
# fn do_draw_circle(s: Surface, c: Circle) { }
struct Circle {
    radius: f64,
    center: Point,
}

impl Copy for Circle {}

impl Clone for Circle {
    fn clone(&self) -> Circle { *self }
}

impl Shape for Circle {
    fn draw(&self, s: Surface) { do_draw_circle(s, *self); }
    fn bounding_box(&self) -> BoundingBox {
        let r = self.radius;
        BoundingBox {
            x: self.center.x - r,
            y: self.center.y - r,
            width: 2.0 * r,
            height: 2.0 * r,
        }
    }
}
```

r[items.impl.trait.coherence]
### trait 实现的一致性 {#trait-implementation-coherence}

r[items.impl.trait.coherence.intro]
如果孤儿规则检查失败或存在重叠的实现实例，则 trait 实现被认为是不一致的。

r[items.impl.trait.coherence.overlapping]
当实现所针对的 trait 的交集非空，并且这些实现可以用相同的类型实例化时，两个 trait 实现会重叠。<!-- 这可能不对？来源：没有两个实现可以用相同的输入类型参数集进行实例化。 -->

r[items.impl.trait.orphan-rule]
#### 孤儿规则 {#orphan-rules}

r[items.impl.trait.orphan-rule.intro]
*孤儿规则*规定，只有 trait 或实现中的至少一个类型在当前 crate 中定义时，才允许该 trait 实现。它防止了跨不同 crate 的冲突 trait 实现，是确保一致性的关键。

孤儿实现是指为外部类型实现外部 trait 的实现。如果这些被自由允许，两个 crate 可能会以不兼容的方式为同一类型实现相同的 trait，从而造成添加或更新依赖项可能因冲突实现而导致编译中断的情况。

孤儿规则使库作者能够为其 trait 添加新的实现，而不必担心会破坏下游代码。如果没有这些限制，库就不能添加 `impl<T: Display> MyTrait for T` 这样的实现，而不会与下游实现产生潜在冲突。

r[items.impl.trait.orphan-rule.def]
给定 `impl<P1..=Pn> Trait<T1..=Tn> for T0`，只有当以下至少一项成立时，`impl` 才有效：

- `Trait` 是[本地 trait][local trait]
- 以下全部：
  - 类型 `T0..=Tn` 中至少有一个必须是[本地类型][local type]。令 `Ti` 为第一个这样的类型。
  - 在 `T0..Ti`（不包括 `Ti`）中不能出现[未覆盖类型][uncovered type]参数 `P1..=Pn`

r[items.impl.trait.uncovered-param]
只有*未覆盖*类型参数的出现受限。

r[items.impl.trait.fundamental]
请注意，就一致性而言，[基本类型][fundamental types]是特殊的。`Box<T>` 中的 `T` 不被视为已覆盖，而 `Box<LocalType>` 被视为本地类型。

r[items.impl.generics]
## 泛型实现 {#generic-implementations}

r[items.impl.generics.intro]
实现可以接受[泛型参数][generic parameters]，这些参数可以在实现的其余部分中使用。实现参数直接写在 `impl` 关键字之后。

```rust
# trait Seq<T> { fn dummy(&self, _: T) { } }
impl<T> Seq<T> for Vec<T> {
    /* ... */
}
impl Seq<bool> for u32 {
    /* 将整数视为位序列 */
}
```

r[items.impl.generics.use]
如果泛型参数至少出现在以下一项中，则该泛型参数*约束*该实现：

* 被实现的 trait，如果有的话
* 实现类型
* 作为包含另一个约束该实现的参数的类型的[约束][bounds]中的[关联类型][associated type]

r[items.impl.generics.constrain]
类型参数和 const 参数必须始终约束该实现。生命周期参数如果用于关联类型中则必须约束该实现。

约束情况的示例：

```rust
# trait Trait{}
# trait GenericTrait<T> {}
# trait HasAssocType { type Ty; }
# struct Struct;
# struct GenericStruct<T>(T);
# struct ConstGenericStruct<const N: usize>([(); N]);
// T 作为 GenericTrait 的参数来约束。
impl<T> GenericTrait<T> for i32 { /* ... */ }

// T 作为 GenericStruct 的参数来约束
impl<T> Trait for GenericStruct<T> { /* ... */ }

// 类似地，N 作为 ConstGenericStruct 的参数来约束
impl<const N: usize> Trait for ConstGenericStruct<N> { /* ... */ }

// T 通过在类型 `U` 的约束中的关联类型来约束，
// `U` 本身是约束该 trait 的泛型参数。
impl<T, U> GenericTrait<U> for u32 where U: HasAssocType<Ty = T> { /* ... */ }

// 与上面类似，只是类型是 `(U, isize)`。`U` 出现在
// 包含 `T` 的类型内部，而不是该类型本身。
impl<T, U> GenericStruct<U> where (U, isize): HasAssocType<Ty = T> { /* ... */ }
```

非约束情况的示例：

```rust,compile_fail
// 以下都是错误，因为它们具有不约束的类型或 const 参数。

// T 不约束，因为它根本没有出现。
impl<T> Struct { /* ... */ }

// N 不约束，原因相同。
impl<const N: usize> Struct { /* ... */ }

// 在实现内部使用 T 不约束该 impl。
impl<T> Struct {
    fn uses_t(t: &T) { /* ... */ }
}

// T 在 U 的约束中用作关联类型，但 U 不约束。
impl<T, U> Struct where U: HasAssocType<Ty = T> { /* ... */ }

// T 用于约束中，但不是作为关联类型，因此不约束。
impl<T, U> GenericTrait<U> for u32 where U: GenericTrait<T> {}
```

允许的不约束生命周期参数示例：

```rust
# struct Struct;
impl<'a> Struct {}
```

不允许的不约束生命周期参数示例：

```rust,compile_fail
# struct Struct;
# trait HasAssocType { type Ty; }
impl<'a> HasAssocType for Struct {
    type Ty = &'a Struct;
}
```

r[items.impl.attributes]
## 实现上的属性 {#attributes-on-implementations}

实现可以在 `impl` 关键字之前包含外部[属性][attributes]，在包含关联程序项的花括号内包含内部[属性][attributes]。内部属性必须在任何关联程序项之前。此处有意义的属性有 [`cfg`]、[`deprecated`]、[`doc`] 和 [lint 检查属性][the lint check attributes]。

[trait]: traits.md
[associated constants]: associated-items.md#associated-constants
[associated functions]: associated-items.md#associated-functions-and-methods
[associated type]: associated-items.md#associated-types
[attributes]: ../attributes.md
[bounds]: ../trait-bounds.md
[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../rustdoc/the-doc-attribute.html
[generic parameters]: generics.md
[methods]: associated-items.md#methods
[path]: ../paths.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[Unsafe traits]: traits.md#unsafe-traits
[local trait]: ../glossary.md#local-trait
[local type]: ../glossary.md#local-type
[fundamental types]: ../glossary.md#fundamental-type-constructors
[uncovered type]: ../glossary.md#uncovered-type
