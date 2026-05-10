r[expr.tuple]
# 元组和元组索引表达式

## 元组表达式

r[expr.tuple.syntax]
```grammar,expressions
TupleExpression -> `(` TupleElements? `)`

TupleElements -> ( Expression `,` )+ Expression?
```

r[expr.tuple.result]
*元组表达式*构造[元组值][tuple type]。

r[expr.tuple.intro]
元组表达式的语法是用括号括起来的、以逗号分隔的表达式列表，称为*元组初始化操作数*。

r[expr.tuple.unary-tuple-restriction]
1 元元组表达式需要在其元组初始化操作数后面加一个逗号，以消除与[括号表达式]的歧义。

r[expr.tuple.value]
元组表达式是[值表达式]，求值为一个新构造的元组类型值。

r[expr.tuple.type]
元组初始化操作数的数量是所构造元组的元数。

r[expr.tuple.unit]
没有任何元组初始化操作数的元组表达式产生单元元组。

r[expr.tuple.fields]
对于其他元组表达式，第一个写入的元组初始化操作数初始化字段 `0`，后续操作数初始化下一个更高编号的字段。例如，在元组表达式 `('a', 'b', 'c')` 中，`'a'` 初始化字段 `0` 的值，`'b'` 初始化字段 `1`，`'c'` 初始化字段 `2`。

元组表达式及其类型示例：

| 表达式                | 类型             |
| --------------------- | ---------------- |
| `()`                  | `()`（单元）     |
| `(0.0, 4.5)`          | `(f64, f64)`     |
| `("x".to_string(), )` | `(String, )`     |
| `("a", 4usize, true)` | `(&'static str, usize, bool)` |

r[expr.tuple-index]
## 元组索引表达式

r[expr.tuple-index.syntax]
```grammar,expressions
TupleIndexingExpression -> Expression `.` TUPLE_INDEX
```

r[expr.tuple-index.intro]
*元组索引表达式*访问[元组][tuple type]和[元组结构体][tuple struct]的字段。

元组索引表达式的语法是一个表达式（称为*元组操作数*），后跟一个 `.`，最后是一个元组索引。

r[expr.tuple-index.index-syntax]
*元组索引*的语法是一个[十进制字面量]，没有前导零、下划线或后缀。例如 `0` 和 `2` 是有效的元组索引，但 `01`、`0_` 和 `0i32` 不是。

r[expr.tuple-index.required-type]
元组操作数的类型必须是[元组类型]或[元组结构体]。

r[expr.tuple-index.index-name-operand]
元组索引必须是元组操作数类型的一个字段名。

r[expr.tuple-index.result]
元组索引表达式的求值除了求值其元组操作数外没有副作用。作为[位置表达式]，它求值为元组操作数中与元组索引同名的字段的位置。

元组索引表达式示例：

```rust
// 索引元组
let pair = ("a string", 2);
assert_eq!(pair.1, 2);

// 索引元组结构体
# struct Point(f32, f32);
let point = Point(1.0, 0.0);
assert_eq!(point.0, 1.0);
assert_eq!(point.1, 0.0);
```

> [!NOTE]
> 与字段访问表达式不同，元组索引表达式可以是[调用表达式]的函数操作数，因为它不会与方法调用混淆，因为方法名不能是数字。

> [!NOTE]
> 尽管数组和切片也有元素，但必须使用[数组或切片索引表达式]或[切片模式]来访问它们的元素。

[array or slice indexing expression]: array-expr.md#array-and-slice-indexing-expressions
[call expression]: ./call-expr.md
[decimal literal]: ../tokens.md#integer-literals
[field access expressions]: ./field-expr.html#field-access-expressions
[operands]: ../expressions.md
[parenthetical expression]: grouped-expr.md
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[slice pattern]: ../patterns.md#slice-patterns
[tuple type]: ../types/tuple.md
[tuple struct]: ../types/struct.md
[value expression]: ../expressions.md#place-expressions-and-value-expressions
