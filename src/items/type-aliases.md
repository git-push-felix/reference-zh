r[items.type]
# 类型别名

r[items.type.syntax]
```grammar,items
TypeAlias ->
    `type` IDENTIFIER GenericParams? ( `:` Bounds )?
        WhereClause?
        ( `=` Type WhereClause?)? `;`
```

r[items.type.intro]
*类型别名*在其所在模块或块的[类型命名空间][type namespace]中为现有[类型][type]定义一个新名称。类型别名用关键字 `type` 声明。每个值都有单一、特定的类型，但可以实现多个不同的 trait，并且可以与多个不同的类型约束兼容。

例如，以下将类型 `Point` 定义为类型 `(u8, u8)` 的同义词，即无符号 8 位整数对的类型：

```rust
type Point = (u8, u8);
let p: Point = (41, 68);
```

r[items.type.constructor-alias]
元组结构体或单元结构体的类型别名不能用于限定该类型的构造器：

```rust,compile_fail
struct MyStruct(u32);

use MyStruct as UseAlias;
type TypeAlias = MyStruct;

let _ = UseAlias(5); // OK
let _ = TypeAlias(5); // 无效
```

r[items.type.associated-type]
类型别名在不用作[关联类型][associated type]时，必须包含一个[Type][grammar-Type]且不能包含 [Bounds]。

r[items.type.associated-trait]
类型别名在用作 [trait] 中的[关联类型][associated type]时，不能包含 [Type][grammar-Type] 的规格说明，但可以包含 [Bounds]。

r[items.type.associated-impl]
类型别名在用作 [trait impl][trait impl] 中的[关联类型][associated type]时，必须包含 [Type][grammar-Type] 的规格说明，且不能包含 [Bounds]。

r[items.type.deprecated]
在 [trait impl][trait impl] 中，类型别名的等号前的 where 子句（如 `type TypeAlias<T> where T: Foo = Bar<T>`）已被弃用。建议使用等号后的 where 子句（如 `type TypeAlias<T> = Bar<T> where T: Foo`）。

[associated type]: associated-items.md#associated-types
[trait impl]: implementations.md#trait-implementations
[trait]: traits.md
[type namespace]: ../names/namespaces.md
[type]: ../types.md
