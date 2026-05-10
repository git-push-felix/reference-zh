r[type.enum]
# 枚举类型

r[type.enum.intro]
*枚举类型*是一种名义上的、异构的不交和类型，由 [`enum` 项]的名称表示。[^enumtype]

r[type.enum.declaration]
[`enum` 项]声明了该类型和若干*变体*，每个变体各自有独立的名称，并使用结构体、元组结构体或类单元结构体的语法。

r[type.enum.constructor]
`enum` 的新实例可以使用[结构体表达式]构造。

r[type.enum.value]
任何 `enum` 值消耗的内存与其对应 `enum` 类型中最大的变体一样多，外加存储判别值所需的大小。

r[type.enum.name]
枚举类型不能以*结构方式*作为类型来表示，而必须通过对 [`enum` 项]的命名引用来表示。

[^enumtype]: `enum` 类型类似于 Haskell 中的 `data` 构造声明，或 Limbo 中的 *pick ADT*。

[`enum` 项]: ../items/enumerations.md
[结构体表达式]: ../expressions/struct-expr.md
