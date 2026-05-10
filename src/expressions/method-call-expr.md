r[expr.method]
# 方法调用表达式

r[expr.method.syntax]
```grammar,expressions
MethodCallExpression -> Expression `.` PathExprSegment `(`CallParams? `)`
```

r[expr.method.intro]
*方法调用*由一个表达式（*接收者*）后跟一个点号、一个表达式路径段和一个带括号的表达式列表组成。

r[expr.method.target]
方法调用被解析为特定 trait 上的关联[方法]，如果左侧的精确 `self`-类型已知，则静态分派到方法；如果左侧表达式是间接的 [trait 对象](../types/trait-object.md)，则动态分派。

```rust
let pi: Result<f32, _> = "3.14".parse();
let log_pi = pi.unwrap_or(1.0).log(2.72);
# assert!(1.14 < log_pi && log_pi < 1.15)
```

r[expr.method.autoref-deref]
在查找方法调用时，接收者可能会被自动解引用或借用以调用某个方法。这需要一个比其他函数更复杂的查找过程，因为可能有多种可能的方法可调用。使用以下过程：

r[expr.method.candidate-receivers]
第一步是构建一个候选接收者类型列表。通过重复[解引用][dereference]接收者表达式的类型来获取这些类型，将遇到的每个类型添加到列表中，然后最后尝试一次数组[非固定大小强制转换]，如果成功则添加结果类型。

r[expr.method.candidate-receivers-refs]
然后，对于每个候选类型 `T`，立即在其后添加 `&T` 和 `&mut T`。

例如，如果接收者的类型为 `Box<[i32;2]>`，则候选类型将为 `Box<[i32;2]>`、`&Box<[i32;2]>`、`&mut Box<[i32;2]>`、`[i32; 2]`（通过解引用）、`&[i32; 2]`、`&mut [i32; 2]`、`[i32]`（通过非固定大小强制转换）、`&[i32]`，最后是 `&mut [i32]`。

r[expr.method.candidate-search]
然后，对于每个候选类型 `T`，在以下位置搜索具有该类型接收者的[可见][visible]方法：

1. `T` 的固有方法（直接在 `T` 上实现的方法）。
1. 由 `T` 实现的[可见][visible] trait 提供的任何方法。如果 `T` 是类型参数，首先查找 `T` 上的 trait 约束提供的方法。然后查找作用域内的所有其他方法。

> [!NOTE]
> 查找按顺序对每个类型进行，这偶尔会导致令人惊讶的结果。下面的代码将打印 "In trait impl!"，因为首先查找 `&self` 方法，在找到结构体的 `&mut self` 方法之前就找到了 trait 方法。
>
> ```rust
> struct Foo {}
>
> trait Bar {
>   fn bar(&self);
> }
>
> impl Foo {
>   fn bar(&mut self) {
>     println!("In struct impl!")
>   }
> }
>
> impl Bar for Foo {
>   fn bar(&self) {
>     println!("In trait impl!")
>   }
> }
>
> fn main() {
>   let mut f = Foo{};
>   f.bar();
> }
> ```

r[expr.method.ambiguous-target]
如果这导致多个可能的候选，则这是一个错误，必须将接收者[转换][disambiguate call]为适当的接收者类型来进行方法调用。

r[expr.method.receiver-constraints]
此过程不考虑接收者的可变性或生命周期，也不考虑方法是否是 `unsafe`。一旦查找到方法，如果由于这些原因之一（或多个）而无法调用它，结果将是编译错误。

r[expr.method.ambiguous-search]
如果到达某一步时存在多个可能的方法，例如泛型方法或 trait 被视为相同，则这是编译错误。这些情况需要[消歧函数调用语法]来进行方法和函数调用。

r[expr.method.edition2021]
> [!EDITION-2021]
> 在 2021 版本之前，在搜索可见方法的过程中，如果候选接收者类型是[数组类型]，则标准库 [`IntoIterator`] trait 提供的方法会被忽略。
>
> 为此目的使用的版本由表示方法名称的记号确定。
>
> 这种特殊情况将来可能会被移除。

> [!WARNING]
> 对于 [trait 对象]，如果存在与 trait 方法同名的固有方法，则在方法调用表达式中尝试调用该方法时将给出编译错误。相反，你可以使用[消歧函数调用语法]调用该方法，在这种情况下，它调用的是 trait 方法，而不是固有方法。无法调用固有方法。只要不在 trait 对象上定义与 trait 方法同名的固有方法，就不会有问题。

[visible]: ../visibility-and-privacy.md
[array type]: ../types/array.md
[trait objects]: ../types/trait-object.md
[disambiguate call]: call-expr.md#disambiguating-function-calls
[disambiguating function call syntax]: call-expr.md#disambiguating-function-calls
[dereference]: operator-expr.md#the-dereference-operator
[methods]: ../items/associated-items.md#methods
[unsized coercion]: ../type-coercions.md#unsized-coercions
[`IntoIterator`]: std::iter::IntoIterator
