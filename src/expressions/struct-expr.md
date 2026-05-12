r[expr.struct]
# 结构体表达式

r[expr.struct.syntax]
```grammar,expressions
StructExpression ->
    PathInExpression `{` (StructExprFields | StructBase)? `}`

StructExprFields ->
    StructExprField (`,` StructExprField)* (`,` StructBase | `,`?)

StructExprField ->
    OuterAttribute*
    (
        IDENTIFIER
      | (IDENTIFIER | TUPLE_INDEX) `:` Expression
    )

StructBase -> `..` Expression
```

r[expr.struct.intro]
*结构体表达式*创建一个结构体、枚举或联合体值。它由一个指向[结构体][struct]、[枚举变体][enum variant]或[联合体][union]项的路径以及该项字段的值组成。

以下是结构体表达式的示例：

```rust
# struct Point { x: f64, y: f64 }
# struct NothingInMe { }
# mod game { pub struct User<'a> { pub name: &'a str, pub age: u32, pub score: usize } }
# enum Enum { Variant {} }
Point {x: 10.0, y: 20.0};
NothingInMe {};
let u = game::User {name: "Joe", age: 35, score: 100_000};
Enum::Variant {};
```

> [!NOTE]
> 元组结构体和元组枚举变体通常使用[调用表达式][expr.call]来实例化，该表达式引用[值命名空间中的构造器][items.struct.tuple]。这与使用花括号引用类型命名空间中构造器的结构体表达式是不同的。
>
> ```rust
> struct Position(i32, i32, i32);
> Position(0, 0, 0);  // 创建元组结构体的典型方式。
> let c = Position;  // `c` 是一个接受 3 个参数的函数。
> let pos = c(8, 6, 7);  // 创建一个 `Position` 值。
>
> enum Version { Triple(i32, i32, i32) };
> Version::Triple(0, 0, 0);
> let f = Version::Triple;
> let ver = f(8, 6, 7);
> ```
>
> 调用路径的最后一段不能引用类型别名：
>
> ```rust
> trait Tr { type T; }
> impl<T> Tr for T { type T = T; }
>
> struct Tuple();
> enum Enum { Tuple() }
>
> // <Unit as Tr>::T(); // 导致错误 -- `::T` 是类型，不是值
> <Enum as Tr>::T::Tuple(); // OK
> ```
>
> ----
>
> 单元结构体和单元枚举变体通常使用[路径表达式][expr.path]来实例化，该表达式引用[值命名空间中的常量][items.struct.unit]。
>
> ```rust
> struct Gamma;
> // Gamma 单元值，引用值命名空间中的常量。
> let a = Gamma;
> // 与 `a` 完全相同的值，但使用引用类型命名空间的
> // 结构体表达式构造。
> let b = Gamma {};
>
> enum ColorSpace { Oklch }
> let c = ColorSpace::Oklch;
> let d = ColorSpace::Oklch {};
> ```

r[expr.struct.field]
## 字段结构体表达式

r[expr.struct.field.intro]
用花括号括起字段的结构体表达式允许你以任意顺序为每个单独的字段指定值。字段名与其值之间用冒号分隔。

r[expr.struct.field.union-constraint]
[联合体][union]类型的值只能使用此语法创建，并且必须恰好指定一个字段。

r[expr.struct.update]
## 函数式更新语法 {#functional-update-syntax}

r[expr.struct.update.intro]
构造结构体类型值的结构体表达式可以用 `..` 后跟一个表达式作为结尾，以表示函数式更新。

r[expr.struct.update.base-same-type]
`..` 后面的表达式（基值）必须具有与正在构造的新结构体类型相同的结构体类型。

r[expr.struct.update.fields]
整个表达式使用为已指定字段给定的值，并移动或复制基值表达式中其余字段的值。

r[expr.struct.update.visibility-constraint]
与所有结构体表达式一样，结构体的所有字段必须是[可见][visible]的，即使那些没有被显式命名的字段也是如此。

```rust
# struct Point3d { x: i32, y: i32, z: i32 }
let mut base = Point3d {x: 1, y: 2, z: 3};
let y_ref = &mut base.y;
Point3d {y: 0, z: 10, .. base}; // OK，仅访问了 base.x
drop(y_ref);
```

r[expr.struct.brace-restricted-positions]
结构体表达式不能直接用于[循环][loop]或 [`if`][if] 表达式的头部，也不能用于 [if let] 或 [match] 表达式的[受检者][scrutinee]。但是，如果结构体表达式位于另一个表达式内部（例如在[括号][parentheses]内），则可以在这些情况下使用。

r[expr.struct.tuple-field]
字段名可以是十进制整数值以指定用于构造元组结构体的索引。这可以与基值结构体一起使用，以填充未指定的其余索引：

```rust
struct Color(u8, u8, u8);
let c1 = Color(0, 0, 0);  // 创建元组结构体的典型方式。
let c2 = Color{0: 255, 1: 127, 2: 0};  // 按索引指定字段。
let c3 = Color{1: 0, ..c2};  // 使用基值结构体填充所有其他字段。
```

r[expr.struct.field.named]
### 结构体字段初始化简写

在初始化具有命名（而非编号）字段的数据结构（结构体、枚举、联合体）时，允许将 `fieldname` 作为 `fieldname: fieldname` 的简写。这允许使用更紧凑的语法并减少重复。例如：

```rust
# struct Point3d { x: i32, y: i32, z: i32 }
# let x = 0;
# let y_value = 0;
# let z = 0;
Point3d { x: x, y: y_value, z: z };
Point3d { x, y: y_value, z };
```

[enum variant]: ../items/enumerations.md
[if let]: if-expr.md#if-let-patterns
[if]: if-expr.md#if-expressions
[loop]: loop-expr.md
[match]: match-expr.md
[parentheses]: grouped-expr.md
[struct]: ../items/structs.md
[union]: ../items/unions.md
[visible]: ../visibility-and-privacy.md
[scrutinee]: ../glossary.md#scrutinee
