r[names.namespaces]
# 命名空间

r[names.namespaces.intro]
*命名空间*是已声明[名称][names]的逻辑分组。名称根据其引用的实体类型被隔离到不同的命名空间中。命名空间允许一个命名空间中的名称出现不会与另一个命名空间中的相同名称冲突。

存在多个不同的命名空间，每个命名空间包含不同类型的实体。名称的使用将根据上下文在不同的命名空间中查找该名称的声明，如[名称解析][name resolution]章节所述。

r[names.namespaces.kinds]
以下是命名空间及其对应实体的列表：

* 类型命名空间
    * [模块声明][Module declarations]
    * [外部 crate 声明][External crate declarations]
    * [外部 crate 预导入][External crate prelude]项
    * [Struct]、[union]、[enum]、enum 变体声明
    * [Trait 项声明][Trait item declarations]
    * [类型别名][Type aliases]
    * [关联类型声明][Associated type declarations]
    * 内置类型：[布尔][boolean]、[数值][numeric]、[`char`] 和 [`str`]
    * [泛型类型参数][Generic type parameters]
    * [`Self` 类型][`Self` type]
    * [工具属性模块][Tool attribute modules]
* 值命名空间
    * [函数声明][Function declarations]
    * [常量项声明][Constant item declarations]
    * [静态项声明][Static item declarations]
    * [结构体构造函数][Struct constructors]
    * [枚举变体构造函数][Enum variant constructors]
    * [`Self` 构造函数][`Self` constructors]
    * [泛型常量参数][Generic const parameters]
    * [关联常量声明][Associated const declarations]
    * [关联函数声明][Associated function declarations]
    * 局部绑定 --- [`let`]、[`if let`]、[`while let`]、[`for`]、[`match`] 分支、[函数参数][function parameters]、[闭包参数][closure parameters]
    * 捕获的[闭包][closure]变量
* 宏命名空间
    * [`macro_rules` 声明][`macro_rules` declarations]
    * [内置属性][Built-in attributes]
    * [工具属性][Tool attributes]
    * [类函数过程宏][Function-like procedural macros]
    * [派生宏][Derive macros]
    * [派生宏辅助属性][Derive macro helpers]
    * [属性宏][Attribute macros]
* 生命周期命名空间
    * [泛型生命周期参数][Generic lifetime parameters]
* 标签命名空间
    * [循环标签][Loop labels]
    * [块标签][Block labels]

一个示例，展示不同命名空间中重叠的名称如何可以被无歧义地使用：

```rust
// Foo 在类型命名空间中引入一个类型，在值命名空间中引入一个构造函数。
struct Foo(u32);

// `Foo` 宏声明在宏命名空间中。
macro_rules! Foo {
    () => {};
}

// `f` 参数类型中的 `Foo` 引用类型命名空间中的 `Foo`。
// `'Foo` 在生命周期命名空间中引入一个新的生命周期。
fn example<'Foo>(f: Foo) {
    // `Foo` 引用值命名空间中的 `Foo` 构造函数。
    let ctor = Foo;
    // `Foo` 引用宏命名空间中的 `Foo` 宏。
    Foo!{}
    // `'Foo` 在标签命名空间中引入一个标签。
    'Foo: loop {
        // `'Foo` 引用 `'Foo` 生命周期参数，`Foo`
        // 引用类型命名空间。
        let x: &'Foo Foo;
        // `'Foo` 引用标签。
        break 'Foo;
    }
}
```

r[names.namespaces.without]
## 没有命名空间的命名实体

以下实体具有显式名称，但这些名称不属于任何特定的命名空间。

### 字段

r[names.namespaces.without.fields]
尽管结构体、枚举和联合体字段是有名称的，但这些命名字段不存在于显式的命名空间中。它们只能通过[字段表达式][field expression]访问，该表达式仅检查所访问特定类型的字段名称。

### Use 声明

r[names.namespaces.without.use]
[use 声明][use declaration]具有它导入作用域的命名别名，但 `use` 项本身不属于特定命名空间。相反，它可以根据导入的项类型将别名引入多个命名空间。

r[names.namespaces.sub-namespaces]
## 子命名空间

r[names.namespaces.sub-namespaces.intro]
宏命名空间分为两个子命名空间：一个用于[叹号风格宏][bang-style macros]，一个用于[属性][attributes]。解析属性时，作用域中的任何叹号风格宏将被忽略。反之，解析叹号风格宏时，将忽略作用域中的属性宏。这防止了一种风格遮蔽另一种风格。

例如，[`cfg` 属性][`cfg` attribute]和 [`cfg` 宏][`cfg` macro]是宏命名空间中两个具有相同名称的不同实体，但它们仍可在各自的上下文中使用。

<!-- ignore: requires external crates -->
> [!NOTE]
> `use` 导入仍不能在模块或块中创建相同名称的重复绑定，无论子命名空间如何。
>
> ```rust,ignore
> #[macro_export]
> macro_rules! mymac {
>     () => {};
> }
>
> use myattr::mymac; // error[E0252]: 名称 `mymac` 被多次定义。
> ```

[`cfg` attribute]: ../conditional-compilation.md#the-cfg-attribute
[`cfg` macro]: ../conditional-compilation.md#the-cfg-macro
[`char`]: ../types/char.md
[`for`]: ../expressions/loop-expr.md#iterator-loops
[`if let`]: ../expressions/if-expr.md#if-let-patterns
[`let`]: ../statements.md#let-statements
[`macro_rules` declarations]: ../macros-by-example.md
[`match`]: ../expressions/match-expr.md
[`Self` constructors]: ../paths.md#self-1
[`Self` type]: ../paths.md#self-1
[`str`]: ../types/str.md
[`use` import]: ../items/use-declarations.md
[`while let`]: ../expressions/loop-expr.md#while-let-patterns
[Associated const declarations]: ../items/associated-items.md#associated-constants
[Associated function declarations]: ../items/associated-items.md#associated-functions-and-methods
[Associated type declarations]: ../items/associated-items.md#associated-types
[Attribute macros]: ../procedural-macros.md#the-proc_macro_attribute-attribute
[attributes]: ../attributes.md
[bang-style macros]: ../macros.md
[Block labels]: expr.loop.block-labels
[boolean]: ../types/boolean.md
[Built-in attributes]: ../attributes.md#built-in-attributes-index
[closure parameters]: ../expressions/closure-expr.md
[closure]: ../expressions/closure-expr.md
[Constant item declarations]: ../items/constant-items.md
[Derive macro helpers]: ../procedural-macros.md#derive-macro-helper-attributes
[Derive macros]: macro.proc.derive
[entity]: ../glossary.md#entity
[Enum variant constructors]: ../items/enumerations.md
[enum]: ../items/enumerations.md
[External crate declarations]: ../items/extern-crates.md
[External crate prelude]: preludes.md#extern-prelude
[field expression]: ../expressions/field-expr.md
[Function declarations]: ../items/functions.md
[function parameters]: ../items/functions.md#function-parameters
[Function-like procedural macros]: ../procedural-macros.md#the-proc_macro-attribute
[Generic const parameters]: ../items/generics.md#const-generics
[Generic lifetime parameters]: ../items/generics.md
[Generic type parameters]: ../items/generics.md
[Loop labels]: ../expressions/loop-expr.md#loop-labels
[Module declarations]: ../items/modules.md
[name resolution]: name-resolution.md
[names]: ../names.md
[numeric]: ../types/numeric.md
[Static item declarations]: ../items/static-items.md
[Struct constructors]: ../items/structs.md
[Struct]: ../items/structs.md
[Tool attribute modules]: ../attributes.md#tool-attributes
[Tool attributes]: ../attributes.md#tool-attributes
[Trait item declarations]: ../items/traits.md
[Type aliases]: ../items/type-aliases.md
[union]: ../items/unions.md
[use declaration]: ../items/use-declarations.md
