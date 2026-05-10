r[expr.array]
# 数组和数组索引表达式

## 数组表达式

r[expr.array.syntax]
```grammar,expressions
ArrayExpression -> `[` ArrayElements? `]`

ArrayElements ->
      Expression ( `,` Expression )* `,`?
    | Expression `;` Expression
```

r[expr.array.constructor]
*数组表达式*构造[数组][array]。数组表达式有两种形式。

r[expr.array.array]
第一种形式列出数组中的每个值。

r[expr.array.array-syntax]
此形式的语法是用方括号括起来的、以逗号分隔的同一类型表达式列表。

r[expr.array.array-behavior]
这将生成一个包含这些值的数组，顺序与它们的书写顺序一致。

r[expr.array.repeat]
第二种形式的语法是用方括号括起来的、由分号（`;`）分隔的两个表达式。

r[expr.array.repeat-operand]
`;` 之前的表达式称为*重复操作数*。

r[expr.array.length-operand]
`;` 之后的表达式称为*长度操作数*。

r[expr.array.length-restriction]
长度操作数必须是[推断常量]或类型为 `usize` 的[常量表达式]（例如[字面量]或[常量项]）。

```rust
const C: usize = 1;
let _: [u8; C] = [0; 1]; // 字面量。
let _: [u8; C] = [0; C]; // 常量项。
let _: [u8; C] = [0; _]; // 推断常量。
let _: [u8; C] = [0; (((_)))]; // 推断常量。
```

> [!NOTE]
> 在数组表达式中，[推断常量]被解析为[表达式][Expression]，但在语义上被视为一种单独的[const 泛型参数][const generic argument]。

r[expr.array.repeat-behavior]
这种形式的数组表达式创建一个数组，长度等于长度操作数的值，每个元素都是重复操作数的副本。即 `[a; b]` 创建一个包含 `b` 个 `a` 值副本的数组。

r[expr.array.repeat-copy]
如果长度操作数的值大于 1，则要求重复操作数的类型实现 [`Copy`]，或是一个 [const 块表达式]，或是一个指向常量项的[路径]。

r[expr.array.repeat-const-item]
当重复操作数是 const 块或指向常量项的路径时，它会被求值长度操作数指定的次数。

r[expr.array.repeat-evaluation-zero]
如果该值为 `0`，则 const 块或常量项根本不会被求值。

r[expr.array.repeat-non-const]
对于既不是 const 块也不是指向常量项的路径的表达式，它会被正好求值一次，然后将结果复制长度操作数的值次。

```rust
[1, 2, 3, 4];
["a", "b", "c", "d"];
[0; 128];              // 包含 128 个零的数组
[0u8, 0u8, 0u8, 0u8,];
[[1, 0, 0], [0, 1, 0], [0, 0, 1]]; // 二维数组
const EMPTY: Vec<i32> = Vec::new();
[EMPTY; 2];
```

r[expr.array.index]
## 数组和切片索引表达式

r[expr.array.index.syntax]
```grammar,expressions
IndexExpression -> Expression `[` Expression `]`
```

r[expr.array.index.array]
[数组][Array]和[切片][slice]类型的值可以通过在其后跟一个用方括号括起来的类型为 `usize` 的表达式（索引）来索引。当数组可变时，所产生的[内存位置]可以被赋值。

r[expr.array.index.trait]
对于其他类型，索引表达式 `a[b]` 等价于 `*std::ops::Index::index(&a, b)`，或在可变位置表达式上下文中等价于 `*std::ops::IndexMut::index_mut(&mut a, b)`，不同之处在于当索引表达式经历[临时值生命周期延长]时，被索引的表达式 `a` 的[临时值作用域]也会被延长。与方法一样，Rust 也会在 `a` 上重复插入解引用操作以找到实现。

```rust
// 持有 `vec![()]` 结果的临时值被延长到块的末尾，
// 因此 `x` 可以在后续语句中使用。
let x = &vec![()][0];
# x;
```

```rust,compile_fail,E0716
// 持有 `vec![()]` 结果的临时值在语句末尾被丢弃，
// 因此之后使用 `y` 是错误的。
let y = &*std::ops::Index::index(&vec![()], 0); // 错误
# y;
```

r[expr.array.index.zero-index]
数组和切片的索引从零开始。

r[expr.array.index.const]
数组访问是[常量表达式]，因此可以在编译时使用常量索引值检查边界。否则将在运行时进行检查，如果失败将使线程进入[*panic 状态*][panic]。

```rust,should_panic
// lint 默认为 deny。
#![warn(unconditional_panic)]

([1, 2, 3, 4])[2];        // 求值为 3

let b = [[1, 0, 0], [0, 1, 0], [0, 0, 1]];
b[1][2];                  // 多维数组索引

let x = (["a", "b"])[10]; // 警告：索引越界

let n = 10;
let y = (["a", "b"])[n];  // panic

let arr = ["a", "b"];
arr[10];                  // 警告：索引越界
```

r[expr.array.index.trait-impl]
数组索引表达式可以通过实现 [Index] 和 [IndexMut] trait 为非数组和切片类型实现。

[`Copy`]: ../special-types-and-traits.md#copy
[IndexMut]: std::ops::IndexMut
[Index]: std::ops::Index
[array]: ../types/array.md
[const generic argument]: items.generics.const.argument
[const block expression]: expr.block.const
[constant expression]: ../const_eval.md#constant-expressions
[constant item]: ../items/constant-items.md
[inferred const]: items.generics.const.inferred
[literal]: ../tokens.md#literals
[memory location]: ../expressions.md#place-expressions-and-value-expressions
[panic]: ../panic.md
[path]: path-expr.md
[slice]: ../types/slice.md
[temporary lifetime extension]: destructors.scope.lifetime-extension
[temporary scope]: destructors.scope.temporary
