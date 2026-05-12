<!-- template:attributes -->
r[attributes.derive]
# 派生

r[attributes.derive.intro]
*`derive` [属性][attributes]* 调用一个或多个[派生宏][derive macros]，允许为数据结构自动生成新的[程序项][items]。你可以使用[过程宏][procedural macros]创建 `derive` 宏。

> [!EXAMPLE]
> [`PartialEq`][macro@PartialEq] 派生宏为 `Foo<T> where T: PartialEq` 生成 [`PartialEq`] 的[实现][implementation]。[`Clone`][macro@Clone] 派生宏类似地为 [`Clone`] 生成实现。
>
> ```rust
> #[derive(PartialEq, Clone)]
> struct Foo<T> {
>     a: i32,
>     b: T,
> }
> ```
>
> 生成的 `impl` 项等价于：
>
> ```rust
> # struct Foo<T> { a: i32, b: T }
> impl<T: PartialEq> PartialEq for Foo<T> {
>     fn eq(&self, other: &Foo<T>) -> bool {
>         self.a == other.a && self.b == other.b
>     }
> }
>
> impl<T: Clone> Clone for Foo<T> {
>     fn clone(&self) -> Self {
>         Foo { a: self.a.clone(), b: self.b.clone() }
>     }
> }
> ```

r[attributes.derive.syntax]
`derive` 属性使用 [MetaListPaths] 语法来指定要调用的[派生宏][derive macros]的路径列表。

r[attributes.derive.allowed-positions]
`derive` 属性只能应用于[结构体][items.struct]、[枚举][items.enum]和[联合体][items.union]。

r[attributes.derive.duplicates]
`derive` 属性可以在一个项上使用任意次数。所有属性中列出的所有派生宏都会被调用。

r[attributes.derive.stdlib]
`derive` 属性在标准库中导出为：

- [`core::derive`]
- [`std::derive`]
- [`core::prelude::v1::derive`]
- [`std::prelude::v1::derive`]

r[attributes.derive.built-in]
内置派生宏定义在[语言预导入][names.preludes.lang]中。内置派生宏的列表如下：

- [`Clone`]
- [`Copy`]
- [`Debug`]
- [`Default`]
- [`Eq`]
- [`Hash`]
- [`Ord`]
- [`PartialEq`]
- [`PartialOrd`]

r[attributes.derive.built-in-automatically_derived]
内置派生宏在其生成的实现中包含 [`automatically_derived` 属性][attributes.derive.automatically_derived]。

r[attributes.derive.behavior]
在宏展开期间，对于派生列表中的每个元素，相应的派生宏展开为零个或多个[程序项][items]。

<!-- template:attributes -->
r[attributes.derive.automatically_derived]
## `automatically_derived` 属性 {#the-automatically_derived-attribute}

r[attributes.derive.automatically_derived.intro]
*`automatically_derived` [属性][attributes]* 用于标注一个[实现][implementation]，以表明它是由[派生宏][derive macro]自动创建的。它没有直接影响，但可以被工具和诊断 lint 用于检测这些自动生成的实现。

> [!EXAMPLE]
> 给定 `struct Example` 上的 [`#[derive(Clone)]`][macro@Clone]，[派生宏][derive macro]可能产生：
>
> ```rust
> # struct Example;
> #[automatically_derived]
> impl ::core::clone::Clone for Example {
>     #[inline]
>     fn clone(&self) -> Self {
>         Example
>     }
> }
> ```

r[attributes.derive.automatically_derived.syntax]
`automatically_derived` 属性使用 [MetaWord] 语法。

r[attributes.derive.automatically_derived.allowed-positions]
`automatically_derived` 属性只能应用于[实现][implementation]。

> [!NOTE]
> `rustc` 忽略其他位置的用法但会发出 lint 警告。这可能在将来成为错误。

r[attributes.derive.automatically_derived.duplicates]
在一个实现上多次使用 `automatically_derived` 的效果与使用一次相同。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。

r[attributes.derive.automatically_derived.behavior]
`automatically_derived` 属性没有行为。

[items]: ../items.md
[derive macro]: macro.proc.derive
[derive macros]: macro.proc.derive
[implementation]: ../items/implementations.md
[items]: ../items.md
[procedural macros]: macro.proc.derive
