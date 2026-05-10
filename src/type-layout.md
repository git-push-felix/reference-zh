r[layout]
# 类型布局

r[layout.intro]
类型的布局是指其大小、对齐方式以及其字段的相对偏移量。对于枚举，判别式的布局和解读方式也是类型布局的一部分。

r[layout.guarantees]
类型布局可能随每次编译而改变。与其尝试详尽记录实现细节，我们只记录当前保证的部分。

注意，即使是具有相同布局的类型，其在跨函数边界的传递方式仍可能不同。关于类型的函数调用 ABI 兼容性，请参见[此处][fn-abi-compatibility]。

r[layout.properties]
## 大小与对齐

所有值都具有对齐方式和大小。

r[layout.properties.align]
值的*对齐方式*指定了哪些地址可以存储该值。对齐方式为 `n` 的值只能存储在地址为 n 的倍数的位置。例如，对齐方式为 2 的值必须存储在偶数地址，而对齐方式为 1 的值可以存储在任何地址。对齐方式以字节为单位，必须至少为 1，并且始终是 2 的幂。可以使用 [`align_of_val`] 函数检查值的对齐方式。

r[layout.properties.size]
值的*大小*是包含该项类型的数组中连续元素之间的偏移量（以字节为单位，包括对齐填充）。值的大小始终是其对齐方式的倍数。注意，有些类型是[零大小][zero-sized]的；0 被认为是任何对齐方式的倍数（例如，在某些平台上，类型 `[u16; 0]` 的大小为 0，对齐方式为 2）。可以使用 [`size_of_val`] 函数检查值的大小。

r[layout.properties.sized]
所有值均具有相同大小和对齐方式，并且两者在编译时均已知的类型，实现了 [`Sized`] trait，并可以通过 [`size_of`] 和 [`align_of`] 函数进行检查。非 [`Sized`] 的类型称为[动态大小类型][dynamically sized types]。由于 `Sized` 类型的所有值共享相同的大小和对齐方式，我们分别将这些共享值称为该类型的大小和对齐方式。

r[layout.primitive]
## 原始数据类型布局

r[layout.primitive.size]
大多数原始类型的大小如下表所示。

| 类型              | `size_of::<Type>()`|
|--                 |--                  |
| `bool`            | 1                  |
| `u8` / `i8`       | 1                  |
| `u16` / `i16`     | 2                  |
| `u32` / `i32`     | 4                  |
| `u64` / `i64`     | 8                  |
| `u128` / `i128`   | 16                 |
| `usize` / `isize` | 见下文             |
| `f32`             | 4                  |
| `f64`             | 8                  |
| `char`            | 4                  |

r[layout.primitive.size-minimum]
`usize` 和 `isize` 的大小足以容纳目标平台上的每个地址。例如，在 32 位目标上为 4 字节，在 64 位目标上为 8 字节。

r[layout.primitive.size-align]
`usize` 和 `isize` 具有相同的大小和对齐方式。

r[layout.primitive.platform-specific-alignment]
原始类型的对齐方式是平台相关的。在大多数情况下，它们的对齐方式等于其大小，但也可能更小。特别地，`i128` 和 `u128` 通常对齐到 4 或 8 字节，尽管它们的大小为 16；在许多 32 位平台上，`i64`、`u64` 和 `f64` 仅对齐到 4 字节，而不是 8。

r[layout.primitive.integer-alignment]
保证相同指示大小的定宽有符号和无符号整数变体的对齐方式相同——即，对于给定大小 `N`，`align_of::<uN>() == align_of::<iN>()`。

r[layout.pointer]
## 指针与引用布局

r[layout.pointer.intro]
指针和引用具有相同的布局。指针或引用的可变性不会改变布局。

r[layout.pointer.thin]
指向固定大小类型的指针具有与 `usize` 相同的大小和对齐方式。

r[layout.pointer.unsized]
指向非固定大小类型的指针是固定大小的。指向非固定大小类型的指针的大小和对齐方式保证大于或等于指向固定大小类型的指针的大小和对齐方式。

> [!NOTE]
> 尽管你不应依赖于此，但目前所有指向 <abbr title="动态大小类型">DST</abbr> 的指针都是 `usize` 大小的两倍，并具有相同的对齐方式。

r[layout.array]
## 数组布局

`[T; N]` 数组的大小为 `size_of::<T>() * N`，与 `T` 具有相同的对齐方式。数组的布局使得其从零开始的第 `n` 个元素从数组开头偏移 `n * size_of::<T>()` 字节。

r[layout.slice]
## 切片布局

切片与其所切片的数组段具有相同的布局。

> [!NOTE]
> 这里说的是原生 `[T]` 类型，而非指向切片的指针（`&[T]`、`Box<[T]>` 等）。

r[layout.str]
## `str` 布局

字符串切片是字符的 UTF-8 表示形式，与类型 `[u8]` 的切片具有相同的布局。引用 `&str` 与引用 `&[u8]` 具有相同的布局。

r[layout.tuple]
## 元组布局

r[layout.tuple.def]
元组按照 [`Rust` 表示法]进行布局。

r[layout.tuple.unit]
例外是单元元组（`()`），它被保证作为[零大小类型][zero-sized type]，大小为 0，对齐方式为 1。

r[layout.trait-object]
## Trait 对象布局

Trait 对象与 trait 对象所指的值具有相同的布局。

> [!NOTE]
> 这里说的是原生 trait 对象类型，而非指向 trait 对象的指针（`&dyn Trait`、`Box<dyn Trait>` 等）。

r[layout.closure]
## 闭包布局

闭包没有布局保证。

r[layout.repr]
## 表示法

r[layout.repr.intro]
所有用户定义的复合类型（`struct`、`enum` 和 `union`）都有一个*表示法*，该表示法指定了该类型的布局。

r[layout.repr.kinds]
类型的可能表示法有：

- [`Rust`]（默认）
- [`C`]
- [原始表示法][primitive representations]
- [`transparent`]

r[layout.repr.attribute]
可以通过将 `repr` 属性应用于类型来更改其表示法。以下示例展示了一个带有 `C` 表示法的结构体。

```rust
#[repr(C)]
struct ThreeInts {
    first: i16,
    second: i8,
    third: i32
}
```

r[layout.repr.align-packed]
可以通过 `align` 和 `packed` 修饰符分别提高或降低对齐方式。它们会修改属性中指定的表示法。如果未指定表示法，则修改默认表示法。

```rust
// 默认表示法，对齐方式降低到 2。
#[repr(packed(2))]
struct PackedStruct {
    first: i16,
    second: i8,
    third: i32
}

// C 表示法，对齐方式提高到 8
#[repr(C, align(8))]
struct AlignedStruct {
    first: i16,
    second: i8,
    third: i32
}
```

> [!NOTE]
> 作为项上的属性，表示法不依赖于泛型参数。任何两个同名的类型具有相同的表示法。例如，`Foo<Bar>` 和 `Foo<Baz>` 都具有相同的表示法。

r[layout.repr.inter-field]
类型的表示法可以更改字段之间的填充，但不会更改字段本身的布局。例如，一个具有 `C` 表示法的结构体包含了一个具有 `Rust` 表示法的结构体 `Inner`，这不会改变 `Inner` 的布局。

<a id="the-default-representation"></a>
r[layout.repr.rust]
### `Rust` 表示法

r[layout.repr.rust.intro]
`Rust` 表示法是未标注 `repr` 属性的名义类型的默认表示法。通过 `repr` 属性显式使用此表示法，保证与完全省略该属性时相同。

r[layout.repr.rust.layout]
此表示法对数据布局的唯一保证是那些为语言健全性所需的保证。它们是：

 1. 字段的偏移量可以被该字段的对齐方式整除。
 2. 类型的对齐方式至少是其字段的最大对齐方式。

r[layout.repr.rust.layout.struct]
对于 [structs]，进一步保证字段不会重叠。也就是说，字段可以排序，使得任何字段的偏移量加上其大小小于等于排序中下一个字段的偏移量。该排序不必须与类型声明中字段的指定顺序相同。

请注意，此保证并不意味着字段具有不同的地址：[零大小类型][zero-sized types]可能与同一结构体中的其他字段具有相同的地址。

r[layout.repr.rust.unspecified]
此表示法对数据布局没有其他保证。

r[layout.repr.c]
### `C` 表示法

r[layout.repr.c.intro]
`C` 表示法设计用于双重目的。一个目的是创建与 C 语言可互操作的类型。第二个目的是创建可以安全地执行依赖于数据布局的操作的类型，例如将值重新解释为不同的类型。

由于这种双重目的，可以创建对 C 编程语言接口无用的类型。

r[layout.repr.c.constraint]
此表示法可以应用于结构体、联合体和枚举。[零变体枚举][zero-variant enums]为例外，对其使用 `C` 表示法会报错。

r[layout.repr.c.struct]
#### `#[repr(C)]` 结构体

r[layout.repr.c.struct.align]
结构体的对齐方式是其内部对齐方式最大的字段的对齐方式。

r[layout.repr.c.struct.size-field-offset]
字段的大小和偏移量由以下算法确定。

从当前偏移量 0 字节开始。

对于结构体中按声明顺序的每个字段，首先确定该字段的大小和对齐方式。如果当前偏移量不是字段对齐方式的倍数，则向当前偏移量添加填充字节，直到其为字段对齐方式的倍数。字段的偏移量就是当前的偏移量数值。然后将当前偏移量增加该字段的大小。

最后，结构体的大小为当前偏移量向上取整到结构体对齐方式的最近倍数。

以下是此算法的伪代码描述。

<!-- ignore: pseudocode -->
```rust,ignore
/// Returns the amount of padding needed after `offset` to ensure that the
/// following address will be aligned to `alignment`.
fn padding_needed_for(offset: usize, alignment: usize) -> usize {
    let misalignment = offset % alignment;
    if misalignment > 0 {
        // round up to next multiple of `alignment`
        alignment - misalignment
    } else {
        // already a multiple of `alignment`
        0
    }
}

struct.alignment = struct.fields().map(|field| field.alignment).max();

let current_offset = 0;

for field in struct.fields_in_declaration_order() {
    // Increase the current offset so that it's a multiple of the alignment
    // of this field. For the first field, this will always be zero.
    // The skipped bytes are called padding bytes.
    current_offset += padding_needed_for(current_offset, field.alignment);

    struct[field].offset = current_offset;

    current_offset += field.size;
}

struct.size = current_offset + padding_needed_for(current_offset, struct.alignment);
```

> [!WARNING]
> 此伪代码为清晰起见使用了忽略溢出问题的朴素算法。要在实际代码中执行内存布局计算，请使用 [`Layout`]。

> [!NOTE]
> 此算法可以产生[零大小][zero-sized]结构体。在 C 中，像 `struct Foo { }` 这样的空结构体声明是非法的。然而，gcc 和 clang 都支持启用此类结构体的选项，并为其分配大小零。相比之下，C++ 为空结构体赋予大小 1，除非它们来自继承或带有 `[[no_unique_address]]` 属性的字段，在这种情况下它们不会增加结构体的总大小。

r[layout.repr.c.union]
#### `#[repr(C)]` 联合体

r[layout.repr.c.union.intro]
用 `#[repr(C)]` 声明的联合体将具有与目标平台的 C 语言中等价 C 联合体声明相同的大小和对齐方式。

r[layout.repr.c.union.size-align]
联合体的大小为其所有字段的最大大小向上取整到其对齐方式，对齐方式为其所有字段的最大对齐方式。这些最大值可能来自不同的字段。每个字段都位于从联合体开头偏移 0 字节的位置。

```rust
#[repr(C)]
union Union {
    f1: u16,
    f2: [u8; 4],
}

assert_eq!(std::mem::size_of::<Union>(), 4);  // 来自 f2
assert_eq!(std::mem::align_of::<Union>(), 2); // 来自 f1

assert_eq!(std::mem::offset_of!(Union, f1), 0);
assert_eq!(std::mem::offset_of!(Union, f2), 0);

#[repr(C)]
union SizeRoundedUp {
   a: u32,
   b: [u16; 3],
}

assert_eq!(std::mem::size_of::<SizeRoundedUp>(), 8);  // 大小为 6，b 提供，
                                                       // 向上取整为 8，来自 a 的对齐方式。
assert_eq!(std::mem::align_of::<SizeRoundedUp>(), 4); // 来自 a

assert_eq!(std::mem::offset_of!(SizeRoundedUp, a), 0);
assert_eq!(std::mem::offset_of!(SizeRoundedUp, b), 0);
```

r[layout.repr.c.enum]
#### `#[repr(C)]` 无字段枚举

对于[无字段枚举][field-less enums]，`C` 表示法具有目标平台的 C ABI 下默认 `enum` 大小和对齐方式的大小和对齐方式。

> [!NOTE]
> C 中的枚举表示法是实现定义的，因此这实际上是一个"最佳猜测"。特别是，当感兴趣的 C 代码使用某些标志编译时，这可能是不正确的。

> [!WARNING]
> C 语言中的 `enum` 与具有此表示法的 Rust [无字段枚举][field-less enums]之间存在关键差异。C 中的 `enum` 主要是一个 `typedef` 加上一些命名常量；换句话说，`enum` 类型的对象可以容纳任何整数值。例如，这常用于 C 中的位标志。相比之下，Rust 的[无字段枚举][field-less enums]只能合法地容纳判别值，其他任何值都是[未定义行为][undefined behavior]。因此，在 FFI 中使用无字段枚举来建模 C `enum` 通常是错误的。

r[layout.repr.c.adt]
#### `#[repr(C)]` 带字段枚举

r[layout.repr.c.adt.intro]
带有字段的 `repr(C)` 枚举的表示法是一个包含两个字段的 `repr(C)` 结构体，在 C 中也被称为"带标签联合体"：

r[layout.repr.c.adt.tag]
- 枚举的 `repr(C)` 版本，其所有字段被移除（"标签"）

r[layout.repr.c.adt.fields]
- 一个 `repr(C)` 联合体，包含每个具有字段的变体的字段所对应的 `repr(C)` 结构体（"负载"）

> [!NOTE]
> 由于 `repr(C)` 结构体和联合体的表示法，如果一个变体只有一个字段，将该字段直接放在联合体中或将其包装在结构体中是没有区别的；任何希望操作此类 `enum` 表示法的系统因此可以使用更方便或更一致的形式。

```rust
// 此枚举与...具有相同的表示法
#[repr(C)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ...这个结构体。
#[repr(C)]
struct MyEnumRepr {
    tag: MyEnumDiscriminant,
    payload: MyEnumFields,
}

// 这是判别式枚举。
#[repr(C)]
enum MyEnumDiscriminant { A, B, C, D }

// 这是变体联合体。
#[repr(C)]
union MyEnumFields {
    A: MyAFields,
    B: MyBFields,
    C: MyCFields,
    D: MyDFields,
}

#[repr(C)]
#[derive(Copy, Clone)]
struct MyAFields(u32);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyBFields(f32, u64);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyCFields { x: u32, y: u8 }

// 此结构体可以省略（它是一个零大小类型），且在 C/C++ 头文件中必须如此。
#[repr(C)]
#[derive(Copy, Clone)]
struct MyDFields;
```

r[layout.repr.primitive]
### 原始表示法

r[layout.repr.primitive.intro]
*原始表示法*是与原始整数类型同名的表示法。即：`u8`、`u16`、`u32`、`u64`、`u128`、`usize`、`i8`、`i16`、`i32`、`i64`、`i128` 和 `isize`。

r[layout.repr.primitive.constraint]
原始表示法只能应用于枚举，并且根据枚举是否有字段而具有不同的行为。对[零变体枚举][zero-variant enums]使用原始表示法是错误的。将两种原始表示法组合在一起是错误的。

r[layout.repr.primitive.enum]
#### 无字段枚举的原始表示法

对于[无字段枚举][field-less enums]，原始表示法设置大小和对齐方式与同名的原始类型相同。例如，带有 `u8` 表示法的无字段枚举只能具有 0 到 255（含）之间的判别值。

r[layout.repr.primitive.adt]
#### 带字段枚举的原始表示法

原始表示法枚举的表示法是一个 `repr(C)` 联合体，包含每个具有字段变体的 `repr(C)` 结构体。联合体中每个结构体的第一个字段是带有原始表示法的枚举的（移除所有字段后的）版本（"标签"），其余字段是该变体的字段。

> [!NOTE]
> 如果标签在联合体中拥有自己的成员，此表示法不变，这或许能让你操作更清晰（尽管遵循 C++ 标准，标签成员应包装在一个 `struct` 中）。

```rust
// 此枚举与...具有相同的表示法
#[repr(u8)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ...这个联合体。
#[repr(C)]
union MyEnumRepr {
    A: MyVariantA,
    B: MyVariantB,
    C: MyVariantC,
    D: MyVariantD,
}

// 这是判别式枚举。
#[repr(u8)]
#[derive(Copy, Clone)]
enum MyEnumDiscriminant { A, B, C, D }

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantA(MyEnumDiscriminant, u32);

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantB(MyEnumDiscriminant, f32, u64);

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantC { tag: MyEnumDiscriminant, x: u32, y: u8 }

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantD(MyEnumDiscriminant);
```

r[layout.repr.primitive-c]
#### 带有字段枚举的原始表示法与 `#[repr(C)]` 的组合

对于带有字段的枚举，还可以组合 `repr(C)` 和原始表示法（例如 `repr(C, u8)`）。这会修改 [`repr(C)`] 的表示法，将判别式枚举的表示法更改为所选的原始表示法。因此，如果你选择 `u8` 表示法，则判别式枚举的大小和对齐方式将为 1 字节。

上面[更早][`repr(C)`]示例中的判别式枚举将变为：

```rust
#[repr(C, u8)] // 添加了 `u8`
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ...

#[repr(u8)] // 所以这里使用 `u8` 而不是 `C`
enum MyEnumDiscriminant { A, B, C, D }

// ...
```

例如，使用 `repr(C, u8)` 枚举不能有 257 个唯一的判别值（"标签"），而仅带有 `repr(C)` 属性的相同枚举可以没有任何问题地编译。

在 `repr(C)` 之外还使用原始表示法可以改变枚举相对于 `repr(C)` 形式的大小：

```rust
#[repr(C)]
enum EnumC {
    Variant0(u8),
    Variant1,
}

#[repr(C, u8)]
enum Enum8 {
    Variant0(u8),
    Variant1,
}

#[repr(C, u16)]
enum Enum16 {
    Variant0(u8),
    Variant1,
}

// C 表示法的大小是平台相关的
assert_eq!(std::mem::size_of::<EnumC>(), 8);
// 一个字节用于判别式，一个字节用于 Enum8::Variant0 中的值
assert_eq!(std::mem::size_of::<Enum8>(), 2);
// 两个字节用于判别式，一个字节用于 Enum16::Variant0 中的值
// 加上一个字节的填充。
assert_eq!(std::mem::size_of::<Enum16>(), 4);
```

[`repr(C)`]: #reprc-enums-with-fields

r[layout.repr.alignment]
### 对齐修饰符

r[layout.repr.alignment.intro]
`align` 和 `packed` 修饰符可以分别用于提高或降低 `struct` 和 `union` 的对齐方式。`packed` 还可能改变字段之间的填充（尽管不会改变任何字段内部的填充）。单独使用时，`align` 和 `packed` 不提供关于结构体布局中字段顺序或枚举变体布局的保证，尽管它们可以与提供此类保证的表示法（如 `C`）组合使用。

r[layout.repr.alignment.constraint-alignment]
对齐方式以整数参数的形式指定，格式为 `#[repr(align(x))]` 或 `#[repr(packed(x))]`。对齐值必须是 2 的幂，范围为 1 到 2<sup>29</sup>。对于 `packed`，如果未给出值，如 `#[repr(packed)]`，则值为 1。

r[layout.repr.alignment.align]
对于 `align`，如果指定的对齐方式小于没有 `align` 修饰符时类型的对齐方式，则对齐方式不受影响。

r[layout.repr.alignment.packed]
对于 `packed`，如果指定的对齐方式大于没有 `packed` 修饰符时类型的对齐方式，则对齐方式和布局不受影响。

r[layout.repr.alignment.packed-fields]
为了定位字段，每个字段的对齐方式取指定对齐方式和该字段类型对齐方式中的较小值。

r[layout.repr.alignment.packed-padding]
保证字段间填充是满足每个字段（可能已更改）的对齐方式所需的最小值（尽管请注意，单独使用的 `packed` 不提供任何关于字段顺序的保证）。这些规则的一个重要结果是，带有 `#[repr(packed(1))]`（或 `#[repr(packed)]`）的类型将没有字段间填充。

r[layout.repr.alignment.constraint-exclusive]
`align` 和 `packed` 修饰符不能应用于同一类型，且 `packed` 类型不能传递性地包含另一个 `align` 类型。`align` 和 `packed` 只能应用于 [`Rust`] 和 [`C`] 表示法。

r[layout.repr.alignment.enum]
`align` 修饰符也可以应用于 `enum`。当应用时，对 `enum` 对齐方式的影响与将 `enum` 包装在带有相同 `align` 修饰符的 newtype `struct` 中相同。

> [!NOTE]
> 不允许引用未对齐的字段，因为这是[未定义行为][undefined behavior]。当字段由于对齐修饰符而变得未对齐时，请考虑以下使用引用和解引用的选项：
>
> ```rust
> #[repr(packed)]
> struct Packed {
>     f1: u8,
>     f2: u16,
> }
> let mut e = Packed { f1: 1, f2: 2 };
> // 不用创建对字段的引用，而是将值复制到局部变量中。
> let x = e.f2;
> // 或者在像 `println!` 这样创建引用的场景中，使用花括号
> // 将其更改为值的副本。
> println!("{}", {e.f2});
> // 或者如果你需要一个指针，使用未对齐的方法进行读取和写入，
> // 而不是直接解引用指针。
> let ptr: *const u16 = &raw const e.f2;
> let value = unsafe { ptr.read_unaligned() };
> let mut_ptr: *mut u16 = &raw mut e.f2;
> unsafe { mut_ptr.write_unaligned(3) }
> ```

r[layout.repr.transparent]
### `transparent` 表示法

r[layout.repr.transparent.constraint-field]
`transparent` 表示法只能用于具有单个变体的 [`struct`][structs] 或 [`enum`][enumerations]，该变体具有：
- 任意数量的、大小为 0 且对齐方式为 1 的字段（例如 [`PhantomData<T>`]），以及
- 至多一个其他字段。

r[layout.repr.transparent.layout-abi]
具有此表示法的结构体和枚举具有与唯一的非 0 大小、非 1 对齐字段相同的布局和 ABI（如果存在），否则具有与单元类型相同的布局和 ABI。

这与 `C` 表示法不同，因为具有 `C` 表示法的结构体始终具有 C `struct` 的 ABI，而例如，具有 `transparent` 表示法且带有原始字段的结构体将具有该原始字段的 ABI。

r[layout.repr.transparent.constraint-exclusive]
因为此表示法将类型布局委托给另一个类型，所以它不能与任何其他表示法一起使用。

[`align_of_val`]: std::mem::align_of_val
[`size_of_val`]: std::mem::size_of_val
[`align_of`]: std::mem::align_of
[`size_of`]: std::mem::size_of
[`Sized`]: std::marker::Sized
[`Copy`]: std::marker::Copy
[dynamically sized types]: dynamically-sized-types.md
[field-less enums]: items/enumerations.md#field-less-enum
[fn-abi-compatibility]: ../core/primitive.fn.md#abi-compatibility
[enumerations]: items/enumerations.md
[zero-variant enums]: items/enumerations.md#zero-variant-enums
[undefined behavior]: behavior-considered-undefined.md
[zero-sized]: glossary.zst
[zero-sized type]: glossary.zst
[zero-sized types]: glossary.zst
[`PhantomData<T>`]: special-types-and-traits.md#phantomdatat
[`Rust`]: #the-rust-representation
[`C`]: #the-c-representation
[primitive representations]: #primitive-representations
[structs]: items/structs.md
[`transparent`]: #the-transparent-representation
[`Layout`]: std::alloc::Layout
