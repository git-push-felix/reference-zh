r[items.generics]
# 泛型参数

r[items.generics.syntax]
```grammar,items
GenericParams -> `<` ( GenericParam (`,` GenericParam)* `,`? )? `>`

GenericParam -> OuterAttribute* ( LifetimeParam | TypeParam | ConstParam )

LifetimeParam -> Lifetime ( `:` LifetimeBounds )?

TypeParam -> IDENTIFIER ( `:` Bounds? )? ( `=` Type )?

ConstParam ->
    `const` IDENTIFIER `:` Type
    ( `=` ( BlockExpression | IDENTIFIER | `-`?LiteralExpression ) )?
```

r[items.generics.syntax.intro]
[函数][Functions]、[类型别名][type aliases]、[结构体][structs]、[枚举][enumerations]、[联合体][unions]、[trait][traits]和[实现][implementations]可以由类型、常量和生命周期来*参数化*。这些参数列在尖<span class="parenthetical">括号（`<...>`）</span>中，通常紧接在程序项名称之后、定义之前。对于没有名称的实现，它们紧随 `impl` 之后。

r[items.generics.syntax.decl-order]
泛型参数的顺序限制为先生命周期参数，然后类型参数和 const 参数交错排列。

r[items.generics.syntax.duplicate-params]
同一参数名称不能在 [GenericParams] 列表中声明多次。

一些带有类型、const 和生命周期参数的程序项示例：

```rust
fn foo<'a, T>() {}
trait A<U> {}
struct Ref<'a, T> where T: 'a { r: &'a T }
struct InnerArray<T, const N: usize>([T; N]);
struct EitherOrderWorks<const N: bool, U>(U);
```

r[items.generics.syntax.scope]
泛型参数在声明它们的程序项定义内处于作用域中。它们对于函数体内声明的程序项不在作用域中，如[程序项声明][item declarations]中所述。有关更多细节，请参见[泛型参数作用域][generic parameter scopes]。

r[items.generics.builtin-generic-types]
[引用][References]、[原始指针][raw pointers]、[数组][arrays]、[切片][slices]、[元组][tuples]和[函数指针][function pointers]也具有生命周期或类型参数，但不使用路径语法引用。

r[items.generics.invalid-lifetimes]
`'_` 和 `'static` 不是有效的生命周期参数名称。

r[items.generics.const]
### const 泛型 {#const-generics}

r[items.generics.const.intro]
*const 泛型参数*允许程序项在常量值上是泛型的。

r[items.generics.const.namespace]
const 标识符在[值命名空间][value namespace]中为常量参数引入一个名称，并且该程序项的所有实例都必须用给定类型的值进行实例化。

r[items.generics.const.allowed-types]
const 参数唯一允许的类型是 `u8`、`u16`、`u32`、`u64`、`u128`、`usize`、`i8`、`i16`、`i32`、`i64`、`i128`、`isize`、`char` 和 `bool`。

r[items.generics.const.use]
const 参数可以在任何可以使用 [const 项][const item]的地方使用，但当在[类型][type]或[数组重复表达式][array repeat expression]中使用时，它必须是独立的（如下所述）。也就是说，它们允许在以下位置使用：

1. 作为构成相关程序项签名的任何类型的应用 const。
2. 作为用于定义[关联常量][associated const]的 const 表达式的一部分，或作为[关联类型][associated type]的参数。
3. 作为程序项中任何函数主体内任何运行时表达式中的值。
4. 作为程序项中任何函数主体内使用的任何类型的参数。
5. 作为程序项中任何字段的类型的一部分。

```rust
// const 泛型参数可以使用的示例。

// 用在程序项本身的签名中。
fn foo<const N: usize>(arr: [i32; N]) {
    // 在函数体内用作类型。
    let x: [i32; N];
    // 用作表达式。
    println!("{}", N * 2);
}

// 用作结构体的字段。
struct Foo<const N: usize>([i32; N]);

impl<const N: usize> Foo<N> {
    // 用作关联常量。
    const CONST: usize = N * 4;
}

trait Trait {
    type Output;
}

impl<const N: usize> Trait for Foo<N> {
    // 用作关联类型。
    type Output = [i32; N];
}
```

```rust,compile_fail
// const 泛型参数不能使用的示例。
fn foo<const N: usize>() {
    // 不能在函数体内的程序项定义中使用。
    const BAD_CONST: [usize; N] = [1; N];
    static BAD_STATIC: [usize; N] = [1; N];
    fn inner(bad_arg: [usize; N]) {
        let bad_value = N * 2;
    }
    type BadAlias = [usize; N];
    struct BadStruct([usize; N]);
}
```

r[items.generics.const.standalone]
作为进一步的限制，const 参数只能作为独立参数出现在[类型][type]或[数组重复表达式][array repeat expression]内部。在这些上下文中，它们只能用作单段[路径表达式][path expression]，可能在一个[块][block]内部（如 `N` 或 `{N}`）。也就是说，它们不能与其他表达式组合。

```rust,compile_fail
// const 参数不能使用的示例。

// 不允许在类型中与其他表达式组合，如此处
// 返回类型中的算术表达式。
fn bad_function<const N: usize>() -> [u8; {N + 1}] {
    // 对于数组重复表达式也是如此。
    [1; {N + 1}]
}
```

r[items.generics.const.argument]
[路径][path]中的 const 实参指定用于该程序项的 const 值。

r[items.generics.const.argument.const-expr]
实参必须是[推断 const][inferred const] 或属于分配给 const 参数类型的[常量表达式][const expression]。除非是单路径段（[IDENTIFIER]）或[字面量][literal]（可以带有前导 `-` 标记），否则常量表达式必须是一个[块表达式][block]（用花括号包围）。

> [!NOTE]
> 这种语法限制是必要的，以避免在解析类型内的表达式时需要无限前瞻。

```rust
struct S<const N: i64>;
const C: i64 = 1;
fn f<const N: i64>() -> S<N> { S }

let _ = f::<1>(); // 字面量。
let _ = f::<-1>(); // 负数字面量。
let _ = f::<{ 1 + 2 }>(); // 常量表达式。
let _ = f::<C>(); // 单段路径。
let _ = f::<{ C + 1 }>(); // 常量表达式。
let _: S<1> = f::<_>(); // 推断 const。
let _: S<1> = f::<(((_)))>(); // 推断 const。
```

> [!NOTE]
> 在泛型实参列表中，[推断 const][inferred const] 被解析为[推断类型][InferredType]，但在语义上被当作一种单独的 [const 泛型实参][const generic argument]处理。

r[items.generics.const.inferred]
在需要 const 实参的地方，可以使用 `_`（可选地由任意数量的匹配括号包围），称为*推断 const*（[路径规则][paths.expr.complex-const-params]，[数组表达式规则][expr.array.length-restriction]）。这会要求编译器在可能的情况下根据周围信息推断 const 实参。

```rust
fn make_buf<const N: usize>() -> [u8; N] {
    [0; _]
    //  ^ 推断 `N`。
}
let _: [u8; 1024] = make_buf::<_>();
//                             ^ 推断 `1024`。
```

> [!NOTE]
> [推断 const][inferred const] 在语义上不是[表达式][Expression]，因此在花括号内不被接受。
>
> ```rust,compile_fail
> fn f<const N: usize>() -> [u8; N] { [0; _] }
> let _: [_; 1] = f::<{ _ }>();
> //                    ^ 错误：此处不允许 `_`
> ```

r[items.generics.const.inferred.constraint]
推断 const 不能在程序项签名中使用。

```rust,compile_fail
fn f<const N: usize>(x: [u8; N]) -> [u8; _] { x }
//                                       ^ 错误：不允许
```

r[items.generics.const.type-ambiguity]
当存在歧义时，如果泛型实参可能被解析为类型或 const 实参，则始终解析为类型。将实参放在块表达式中可以强制将其解释为 const 实参。

<!-- TODO: Rewrite the paragraph above to be in terms of namespaces, once namespaces are introduced, and it is clear which namespace each parameter lives in. -->

```rust,compile_fail
type N = u32;
struct Foo<const N: usize>;
// 以下是错误，因为 `N` 被解释为类型别名 `N`。
fn foo<const N: usize>() -> Foo<N> { todo!() } // 错误
// 可以通过用花括号包裹来修复，以强制将其解释为
// const 参数 `N`：
fn bar<const N: usize>() -> Foo<{ N }> { todo!() } // ok
```

r[items.generics.const.variance]
与类型和生命周期参数不同，const 参数可以在参数化的程序项外部声明而不被使用，但[泛型实现][generic implementations]中描述的实现除外：

```rust,compile_fail
// ok
struct Foo<const N: usize>;
enum Bar<const M: usize> { A, B }

// 错误：未使用的参数
struct Baz<T>;
struct Biz<'a>;
struct Unconstrained;
impl<const N: usize> Unconstrained {}
```

r[items.generics.const.exhaustiveness]
在解决 trait 约束义务时，在确定约束是否满足时不考虑 const 参数的所有实现的穷尽性。例如，在以下代码中，即使 `bool` 类型的所有可能的 const 值都已实现，trait 约束仍未满足，这仍然是一个错误：

```rust,compile_fail
struct Foo<const B: bool>;
trait Bar {}
impl Bar for Foo<true> {}
impl Bar for Foo<false> {}

fn needs_bar(_: impl Bar) {}
fn generic<const B: bool>() {
    let v = Foo::<B>;
    needs_bar(v); // 错误：trait 约束 `Foo<B>: Bar` 不满足
}
```

r[items.generics.where]
## where 子句 {#where-clauses}

r[items.generics.where.syntax]
```grammar,items
WhereClause -> `where` ( WhereClauseItem `,` )* WhereClauseItem?

WhereClauseItem ->
      LifetimeWhereClauseItem
    | TypeBoundWhereClauseItem

LifetimeWhereClauseItem -> Lifetime `:` LifetimeBounds

TypeBoundWhereClauseItem -> ForLifetimes? Type `:` Bounds?
```

r[items.generics.where.intro]
*where 子句*提供了另一种为类型和生命周期参数指定约束的方式，以及为非类型参数的类型指定约束的方式。

r[items.generics.where.higher-ranked-lifetimes]
`for` 关键字可用于引入[高阶生命周期][higher-ranked lifetimes]。它只允许 [LifetimeParam] 参数。

```rust
struct A<T>
where
    T: Iterator,            // 也可以用 A<T: Iterator>
    T::Item: Copy,          // 对关联类型的约束
    String: PartialEq<T>,   // 对 `String` 的约束，使用类型参数
    i32: Default,           // 允许，但没有用
{
    f: T,
}
```

r[items.generics.attributes]
## 属性 {#attributes}

泛型生命周期和类型参数上允许使用[属性][attributes]。[attributes]。此位置没有执行任何操作的内置属性，但自定义 derive 属性可能赋予其意义。

此示例展示了使用自定义 derive 属性来修改泛型参数的含义。

<!-- ignore: requires proc macro derive -->
```rust,ignore
// 假设 MyFlexibleClone 的 derive 声明了 `my_flexible_clone` 为
// 它能理解的属性。
#[derive(MyFlexibleClone)]
struct Foo<#[my_flexible_clone(unbounded)] H> {
    a: *const H
}
```

[array repeat expression]: ../expressions/array-expr.md
[arrays]: ../types/array.md
[slices]: ../types/slice.md
[associated const]: associated-items.md#associated-constants
[associated type]: associated-items.md#associated-types
[attributes]: ../attributes.md
[block]: ../expressions/block-expr.md
[const contexts]: ../const_eval.md#const-context
[const expression]: ../const_eval.md#constant-expressions
[const generic argument]: items.generics.const.argument
[const item]: constant-items.md
[enumerations]: enumerations.md
[functions]: functions.md
[function pointers]: ../types/function-pointer.md
[generic implementations]: implementations.md#generic-implementations
[generic parameter scopes]: ../names/scopes.md#generic-parameter-scopes
[higher-ranked lifetimes]: ../trait-bounds.md#higher-ranked-trait-bounds
[implementations]: implementations.md
[inferred const]: items.generics.const.inferred
[item declarations]: ../statements.md#item-declarations
[item]: ../items.md
[literal]: ../expressions/literal-expr.md
[path]: ../paths.md
[path expression]: ../expressions/path-expr.md
[raw pointers]: ../types/pointer.md#raw-pointers-const-and-mut
[references]: ../types/pointer.md#shared-references-
[structs]: structs.md
[tuples]: ../types/tuple.md
[trait object]: ../types/trait-object.md
[traits]: traits.md
[type aliases]: type-aliases.md
[type]: ../types.md
[unions]: unions.md
[value namespace]: ../names/namespaces.md
