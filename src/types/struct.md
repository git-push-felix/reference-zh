r[type.struct]
# 结构体类型

r[type.struct.intro]
`struct` *类型*是其他类型的异构积，称为类型的*字段*。[^structtype]

r[type.struct.constructor]
`struct` 的新实例可以使用[结构体表达式][struct expression]构造。

r[type.struct.layout]
`struct` 的内存布局默认是未定义的，以便编译器进行字段重排等优化，但可以使用 [`repr` 属性][`repr` attribute]来固定。在两种情况下，字段都可以在相应的结构体*表达式*中以任意顺序给出；生成的 `struct` 值始终具有相同的内存布局。

r[type.struct.field-visibility]
`struct` 的字段可以受[可见性修饰符][visibility modifiers]限定，以允许在模块外部访问结构体中的数据。

r[type.struct.tuple]
*元组结构体*类型与结构体类型类似，只是字段是匿名的。

r[type.struct.unit]
*类单元结构体*类型类似于结构体类型，只是它没有字段。由关联的[结构体表达式][struct expression]构造的那个值是在这样的类型中存在的唯一值。

[^structtype]: `struct` 类型类似于 C 中的 `struct` 类型、ML 家族中的 *record* 类型或 Lisp 家族中的 *struct* 类型。

[`repr` attribute]: ../type-layout.md#representations
[struct expression]: ../expressions/struct-expr.md
[visibility modifiers]: ../visibility-and-privacy.md
