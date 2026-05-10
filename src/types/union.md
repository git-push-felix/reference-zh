r[type.union]
# 联合体类型

r[type.union.intro]
*联合体类型*是一种名义上的、异构的类 C 联合体，由 [`union` 项][item]的名称来表示。

r[type.union.access]
联合体没有"活动字段"的概念。相反，每次联合体访问会将联合体内容的部分按所访问字段的类型进行转换（transmute）。

r[type.union.safety]
由于转换可能导致意外或未定义行为，读取联合体字段需要 `unsafe`。

r[type.union.constraint]
联合体字段类型也限制为一组确保它们永远不需要被丢弃的类型。详见[该项][item]的文档。

r[type.union.layout]
`union` 的内存布局默认是未定义的（特别是字段*不*必须在偏移量 0 处），但是 `#[repr(...)]` 属性可以用于固定布局。

[`Copy`]: ../special-types-and-traits.md#copy
[item]: ../items/unions.md
