r[type]
# 类型

r[type.intro]
Rust 程序中的每个变量、项和值都有一个类型。*值*的*类型*定义了其内存的解释方式以及可对该值执行的操作。

r[type.builtin]
内置类型与语言紧密集成，以非常规的方式实现，用户定义类型无法模拟。

r[type.user-defined]
用户定义类型的功能有限。

r[type.kinds]
类型的分类如下：

* 原始类型：
    * [布尔型][Boolean] --- `bool`
    * [数值型][Numeric] --- 整数和浮点数
    * [`char`]
    * [`str`]
    * [never 类型][Never] --- `!` --- 没有值的类型
* 序列类型：
    * [元组][Tuple]
    * [数组][Array]
    * [切片][Slice]
* 用户定义类型：
    * [结构体][Struct]
    * [枚举][Enum]
    * [联合体][Union]
* 函数类型：
    * [函数][Functions]
    * [闭包][Closures]
* 指针类型：
    * [引用][References]
    * [裸指针][Raw pointers]
    * [函数指针][Function pointers]
* Trait 类型：
    * [Trait 对象][Trait objects]
    * [impl Trait][Impl trait]

r[type.name]
## 类型表达式

r[type.name.syntax]
```grammar,types
Type ->
      TypeNoBounds
    | ImplTraitType
    | TraitObjectType

TypeNoBounds ->
      ParenthesizedType
    | ImplTraitTypeOneBound
    | TraitObjectTypeOneBound
    | TypePath
    | TupleType
    | NeverType
    | RawPointerType
    | ReferenceType
    | ArrayType
    | SliceType
    | InferredType
    | QualifiedPathInType
    | BareFunctionType
    | MacroInvocation
```

r[type.name.intro]
*类型表达式*是上述 [Type] 语法规则所定义的引用类型的语法。它可以引用：

r[type.name.sequence]
* 序列类型（[元组][tuple]、[数组][array]、[切片][slice]）。

r[type.name.path]
* [类型路径][Type paths]，可以引用：
    * 原始类型（[布尔型][boolean]、[数值型][numeric]、[`char`]、[`str`]）。
    * 指向某个[项][item]的路径（[结构体][struct]、[枚举][enum]、[联合体][union]、[类型别名][type alias]、[trait]）。
    * [`Self` 路径][`Self` path]，其中 `Self` 是实现类型。
    * 泛型[类型参数][type parameters]。

r[type.name.pointer]
* 指针类型（[引用][reference]、[裸指针][raw pointer]、[函数指针][function pointer]）。

r[type.name.inference]
* [推断类型][inferred type]，请求编译器来确定类型。

r[type.name.grouped]
* [括号][Parentheses]，用于消除歧义。

r[type.name.trait]
* Trait 类型：[Trait 对象][Trait objects] 和 [impl trait][Impl trait]。

r[type.name.never]
* [never][never] 类型。

r[type.name.macro-expansion]
* [宏][Macros]，展开为一个类型表达式。

r[type.name.parenthesized]
### 括号类型

r[type.name.parenthesized.syntax]
```grammar,types
ParenthesizedType -> `(` Type `)`
```

r[type.name.parenthesized.intro]
在某些情况下，类型的组合可能存在歧义。在类型周围使用括号可以消除歧义。例如，[引用类型][reference type] 中的 `+` 运算符对[类型边界][type boundaries]的作用域可能不明确，因此必须使用括号。需要此消歧方式的语法规则使用 [TypeNoBounds] 规则而非 [Type][grammar-Type]。

```rust
# use std::any::Any;
type T<'a> = &'a (dyn Any + Send);
```

r[type.recursive]
## 递归类型

r[type.recursive.intro]
名义类型------[结构体][structs]、[枚举][enumerations]和[联合体][unions]------可以是递归的。也就是说，每个 `enum` 变体或 `struct` 或 `union` 字段可以直接或间接地引用外围的 `enum` 或 `struct` 类型自身。

r[type.recursive.constraint]
此类递归有若干限制：

* 递归类型必须包含一个名义类型在递归中（不仅仅是[类型别名][type aliases]或其他结构类型如[数组][arrays]或[元组][tuples]）。因此 `type Rec = &'static [Rec]` 是不允许的。
* 递归类型的大小必须是有限的；换句话说，类型的递归字段必须是[指针类型][pointer types]。

*递归*类型的一个示例及其用法：

```rust
enum List<T> {
    Nil,
    Cons(T, Box<List<T>>)
}

let a: List<i32> = List::Cons(7, Box::new(List::Cons(13, Box::new(List::Nil))));
```

[`char`]: types/char.md
[`str`]: types/str.md
[Array]: types/array.md
[Boolean]: types/boolean.md
[Closures]: types/closure.md
[Enum]: types/enum.md
[Function pointers]: types/function-pointer.md
[Functions]: types/function-item.md
[Impl trait]: types/impl-trait.md
[Macros]: macros.md
[Numeric]: types/numeric.md
[Parentheses]: #parenthesized-types
[Raw pointers]: types/pointer.md#raw-pointers-const-and-mut
[References]: types/pointer.md#shared-references-
[Slice]: types/slice.md
[Struct]: types/struct.md
[Trait objects]: types/trait-object.md
[Tuple]: types/tuple.md
[Type paths]: paths.md#paths-in-types
[Union]: types/union.md
[`Self` path]: paths.md#self-1
[arrays]: types/array.md
[enumerations]: types/enum.md
[function pointer]: types/function-pointer.md
[inferred type]: types/inferred.md
[item]: items.md
[never]: types/never.md
[pointer types]: types/pointer.md
[raw pointer]: types/pointer.md#raw-pointers-const-and-mut
[reference type]: types/pointer.md#shared-references-
[reference]: types/pointer.md#shared-references-
[structs]: types/struct.md
[trait]: types/trait-object.md
[tuples]: types/tuple.md
[type alias]: items/type-aliases.md
[type aliases]: items/type-aliases.md
[type boundaries]: trait-bounds.md
[type parameters]: types/parameters.md
[unions]: types/union.md
[enum]: types/enum.md
[boolean]: types/boolean.md
[numeric]: types/numeric.md
[struct]: types/struct.md
[slice]: types/slice.md
[tuple]: types/tuple.md
[array]: types/array.md
[union]: types/union.md
