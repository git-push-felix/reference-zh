r[names.scopes]
# 作用域

r[names.scopes.intro]
*作用域*是源文本中可以以其名称引用某命名[实体][entity]的区域。以下各节提供了作用域规则和行为的详细信息，这些取决于实体的类型及其声明位置。名称如何解析为实体的过程在[名称解析][name resolution]章节中描述。有关用于运行析构函数的"丢弃作用域"的更多信息，请参见[析构函数][destructors]章节。

r[names.scopes.items]
## 项作用域

r[names.scopes.items.module]
直接在[模块][module]中声明的[项][items]的名称具有从模块开头延伸到模块末尾的作用域。这些项也是模块的成员，可以通过从其模块开始的[路径][path]引用。

r[names.scopes.items.statement]
作为[语句][statement]声明的项的名称具有从该语句所在块的开头延伸到块末尾的作用域。

r[names.scopes.items.duplicate]
在同一模块或块内引入与同[命名空间][namespace]中另一个项名称重复的项是错误的。[星号 glob 导入][Asterisk glob imports]对处理重复名称和遮蔽有特殊行为，更多细节请参见链接的章节。

r[names.scopes.items.shadow-prelude]
模块中的项可以遮蔽[预导入](#预导入作用域)中的项。

r[names.scopes.items.nested-modules]
外部模块的项名称不在嵌套模块的作用域中。可以使用[路径][path]引用另一个模块中的项。

r[names.scopes.associated-items]
### 关联项作用域

r[names.scopes.associated-items.scope]
[关联项][Associated items]没有作用域，只能通过从它们关联的类型或 trait 开始的[路径][path]引用。[方法][methods]也可以通过[调用表达式][call expressions]引用。

r[names.scopes.associated-items.duplicate]
类似于模块或块内的项，在 trait 或实现中引入与同一命名空间中 trait 或 impl 中另一个项重复的项是错误的。

r[names.scopes.pattern-bindings]
## 模式绑定作用域

局部变量[模式][pattern]绑定的作用域取决于它使用的上下文：

r[names.scopes.pattern-bindings.let]
* [`let` 语句][`let` statement]绑定从 `let` 语句之后开始，直到声明它的块结束。
r[names.scopes.pattern-bindings.parameter]
* [函数参数][Function parameter]绑定在函数体内部。
r[names.scopes.pattern-bindings.closure]
* [闭包参数][Closure parameter]绑定在闭包体内部。
r[names.scopes.pattern-bindings.loop]
* [`for`] 绑定在循环体内部。
r[names.scopes.pattern-bindings.let-chains]
* [`if let`] 和 [`while let`] 绑定在后续条件以及结果块中有效。
r[names.scopes.pattern-bindings.match-arm]
* [`match` 分支][`match` arms]绑定在[匹配守卫][match guard]和匹配分支表达式内部。
r[names.scopes.pattern-bindings.match-guard-let]
* [`match` 守卫 `let`][`match` guard `let`] 绑定在后续守卫条件和匹配分支表达式中有效。

r[names.scopes.pattern-bindings.items]
局部变量作用域不扩展到项声明内部。
<!-- 不完全是，参见 https://github.com/rust-lang/rust/issues/33118 -->

### 模式绑定遮蔽

r[names.scopes.pattern-bindings.shadow]
模式绑定允许遮蔽作用域中的任何名称，但以下情况会导致错误：

* [常量泛型参数][Const generic parameters]
* [静态项][Static items]
* [常量项][Const items]
* [结构体][structs]和[枚举][enums]的构造函数

以下示例说明了局部绑定如何遮蔽项声明：

```rust
fn shadow_example() {
    // 由于作用域中还没有局部变量，这将解析为函数。
    foo(); // 打印 `function`
    let foo = || println!("closure");
    fn foo() { println!("function"); }
    // 这将解析为局部闭包，因为它遮蔽了该项。
    foo(); // 打印 `closure`
}
```

r[names.scopes.generic-parameters]
## 泛型参数作用域

r[names.scopes.generic-parameters.param-list]
泛型参数在 [GenericParams] 列表中声明。泛型参数的作用域在其声明的项内部。

r[names.scopes.generic-parameters.order-independent]
所有参数都在泛型参数列表中的作用域内，无论声明顺序如何。以下展示了一些在声明之前引用参数的示例：

```rust
// 'b 约束在其声明之前被引用。
fn params_scope<'a: 'b, 'b>() {}

# trait SomeTrait<const Z: usize> {}
// 常量 N 在其声明之前在 trait 约束中被引用。
fn f<T: SomeTrait<N>, const N: usize>() {}
```

r[names.scopes.generic-parameters.bounds]
泛型参数也在类型约束和 where 子句的作用域中，例如：

```rust
# trait SomeTrait<'a, T> {}
// `SomeTrait` 的 `<'a, U>` 引用 `bounds_scope` 的 'a 和 U 参数。
fn bounds_scope<'a, T: SomeTrait<'a, U>, U>() {}

fn where_scope<'a, T, U>()
    where T: SomeTrait<'a, U>
{}
```

r[names.scopes.generic-parameters.inner-items]
在函数内部声明的[项][items]引用其外部作用域的泛型参数是错误的。

```rust,compile_fail
fn example<T>() {
    fn inner(x: T) {} // 错误：不能使用外部函数的泛型参数
}
```

### 泛型参数遮蔽

r[names.scopes.generic-parameters.shadow]
遮蔽泛型参数是错误的，但函数内部声明的项允许遮蔽该函数的泛型参数名称。

```rust
fn example<'a, T, const N: usize>() {
    // 函数内部的项允许遮蔽作用域中的泛型参数。
    fn inner_lifetime<'a>() {} // 正确
    fn inner_type<T>() {} // 正确
    fn inner_const<const N: usize>() {} // 正确
}
```

```rust,compile_fail
trait SomeTrait<'a, T, const N: usize> {
    fn example_lifetime<'a>() {} // 错误：'a 已在使用中
    fn example_type<T>() {} // 错误：T 已在使用中
    fn example_const<const N: usize>() {} // 错误：N 已在使用中
    fn example_mixed<const T: usize>() {} // 错误：T 已在使用中
}
```

r[names.scopes.lifetimes]
### 生命周期作用域

生命周期参数在 [GenericParams] 列表和[高阶 trait 约束][hrtb]中声明。

r[names.scopes.lifetimes.special]
`'static` 生命周期和[占位生命周期][placeholder lifetime] `'_` 具有特殊含义，不能作为参数声明。

#### 生命周期泛型参数作用域

r[names.scopes.lifetimes.generic]
[常量][Constant]和[静态][static]项以及 [const 上下文][const contexts] 仅允许 `'static` 生命周期引用，因此没有其他生命周期可以在它们的作用域中。[关联常量][Associated consts]确实允许引用在其 trait 或实现中声明的生命周期。

#### 高阶 trait 约束作用域

r[names.scopes.lifetimes.higher-ranked]
声明为[高阶 trait 约束][hrtb]的生命周期参数的作用域取决于其使用场景。

* 作为 [TypeBoundWhereClauseItem]，声明的生命周期在类型和类型约束的作用域中。
* 作为 [TraitBound]，声明的生命周期在约束类型路径的作用域中。
* 作为 [BareFunctionType]，声明的生命周期在函数参数和返回类型的作用域中。

```rust
# trait Trait<'a>{}

fn where_clause<T>()
    // 'a 在类型和类型约束中都在作用域中。
    where for <'a> &'a T: Trait<'a>
{}

fn bound<T>()
    // 'a 在约束内部的作用域中。
    where T: for <'a> Trait<'a>
{}

# struct Example<'a> {
#     field: &'a u32
# }

// 'a 在参数和返回类型中都在作用域中。
type FnExample = for<'a> fn(x: Example<'a>) -> Example<'a>;
```

#### impl trait 限制

r[names.scopes.lifetimes.impl-trait]
[Impl trait] 类型只能引用在函数或实现上声明的生命周期。

<!-- 无法演示作用域错误，因为编译器会 panic
     https://github.com/rust-lang/rust/issues/67830
-->
```rust
# trait Trait1 {
#     type Item;
# }
# trait Trait2<'a> {}
#
# struct Example;
#
# impl Trait1 for Example {
#     type Item = Element;
# }
#
# struct Element;
# impl<'a> Trait2<'a> for Element {}
#
// 此处的 `impl Trait2` 不允许引用 'b，但允许引用 'a。
fn foo<'a>() -> impl for<'b> Trait1<Item = impl Trait2<'a> + use<'a>> {
    // ...
#    Example
}
```

r[names.scopes.loop-label]
## 循环标签作用域

r[names.scopes.loop-label.scope]
[循环标签][Loop labels]可以由[循环表达式][loop expression]声明。循环标签的作用域从它被声明的点开始直到循环表达式的末尾。作用域不会扩展到[项][items]、[闭包][closures]、[async 块][async blocks]、[常量参数][const arguments]、[const 上下文][const contexts]以及定义它的 [`for` 循环][`for` loop]的迭代器表达式中。

```rust
'a: for n in 0..3 {
    if n % 2 == 0 {
        break 'a;
    }
    fn inner() {
        // 在此处使用 'a 会是错误。
        // break 'a;
    }
}

// 标签在 `while` 循环的表达式中处于作用域中。
'a: while break 'a {}         // 循环不运行。
'a: while let _ = break 'a {} // 循环不运行。

// 标签在定义它的 `for` 循环中不在作用域中：
'a: for outer in 0..5 {
    // 这将中断外部循环，跳过内部循环并停止外部循环。
    'a: for inner in { break 'a; 0..1 } {
        println!("{}", inner); // 这不会运行。
    }
    println!("{}", outer); // 这也不会运行。
}

```

r[names.scopes.loop-label.shadow]
循环标签可以遮蔽外部作用域中相同名称的标签。对标签的引用指向最接近的定义。

```rust
// 循环标签遮蔽示例。
'a: for outer in 0..5 {
    'a: for inner in 0..5 {
        // 这将终止内部循环，但外部循环继续运行。
        break 'a;
    }
}
```

r[names.scopes.prelude]
## 预导入作用域

r[names.scopes.prelude.intro]
[预导入][Preludes]将实体带入每个模块的作用域。这些实体不是模块的成员，而是在[名称解析][name resolution]期间被隐式查询。

r[names.scopes.prelude.shadow]
预导入名称可以被模块中的声明遮蔽。

r[names.scopes.prelude.layers]
预导入是分层的，如果一个预导入包含与另一个预导入同名的实体，则一个可以遮蔽另一个。预导入可能遮蔽其他预导入的顺序如下，其中较前的条目可以遮蔽较后的条目：

1. [外部 crate 预导入][Extern prelude]
2. [工具预导入][Tool prelude]
3. [`macro_use` 预导入][`macro_use` prelude]
4. [标准库预导入][Standard library prelude]
5. [语言预导入][Language prelude]

r[names.scopes.macro_rules]
## `macro_rules` 作用域

`macro_rules` 宏的作用域在[声明宏][Macros By Example]章节中描述。行为取决于 [`macro_use`] 和 [`macro_export`] 属性的使用。

r[names.scopes.derive]
## 派生宏辅助属性

r[names.scopes.derive.scope]
[派生宏辅助属性][Derive macro helper attributes]在其对应 [`derive` 属性][`derive` attribute]指定的项中处于作用域中。作用域从 `derive` 属性之后开始直到该项目的末尾。<!-- 注意：不严格正确，参见 https://github.com/rust-lang/rust/issues/79202，但这是预期行为。 -->

r[names.scopes.derive.shadow]
辅助属性遮蔽作用域中同名的其他属性。

r[names.scopes.self]
## `Self` 作用域

r[names.scopes.self.intro]
尽管 [`Self`] 是具有特殊含义的关键字，但它与名称解析的交互方式类似于普通名称。

r[names.scopes.self.def-scope]
在 [struct]、[enum]、[union]、[trait] 或[实现][implementation]的定义中，隐式的 `Self` 类型被视为类似于[泛型参数](#泛型参数作用域)，并以与泛型类型参数相同的方式处于作用域中。

r[names.scopes.self.impl-scope]
在[实现][implementation]的值[命名空间][namespace]中，隐式的 `Self` 构造函数在实现体（实现的[关联项][associated items]）内部处于作用域中。

```rust
// 结构体定义中的 Self 类型。
struct Recursive {
    f1: Option<Box<Self>>
}

// 泛型参数中的 Self 类型。
struct SelfGeneric<T: Into<Self>>(T);

// 实现中的 Self 值构造函数。
struct ImplExample();
impl ImplExample {
    fn example() -> Self { // Self 类型
        Self() // Self 值构造函数
    }
}
```

[`derive` attribute]: ../attributes/derive.md
[`for` loop]: ../expressions/loop-expr.md#iterator-loops
[`for`]: ../expressions/loop-expr.md#iterator-loops
[`if let`]: ../expressions/if-expr.md#if-let-patterns
[`while let`]: ../expressions/loop-expr.md#while-let-patterns
[`let` statement]: ../statements.md#let-statements
[`macro_export`]: ../macros-by-example.md#the-macro_export-attribute
[`macro_use` prelude]: preludes.md#macro_use-prelude
[`macro_use`]: ../macros-by-example.md#the-macro_use-attribute
[`match` arms]: ../expressions/match-expr.md
[`match` guard `let`]: expr.match.guard.let
[`Self`]: ../paths.md#self-1
[Associated consts]: ../items/associated-items.md#associated-constants
[associated items]: ../items/associated-items.md
[Asterisk glob imports]: ../items/use-declarations.md
[async blocks]: ../expressions/block-expr.md#async-blocks
[call expressions]: ../expressions/call-expr.md
[Closure parameter]: ../expressions/closure-expr.md
[closures]: ../expressions/closure-expr.md
[const arguments]: ../items/generics.md#const-generics
[const contexts]: ../const_eval.md#const-context
[Const generic parameters]: ../items/generics.md#const-generics
[Const items]: ../items/constant-items.md
[Constant]: ../items/constant-items.md
[Derive macro helper attributes]: ../procedural-macros.md#derive-macro-helper-attributes
[destructors]: ../destructors.md
[entity]: ../names.md
[enum]: ../items/enumerations.mdr
[enums]: ../items/enumerations.md
[Extern prelude]: preludes.md#extern-prelude
[Function parameter]: ../items/functions.md#function-parameters
[hrtb]: ../trait-bounds.md#higher-ranked-trait-bounds
[Impl trait]: ../types/impl-trait.md
[implementation]: ../items/implementations.md
[items]: ../items.md
[Language prelude]: preludes.md#language-prelude
[loop expression]: ../expressions/loop-expr.md
[Loop labels]: ../expressions/loop-expr.md#loop-labels
[Macros By Example]: ../macros-by-example.md
[match guard]: ../expressions/match-expr.md#match-guards
[methods]: ../items/associated-items.md#methods
[module]: ../items/modules.md
[name resolution]: name-resolution.md
[namespace]: namespaces.md
[path]: ../paths.md
[pattern]: ../patterns.md
[placeholder lifetime]: ../lifetime-elision.md
[preludes]: preludes.md
[Standard library prelude]: preludes.md#standard-library-prelude
[statement]: ../statements.md
[Static items]: ../items/static-items.md
[static]: ../items/static-items.md
[struct]: ../items/structs.md
[structs]: ../items/structs.md
[Tool prelude]: preludes.md#tool-prelude
[trait]: ../items/traits.md
[union]: ../items/unions.md
