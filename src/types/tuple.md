r[type.tuple]
# 元组类型

r[type.tuple.syntax]
```grammar,types
TupleType ->
      `(` `)`
    | `(` ( Type `,` )+ Type? `)`
```

r[type.tuple.intro]
*元组类型*是其他类型的异构列表的一族结构类型[^1]。

元组类型的语法由括号括起的、逗号分隔的类型列表构成。

r[type.tuple.restriction]
1-元组需要在元素类型之后添加逗号，以与[括号类型]消除歧义。

r[type.tuple.field-number]
元组类型的字段数量等于类型列表的长度。这个字段数量确定了元组的*元数*。具有 `n` 个字段的元组称为 *n-元组*。例如，具有 2 个字段的元组是 2-元组。

r[type.tuple.field-name]
元组的字段使用提高的数字名称来命名，与其在类型列表中的位置对应。第一个字段是 `0`，第二个字段是 `1`，以此类推。每个字段的类型是元组类型列表中相同位置的类型。

r[type.tuple.unit]
为方便和历史原因，没有字段的元组类型（`()`）通常被称为*单元类型*。它的唯一值也称为*单元值*。

元组类型的一些示例：

* `()`（单元）
* `(i32,)`（1-元组）
* `(f64, f64)`
* `(String, i32)`
* `(i32, String)`（与上一个示例不同类型）
* `(i32, f64, Vec<String>, Option<bool>)`

r[type.tuple.constructor]
此类型的值使用[元组表达式]构造。此外，如果没有其他有意义的求值结果，各种表达式将产生单元值。

r[type.tuple.access]
元组字段可以通过[元组索引表达式]或[模式匹配]来访问。

[^1]: 如果内部类型等价，结构类型始终等价。关于元组的命名版本，请参见[元组结构体][tuple structs]。

[括号类型]: ../types.md#parenthesized-types
[模式匹配]: ../patterns.md#tuple-patterns
[元组表达式]: ../expressions/tuple-expr.md#tuple-expressions
[元组索引表达式]: ../expressions/tuple-expr.md#tuple-indexing-expressions
[tuple structs]: ./struct.md
