r[items]
# 程序项

r[items.syntax]
```grammar,items
Item ->
    OuterAttribute* ( VisItem | MacroItem )

VisItem ->
    Visibility?
    (
        Module
      | ExternCrate
      | UseDeclaration
      | Function
      | TypeAlias
      | Struct
      | Enumeration
      | Union
      | ConstantItem
      | StaticItem
      | Trait
      | Implementation
      | ExternBlock
    )

MacroItem ->
      MacroInvocationSemi
    | MacroRulesDefinition
```

r[items.intro]
*程序项*是 crate 的组成部分。程序项在 crate 内通过嵌套的[模块][modules]集合来组织。每个 crate 都有一个最外层的匿名模块，crate 内的所有其他程序项都在该模块树中拥有自己的[路径][paths]。

r[items.static-def]
程序项在编译时完全确定，通常在执行期间保持不变，并且可以驻留在只读内存中。

r[items.kinds]
有以下几种程序项：

* [模块][modules]
* [`extern crate` 声明][`extern crate` declarations]
* [`use` 声明][`use` declarations]
* [函数定义][function definitions]
* [类型别名定义][type alias definitions]
* [结构体定义][struct definitions]
* [枚举定义][enumeration definitions]
* [联合体定义][union definitions]
* [常量项][constant items]
* [静态项][static items]
* [trait 定义][trait definitions]
* [实现][implementations]
* [`extern` 块][`extern` blocks]

r[items.locations]
程序项可以在[crate 根][root of the crate]、[模块][modules]或[块表达式][block expression]中声明。

r[items.associated-locations]
一部分程序项，称为[关联程序项][associated items]，可以在 [traits] 和[实现][implementations]中声明。

r[items.extern-locations]
一部分程序项，称为外部程序项，可以在 [`extern` 块][`extern` blocks]中声明。

r[items.decl-order]
程序项可以以任意顺序定义，但 [`macro_rules`] 有自己的作用域行为，属于例外。

r[items.name-resolution]
程序项名称的[名称解析][name resolution]允许在模块或块中，在引用该程序项的位置之前或之后定义该程序项。

有关程序项的作用域规则，请参见[程序项作用域][item scopes]。

[`extern crate` declarations]: items/extern-crates.md
[`extern` blocks]: items/external-blocks.md
[`macro_rules`]: macros-by-example.md
[`use` declarations]: items/use-declarations.md
[associated items]: items/associated-items.md
[block expression]: expressions/block-expr.md
[constant items]: items/constant-items.md
[enumeration definitions]: items/enumerations.md
[function definitions]: items/functions.md
[implementations]: items/implementations.md
[item scopes]: names/scopes.md#item-scopes
[modules]: items/modules.md
[name resolution]: names/name-resolution.md
[paths]: paths.md
[root of the crate]: crates-and-source-files.md
[statement]: statements.md
[static items]: items/static-items.md
[struct definitions]: items/structs.md
[trait definitions]: items/traits.md
[traits]: items/traits.md
[type alias definitions]: items/type-aliases.md
[union definitions]: items/unions.md
