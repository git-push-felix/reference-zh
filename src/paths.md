r[paths]
# 路径

r[paths.intro]
*路径*是由 `::` 分隔的一个或多个路径段组成的序列。路径用于引用[程序项][items]、值、[类型][types]、[宏][macros]和[属性][attributes]。

仅由标识符段组成的简单路径的两个示例：

<!-- ignore: syntax fragment -->
```rust,ignore
x;
x::y::z;
```

## 路径的类型

r[paths.simple]
### 简单路径

r[paths.simple.syntax]
```grammar,paths
SimplePath ->
    `::`? SimplePathSegment (`::` SimplePathSegment)*

SimplePathSegment ->
    IDENTIFIER | `super` | `self` | `crate` | `$crate`
```

r[paths.simple.intro]
简单路径用于[可见性][visibility]标记、[属性][attributes]、[宏][mbe]和 [`use`] 项。例如：

```rust
use std::io::{self, Write};
mod m {
    #[clippy::cyclomatic_complexity = "0"]
    pub (in super) fn f1() {}
}
```

r[paths.expr]
### 表达式中的路径

r[paths.expr.syntax]
```grammar,paths
PathInExpression ->
    `::`? PathExprSegment (`::` PathExprSegment)*

PathExprSegment ->
    PathIdentSegment (`::` GenericArgs)?

PathIdentSegment ->
    IDENTIFIER | `super` | `self` | `Self` | `crate` | `$crate`

GenericArgs ->
      `<` `>`
    | `<` ( GenericArg `,` )* GenericArg `,`? `>`

GenericArg ->
    Lifetime | Type | GenericArgsConst | GenericArgsBinding | GenericArgsBounds

GenericArgsConst ->
      BlockExpression
    | LiteralExpression
    | `-` LiteralExpression
    | SimplePathSegment

GenericArgsBinding ->
    IDENTIFIER GenericArgs? `=` Type

GenericArgsBounds ->
    IDENTIFIER GenericArgs? `:` Bounds
```

r[paths.expr.intro]
表达式中的路径允许指定带有泛型参数的路径。它们用于[表达式][expressions]和[模式][patterns]中的各种位置。

r[paths.expr.turbofish]
在泛型参数的开头 `<` 之前需要 `::`，以避免与小于运算符歧义。这俗称"turbofish"语法。

```rust
(0..10).collect::<Vec<_>>();
Vec::<u8>::with_capacity(1024);
```

r[paths.expr.argument-order]
泛型参数的顺序限制为生命周期参数，然后是类型参数，然后是常量参数，然后是等式约束。

r[paths.expr.complex-const-params]
常量参数必须用花括号包围，除非它们是[字面量][literal]、[推断常量][inferred const]或单段路径。[推断常量][inferred const]不能使用花括号包围。

```rust
mod m {
    pub const C: usize = 1;
}
const C: usize = m::C;
fn f<const N: usize>() -> [u8; N] { [0; N] }

let _ = f::<1>(); // 字面量。
let _: [_; 1] = f::<_>(); // 推断常量。
let _: [_; 1] = f::<(((_)))>(); // 推断常量。
let _ = f::<C>(); // 单段路径。
let _ = f::<{ m::C }>(); // 多段路径必须使用花括号。
```

```rust,compile_fail
fn f<const N: usize>() -> [u8; N] { [0; _] }
let _: [_; 1] = f::<{ _ }>();
//                    ^ 错误：此处不允许 `_`
```

> [!NOTE]
> 在泛型参数列表中，[推断常量][inferred const]被解析为[推断类型][InferredType]，但随后在语义上被视为一种单独的[常量泛型参数][const generic argument]。

r[paths.expr.impl-trait-params]
对应于 `impl Trait` 类型的合成类型参数是隐式的，不能显式指定。

r[paths.qualified]
## 限定路径

r[paths.qualified.syntax]
```grammar,paths
QualifiedPathInExpression -> QualifiedPathType (`::` PathExprSegment)+

QualifiedPathType -> `<` Type (`as` TypePath)? `>`

QualifiedPathInType -> QualifiedPathType (`::` TypePathSegment)+
```

r[paths.qualified.intro]
完全限定路径允许消除 [trait 实现][trait implementations]的路径歧义，并允许指定[规范路径](#规范路径)。在类型规范中使用时，它支持使用下面指定的类型语法。

```rust
struct S;
impl S {
    fn f() { println!("S"); }
}
trait T1 {
    fn f() { println!("T1 f"); }
}
impl T1 for S {}
trait T2 {
    fn f() { println!("T2 f"); }
}
impl T2 for S {}
S::f();  // 调用固有 impl。
<S as T1>::f();  // 调用 T1 trait 函数。
<S as T2>::f();  // 调用 T2 trait 函数。
```

r[paths.type]
### 类型中的路径

r[paths.type.syntax]
```grammar,paths
TypePath -> `::`? TypePathSegment (`::` TypePathSegment)*

TypePathSegment -> PathIdentSegment (`::`? (GenericArgs | TypePathFn))?

TypePathFn -> `(` TypePathFnInputs? `)` (`->` TypeNoBounds)?

TypePathFnInputs -> Type (`,` Type)* `,`?
```

r[paths.type.intro]
类型路径用于类型定义、trait 约束和限定路径中。

r[paths.type.turbofish]
尽管在泛型参数之前允许使用 `::`，但不是必需的，因为这里不像 [PathInExpression] 中存在歧义。

```rust
# mod ops {
#     pub struct Range<T> {f1: T}
#     pub trait Index<T> {}
#     pub struct Example<'a> {f1: &'a i32}
# }
# struct S;
impl ops::Index<ops::Range<usize>> for S { /*...*/ }
fn i<'a>() -> impl Iterator<Item = ops::Example<'a>> {
    // ...
#    const EXAMPLE: Vec<ops::Example<'static>> = Vec::new();
#    EXAMPLE.into_iter()
}
type G = std::boxed::Box<dyn std::ops::FnOnce(isize) -> isize>;
```

r[paths.qualifiers]
## 路径限定符

路径可以使用各种前导限定符来表示，以改变其解析方式的含义。

> [!NOTE]
> [`use` 声明][use declarations]对 `self`、`super`、`crate` 和 `$crate` 有额外的行为和限制。

r[paths.qualifiers.global-root]
### `::`

r[paths.qualifiers.global-root.intro]
以 `::` 开头的路径被视为*全局路径*，路径段的解析起点根据版次不同而有所差异。路径中的每个标识符必须解析为一个项。

r[paths.qualifiers.global-root.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，标识符从"crate 根"（2018 版次中的 `crate::`）解析，其中包含各种不同的项，包括外部 crate、默认 crate 如 `std` 或 `core`，以及 crate 顶层的项（包括 `use` 导入）。
>
> 从 2018 版次开始，以 `::` 开头的路径从[外部 crate 预导入][extern prelude]中的 crate 解析。也就是说，它们必须后跟一个 crate 的名称。

```rust
pub fn foo() {
    // 在 2018 版次中，这通过外部 crate 预导入访问 `std`。
    // 在 2015 版次中，这通过 crate 根访问 `std`。
    let now = ::std::time::Instant::now();
    println!("{:?}", now);
}
```

```rust,edition2015
// 2015 版次
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        ::a::foo(); // 调用 `a` 的 foo 函数
        // 在 Rust 2018 中，`::a` 将被解释为 crate `a`。
    }
}
# fn main() {}
```

r[paths.qualifiers.mod-self]
### `self`

r[paths.qualifiers.mod-self.intro]
`self` 将路径相对于当前模块解析。

r[paths.qualifiers.mod-self.restriction]
`self` 只能作为路径的第一个段（前面没有 `::`）或最后一个段（前面有 `::`）使用。

r[paths.qualifiers.mod-self.trailing]
当 `self` 作为路径的最后一个段出现时，它引用前面段命名的实体。前面的路径必须解析为[模块][module]、[枚举][enumeration]或 [trait]。

```rust
mod m {
    pub enum E { V1 }
    pub trait Tr {}
    pub(in crate::m::self) fn g() {} // 正确：模块可以是 `self` 的父级。
}
type Ty = m::E::self; // 正确：枚举可以是 `self` 的父级。
fn f<T: m::Tr::self>() {} // 正确：trait 可以是 `self` 的父级。
# fn main() { let _: Ty = m::E::V1; }
```

```rust,compile_fail,E0223
struct S;
type Ty = S::self; // 错误：结构体不能是 `self` 的父级。
# fn main() {}
```

> [!NOTE]
> 有关 `use` 声明中 `self` 的附加规则，请参见 [items.use.self]。

r[paths.qualifiers.self-pat]
在方法体中，仅由单个 `self` 段组成的路径解析为方法的 self 参数。

```rust
fn foo() {}
fn bar() {
    self::foo();
}
struct S(bool);
impl S {
  fn baz(self) {
        self.0;
    }
}
# fn main() {}
```

r[paths.qualifiers.type-self]
### `Self`

r[paths.qualifiers.type-self.intro]
`Self`（大写 "S"）用于引用当前正在实现或定义的类型。它可以在以下情况下使用：

r[paths.qualifiers.type-self.trait]
* 在 [trait] 定义中，它引用实现该 trait 的类型。

r[paths.qualifiers.type-self.impl]
* 在[实现][implementation]中，它引用正在实现的类型。当实现元组或单元[结构体][struct]时，它也引用[值命名空间][value namespace]中的构造函数。

r[paths.qualifiers.type-self.type]
* 在 [struct]、[枚举][enumeration] 或 [union] 的定义中，它引用正在定义的类型。定义不允许无限递归（必须存在间接引用）。

r[paths.qualifiers.type-self.scope]
`Self` 的作用域行为类似于泛型参数；更多细节请参见 [`Self` 作用域][`Self` scope]部分。

r[paths.qualifiers.type-self.allowed-positions]
`Self` 只能作为第一个段使用，前面不能有 `::`。

r[paths.qualifiers.type-self.no-generics]
`Self` 路径不能包含泛型参数（如 `Self::<i32>`）。

```rust
trait T {
    type Item;
    const C: i32;
    // `Self` 将是实现 `T` 的任何类型。
    fn new() -> Self;
    // `Self::Item` 将是实现中的类型别名。
    fn f(&self) -> Self::Item;
}
struct S;
impl T for S {
    type Item = i32;
    const C: i32 = 9;
    fn new() -> Self {           // `Self` 是类型 `S`。
        S
    }
    fn f(&self) -> Self::Item {  // `Self::Item` 是类型 `i32`。
        Self::C                  // `Self::C` 是常量值 `9`。
    }
}

// `Self` 在 trait 定义的泛型中处于作用域中，
// 用于引用正在定义的类型。
trait Add<Rhs = Self> {
    type Output;
    // `Self` 也可以引用正在实现的类型的关联项。
    fn add(self, rhs: Rhs) -> Self::Output;
}

struct NonEmptyList<T> {
    head: T,
    // 结构体可以引用自身（只要不是无限递归）。
    tail: Option<Box<Self>>,
}
```

r[paths.qualifiers.super]
### `super`

r[paths.qualifiers.super.intro]
路径中的 `super` 解析为父模块。

r[paths.qualifiers.super.allowed-positions]
它只能用于路径的前导段中，可能位于初始 `self` 段之后。

```rust
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        super::a::foo(); // 调用 a 的 foo 函数
    }
}
# fn main() {}
```

r[paths.qualifiers.super.repetition]
`super` 可以在第一个 `super` 或 `self` 之后重复多次，以引用祖先模块。

```rust
mod a {
    fn foo() {}

    mod b {
        mod c {
            fn foo() {
                super::super::foo(); // 调用 a 的 foo 函数
                self::super::super::foo(); // 调用 a 的 foo 函数
            }
        }
    }
}
# fn main() {}
```

r[paths.qualifiers.crate]
### `crate`

r[paths.qualifiers.crate.intro]
`crate` 将路径相对于当前 crate 解析。

r[paths.qualifiers.crate.allowed-positions]
`crate` 只能作为第一个段使用，前面不能有 `::`。

```rust
fn foo() {}
mod a {
    fn bar() {
        crate::foo();
    }
}
# fn main() {}
```

r[paths.qualifiers.macro-crate]
### `$crate`

r[paths.qualifiers.macro-crate.allowed-positions]
[`$crate`] 仅在[宏转录器][macro transcribers]中使用，并且只能作为第一个段使用，前面不能有 `::`。

r[paths.qualifiers.macro-crate.hygiene]
[`$crate`] 将展开为访问定义该宏的 crate 顶层项的路径，无论该宏是在哪个 crate 中被调用的。

```rust
pub fn increment(x: u32) -> u32 {
    x + 1
}

#[macro_export]
macro_rules! inc {
    ($x:expr) => ( $crate::increment($x) )
}
# fn main() { }
```

r[paths.canonical]
## 规范路径

r[paths.canonical.intro]
在模块或实现中定义的每个项都有一个*规范路径*，对应于其在其 crate 内的定义位置。

r[paths.canonical.alias]
所有这些项的其他路径都是别名。

r[paths.canonical.def]
规范路径定义为*路径前缀*加上该项本身定义的路径段。

r[paths.canonical.non-canonical]
[实现][implementations]和 [use 声明][use declarations]没有规范路径，尽管实现定义的项有规范路径。在块表达式中定义的项没有规范路径。在没有规范路径的模块中定义的项没有规范路径。在引用没有规范路径的项的实现中定义的关联项（例如作为实现类型、被实现的 trait、类型参数或类型参数上的约束）没有规范路径。

r[paths.canonical.module-prefix]
模块的路径前缀是该模块的规范路径。

r[paths.canonical.bare-impl-prefix]
对于裸实现，它是被实现项的规范路径，用<span class="parenthetical">尖括号（`<>`）</span>包围。

r[paths.canonical.trait-impl-prefix]
对于 [trait 实现][trait implementations]，它是被实现项的规范路径后跟 `as`，再后跟 trait 的规范路径，全部用<span class="parenthetical">尖括号（`<>`）</span>包围。

r[paths.canonical.local-canonical-path]
规范路径仅在给定 crate 内有意义。跨 crate 没有全局命名空间；项的规范路径仅在其 crate 内标识它。

```rust
// 注释显示该项的规范路径。

mod a { // crate::a
    pub struct Struct; // crate::a::Struct

    pub trait Trait { // crate::a::Trait
        fn f(&self); // crate::a::Trait::f
    }

    impl Trait for Struct {
        fn f(&self) {} // <crate::a::Struct as crate::a::Trait>::f
    }

    impl Struct {
        fn g(&self) {} // <crate::a::Struct>::g
    }
}

mod without { // crate::without
    fn canonicals() { // crate::without::canonicals
        struct OtherStruct; // 无

        trait OtherTrait { // 无
            fn g(&self); // 无
        }

        impl OtherTrait for OtherStruct {
            fn g(&self) {} // 无
        }

        impl OtherTrait for crate::a::Struct {
            fn g(&self) {} // 无
        }

        impl crate::a::Trait for OtherStruct {
            fn f(&self) {} // 无
        }
    }
}

# fn main() {}
```

[`$crate`]: macro.decl.hygiene.crate
[implementations]: items/implementations.md
[items]: items.md
[literal]: expressions/literal-expr.md
[use declarations]: items/use-declarations.md
[`Self` scope]: names/scopes.md#self-scope
[`use`]: items/use-declarations.md
[attributes]: attributes.md
[const generic argument]: items.generics.const.argument
[enumeration]: items/enumerations.md
[expressions]: expressions.md
[extern prelude]: names/preludes.md#extern-prelude
[implementation]: items/implementations.md
[inferred const]: items.generics.const.inferred
[macro transcribers]: macros-by-example.md
[macros]: macros.md
[mbe]: macros-by-example.md
[module]: items/modules.md
[patterns]: patterns.md
[struct]: items/structs.md
[trait implementations]: items/implementations.md#trait-implementations
[trait]: items/traits.md
[traits]: items/traits.md
[types]: types.md
[union]: items/unions.md
[`use` declarations]: items/use-declarations.md
[value namespace]: names/namespaces.md
[visibility]: visibility-and-privacy.md
