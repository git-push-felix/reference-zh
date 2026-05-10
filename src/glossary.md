# 术语表

r[glossary.ast]
### 抽象语法树

"抽象语法树"（AST）是编译器在编译程序时对程序结构的一种中间表示。

### 对齐

值的对齐指定了值首选从哪些地址开始。始终是 2 的幂。对值的引用必须对齐。[更多信息][alignment]。

r[glossary.abi]
### 应用二进制接口（ABI）

*应用二进制接口*（ABI）定义了编译后的代码如何与其他编译后的代码交互。使用 [`extern` 块][`extern` blocks]和 [`extern fn`] 时，*ABI 字符串*影响：

- **调用约定**：函数参数如何传递、值如何返回（例如在寄存器中还是在栈上）以及谁负责清理栈。
- **展开**：是否允许栈展开。例如，`"C-unwind"` ABI 允许跨 FFI 边界展开，而 `"C"` ABI 不允许。

### 元数

元数指函数或运算符接受的参数数量。例如，`f(2, 3)` 和 `g(4, 6)` 的元数为 2，而 `h(8, 2, 6)` 的元数为 3。`!` 运算符的元数为 1。

### 数组

数组，有时也称为固定大小数组或内联数组，是一个描述元素集合的值，每个元素通过程序在运行时可以计算的索引来选择。它占用连续的内存区域。

### 关联项

关联项是与另一个项关联的项。关联项在[实现][implementations]中定义，在 [trait][traits] 中声明。只有函数、常量和类型别名可以关联。与[自由项][free item]对比。

### 毯式实现

任何在其中类型以[未覆盖](#未覆盖类型)形式出现的实现。`impl<T> Foo for T`、`impl<T> Bar<T> for T`、`impl<T> Bar<Vec<T>> for T` 和 `impl<T> Bar<T> for Vec<T>` 被视为毯式 impl。然而，`impl<T> Bar<Vec<T>> for Vec<T>` 不是毯式 impl，因为在此 `impl` 中出现的所有 `T` 实例都被 `Vec` 覆盖。

### 约束

约束是对类型或 trait 的限制。例如，如果对函数接受的参数设置了约束，则传递给该函数的类型必须遵守该约束。

### 组合子

组合子是高阶函数，仅应用函数和先前定义的组合子来提供其参数的结果。它们可用于以模块化方式管理控制流。

### Crate

Crate 是编译和链接的单位。存在不同[类型的 crate][types of crates]，例如库或可执行文件。Crate 可以链接和引用其他库 crate，称为外部 crate。一个 crate 有一个自包含的[模块][modules]树，从称为 crate 根的无名根模块开始。[程序项][Items]可以通过在 crate 根中标记为 public 使其对其他 crate 可见，包括通过 public 模块的[路径][paths]。[更多信息][crate]。

### 分发

分发是在涉及多态时确定实际运行哪个特定版本代码的机制。两种主要的分发形式是静态分发和动态分发。Rust 通过使用 [trait 对象][type.trait-object]支持动态分发。

### 动态大小类型

动态大小类型（DST）是一种没有静态已知大小或对齐的类型。

### 实体

[*实体*][*entity*]是一种语言构造，可以在源程序中以某种方式引用，通常通过[路径][paths]。实体包括[类型][types]、[程序项][items]、[泛型参数][generic parameters]、[变量绑定][variable bindings]、[循环标签][loop labels]、[生命周期][lifetimes]、[字段][fields]、[属性][attributes]和 [lint][lints]。

### 表达式

表达式是值、常量、变量、运算符和函数的组合，计算出单个值，可能带有副作用。

例如，`2 + (3 * 4)` 是一个返回值为 14 的表达式。

### 自由项

不是[实现][implementation]成员的[项][item]，例如*自由函数*或*自由常量*。与[关联项][associated item]对比。

### 基础 trait

基础 trait 是这样一个 trait：为现有类型添加一个 impl 是一个破坏性更改。`Fn` trait 和 `Sized` 是基础 trait。

### 基础类型构造器

基础类型构造器是这样一个类型：在其上实现[毯式实现](#毯式实现)是一个破坏性更改。`&`、`&mut`、`Box` 和 `Pin` 是基础类型构造器。

任何时候类型 `T` 被认为是[局部的](#局部类型)，`&T`、`&mut T`、`Box<T>` 和 `Pin<T>` 也被认为是局部的。基础类型构造器不能[覆盖](#未覆盖类型)其他类型。任何时候使用术语"已覆盖类型"时，`&T`、`&mut T`、`Box<T>` 和 `Pin<T>` 中的 `T` 不被认为是被覆盖的。

### 有人居住的

如果类型有构造函数并因此可以被实例化，则该类型是有人居住的。有人居住的类型不是"空"的，因为可以存在该类型的值。与[无人居住的](#无人居住的)相反。

### 固有实现

适用于名义类型而非 trait-类型对的[实现][implementation]。[更多信息][inherent implementation]。

### 固有方法

在[固有实现][inherent implementation]中定义的[方法][method]，而非在 trait 实现中。

### 已初始化

如果一个变量已被赋值且此后未被移动，则该变量是已初始化的。所有其他内存位置被假定为未初始化的。只有不安全 Rust 可以创建未初始化内存位置。

### 局部 trait

在当前 crate 中定义的 `trait`。一个 trait 定义是局部的或不取决于应用的类型参数。给定 `trait Foo<T, U>`，`Foo` 始终是局部的，无论替换 `T` 和 `U` 的类型是什么。

### 局部类型

在当前 crate 中定义的 `struct`、`enum` 或 `union`。这不受应用的类型参数影响。`struct Foo` 被认为是局部的，但 `Vec<Foo>` 不是。`LocalType<ForeignType>` 是局部的。类型别名不影响局部性。

### 模块

模块是包含零个或多个[程序项][items]的容器。模块组织成树，从根部的无名模块开始，称为 crate 根或根模块。[路径][Paths] 可用于引用其他模块的项，这可能受到[可见性规则][visibility rules]的限制。[更多信息][modules]

### 名称

[*名称*][*name*] 是引用[实体](#实体)的[标识符][identifier]或[生命周期或循环标签][lifetime or loop label]。*名称绑定*是当实体声明引入与该实体关联的标识符或标签时。路径、标识符和标签用于引用实体。

### 名称解析

[*名称解析*][*Name resolution*] 是将[路径][paths]、[标识符][identifiers]和[标签][labels]绑定到[实体](#实体)声明的编译时过程。

### 命名空间

*命名空间*是基于名称所指[实体](#实体)的类型对已声明[名称](#名称)的逻辑分组。命名空间允许一个命名空间中的名称出现不会与另一个命名空间中的相同名称冲突。

在命名空间内，名称组织成层次结构，层次结构的每个级别都有自己的一组命名实体。

### 名义类型

可以直接通过路径引用的类型。具体指[枚举][enums]、[结构体][structs]、[联合体][unions]和 [trait 对象类型][trait object types]。

### Dyn 兼容 trait

可以在 [trait 对象类型][trait object types]（`dyn Trait`）中使用的 [Trait][Traits]。只有遵循特定[规则][dyn compatibility]的 trait 才是*dyn 兼容的*。

这些以前称为*对象安全* trait。

### 路径

[*路径*][*path*] 是用于引用当前作用域或[命名空间](#命名空间)层次结构其他级别中的[实体](#实体)的一个或多个路径段的序列。

### 预导入

预导入，或称 Rust 预导入，是一小部分项——主要是 trait——被导入到每个 crate 的每个模块中。预导入中的 trait 是普遍存在的。

### 作用域

[*作用域*][*scope*] 是源文本中可以以其名称引用某命名[实体](#实体)的区域。

### 被匹配项

被匹配项是在 `match` 表达式和类似模式匹配构造中被匹配的表达式。例如，在 `match x { A => 1, B => 2 }` 中，表达式 `x` 是被匹配项。

### 大小

值的大小有两个定义。

第一个是存储该值必须分配多少内存。

第二个是该类型项的数组中连续元素之间的字节偏移量。

它是对齐的倍数，包括零。大小可能因编译器版本（随着新优化的实现）和目标平台而改变（类似于 `usize` 如何因平台而异）。

[更多信息][alignment]。

### 切片

切片是对连续序列的动态大小视图，写作 `[T]`。

它经常以其借用的形式出现，可以是可变或共享的。共享切片类型是 `&[T]`，可变切片类型是 `&mut [T]`，其中 `T` 表示元素类型。

### 语句

语句是编程语言中最小的独立元素，命令计算机执行某个操作。

### 字符串字面量

字符串字面量是直接存储在最终二进制文件中的字符串，因此将在 `'static` 持续时间内有效。

其类型是 `'static` 持续时间的借用字符串切片 `&'static str`。

### 字符串切片

字符串切片是 Rust 中最原始的字符串类型，写作 `str`。它经常以其借用的形式出现，可以是可变或共享的。共享字符串切片类型是 `&str`，可变字符串切片类型是 `&mut str`。

字符串切片始终是有效的 UTF-8。

### Trait

Trait 是一种语言项，用于描述类型必须提供的功能。它允许类型对其行为做出某些承诺。

泛型函数和泛型结构体可以使用 trait 来约束它们接受的类型。

### Turbofish

表达式中带有泛型参数的路径必须在开括号前加上 `::`。与用于泛型的尖括号结合，这看起来像一条鱼 `::<>`。因此，此语法俗称 turbofish 语法。

示例：

```rust
let ok_num = Ok::<_, ()>(5);
let vec = [1, 2, 3].iter().map(|n| n * 2).collect::<Vec<_>>();
```

此 `::` 前缀对于在有多个比较的逗号分隔列表中消除泛型路径的歧义是必需的。参见 [the bastion of the turbofish][turbofish test] 了解不加前缀会产生歧义的示例。

### 未覆盖类型

不作为其他类型参数出现的类型。例如，`T` 是未覆盖的，但 `Vec<T>` 中的 `T` 是已覆盖的。这仅与类型参数相关。

### 未定义行为

未指定的编译时或运行时行为。这可能导致但不限于：进程终止或损坏；不当、不正确或意外的计算；或特定于平台的结果。[更多信息][undefined-behavior]。

r[glossary.uninhabited]
### 无人居住的

如果类型没有构造函数并因此永远不能被实例化，则该类型是无人居住的。无人居住的类型是"空"的，因为没有该类型的值。无人居住类型的经典示例是 [never 类型][never type] `!`，或没有变体的枚举 `enum Never { }`。与[有人居住的](#有人居住的)相反。

r[glossary.zst]
### 零大小类型（ZST）

如果类型的大小为 0，则该类型是零大小的（ZST）。此类类型至多有一个可能的值。示例包括：

- [单元类型][unit type]（见 [layout.tuple.unit]）。
- [函数项][function items]（见 [type.fn-item.intro]）。
- [元组结构体][tuple-like structs]的构造函数（见 [type.fn-item.intro]）。
- [元组枚举变体][tuple-like enum variants]的构造函数（见 [type.fn-item.intro]）。
- 没有字段或所有字段为零大小的 `repr(C)` [结构体][structs]（见 [layout.repr.c.struct.size-field-offset]）。
- 没有字段或所有字段为零大小的 `repr(transparent)` [结构体][structs]（见 [layout.repr.transparent.layout-abi]）。
- 零大小类型的[数组][Arrays]（见 [layout.array]）。
- 长度为零的[数组][Arrays]（见 [layout.array]）。
- 零大小类型的[联合体][Unions]（见 [items.union.common-storage]）。

```rust
# use core::mem::{size_of, size_of_val};
fn f() {}
struct S(u8);
enum E { V(u8) }
#[repr(C)]
struct C1 {}
#[repr(C)]
struct C2 {
    f1: (),
    f2: [(); 10],
    f3: [u8; 0],
    f4: C1,
}
#[repr(transparent)]
struct T1 {}
#[repr(transparent)]
struct T2 {
    f1: (),
    f2: [(); 10],
    f3: [u8; 0],
}
union U {
    f1: (),
    f2: [(); 10],
    f3: [u8; 0],
}
assert_eq!(0, size_of::<()>());
assert_eq!(0, size_of_val(&f));
assert_eq!(0, size_of_val(&S));
assert_eq!(0, size_of_val(&E::V));
assert_eq!(0, size_of::<C1>());
assert_eq!(0, size_of::<C2>());
assert_eq!(0, size_of::<T1>());
assert_eq!(0, size_of::<T2>());
assert_eq!(0, size_of::<[(); 10]>());
assert_eq!(0, size_of::<[u8; 0]>());
assert_eq!(0, size_of::<U>());
```

[`extern` blocks]: items.extern
[`extern fn`]: items.fn.extern
[alignment]: type-layout.md#size-and-alignment
[arrays]: type.array
[associated item]: #关联项
[attributes]: attributes.md
[*entity*]: names.md
[crate]: crates-and-source-files.md
[dyn compatibility]: items/traits.md#dyn-compatibility
[enums]: items/enumerations.md
[fields]: expressions/field-expr.md
[free item]: #自由项
[function items]: type.fn-item
[generic parameters]: items/generics.md
[identifier]: identifiers.md
[identifiers]: identifiers.md
[implementation]: items/implementations.md
[implementations]: items/implementations.md
[inherent implementation]: items/implementations.md#inherent-implementations
[item]: items.md
[items]: items.md
[labels]: tokens.md#lifetimes-and-loop-labels
[lifetime or loop label]: tokens.md#lifetimes-and-loop-labels
[lifetimes]: tokens.md#lifetimes-and-loop-labels
[lints]: attributes/diagnostics.md#lint-check-attributes
[loop labels]: tokens.md#lifetimes-and-loop-labels
[method]: items/associated-items.md#methods
[modules]: items/modules.md
[*Name resolution*]: names/name-resolution.md
[*name*]: names.md
[*namespace*]: names/namespaces.md
[never type]: types/never.md
[*path*]: paths.md
[Paths]: paths.md
[*scope*]: names/scopes.md
[structs]: items/structs.md
[tuple-like enum variants]: items.enum.constructor-namespace
[tuple-like structs]: items.struct.tuple
[trait object types]: types/trait-object.md
[traits]: items/traits.md
[turbofish test]: https://github.com/rust-lang/rust/blob/1.58.0/src/test/ui/parser/bastion-of-the-turbofish.rs
[types of crates]: linkage.md
[types]: types.md
[undefined-behavior]: behavior-considered-undefined.md
[unions]: items/unions.md
[unit type]: type.tuple.unit
[variable bindings]: patterns.md
[visibility rules]: visibility-and-privacy.md
