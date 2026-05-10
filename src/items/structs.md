r[items.struct]
# 结构体

r[items.struct.syntax]
```grammar,items
Struct ->
      StructStruct
    | TupleStruct

StructStruct ->
    `struct` IDENTIFIER GenericParams? WhereClause? ( `{` StructFields? `}` | `;` )

TupleStruct ->
    `struct` IDENTIFIER GenericParams? `(` TupleFields? `)` WhereClause? `;`

StructFields -> StructField (`,` StructField)* `,`?

StructField -> OuterAttribute* Visibility? IDENTIFIER `:` Type

TupleFields -> TupleField (`,` TupleField)* `,`?

TupleField -> OuterAttribute* Visibility? Type
```

r[items.struct.intro]
*结构体*是用关键字 `struct` 定义的具名[结构体类型][struct type]。

r[items.struct.namespace]
结构体声明在其所在模块或块的[类型命名空间][type namespace]中定义给定的名称。

一个 `struct` 程序项及其使用的示例：

```rust
struct Point {x: i32, y: i32}
let p = Point {x: 10, y: 11};
let px: i32 = p.x;
```

r[items.struct.tuple]
*元组结构体*是具名的[元组类型][tuple type]，同样用关键字 `struct` 定义。除了定义类型外，它还在[值命名空间][value namespace]中定义一个同名的构造器。构造器是一个可以调用以创建结构体新实例的函数。例如：

```rust
struct Point(i32, i32);
let p = Point(10, 11);
let px: i32 = match p { Point(x, _) => x };
```

r[items.struct.unit]
*类单元结构体*是没有字段的结构体，通过完全省略字段列表来定义。这种结构体隐式定义了一个与其类型同名的[常量][constant]。例如：

```rust
struct Cookie;
let c = [Cookie, Cookie {}, Cookie, Cookie {}];
```

等价于

```rust
struct Cookie {}
const Cookie: Cookie = Cookie {};
let c = [Cookie, Cookie {}, Cookie, Cookie {}];
```

r[items.struct.layout]
结构体的精确内存布局未指定。可以使用 [`repr` 属性][`repr` attribute]来指定特定的布局。

[`repr` attribute]: ../type-layout.md#representations
[constant]: constant-items.md
[struct type]: ../types/struct.md
[tuple type]: ../types/tuple.md
[type namespace]: ../names/namespaces.md
[value namespace]: ../names/namespaces.md
