r[dynamic-sized]
# 动态大小类型

r[dynamic-sized.intro]
大多数类型具有在编译时已知的固定大小，并实现了 [`Sized`][sized] trait。大小仅在运行时才知道的类型称为*动态大小类型*（*DST*），或非正式地称为无大小类型。[切片][Slices]、[trait 对象][trait objects]和 [str] 是 <abbr title="dynamically sized types">DST</abbr> 的示例。

r[dynamic-sized.restriction]
此类类型只能在某些情况下使用：

r[dynamic-sized.pointer-types]
* 指向 <abbr title="dynamically sized types">DST</abbr> 的[指针类型][Pointer types]具有固定大小，但大小是指向有大小类型的指针的两倍
    * 指向切片和 `str` 的指针还存储元素数量。
    * 指向 trait 对象的指针还存储一个指向虚表的指针。

r[dynamic-sized.question-sized]
* <abbr title="dynamically sized types">DST</abbr> 可以作为类型实参提供给具有特殊 `?Sized` 约束的泛型类型参数。当对应的关联类型声明具有 `?Sized` 约束时，它们也可以用于关联类型定义。默认情况下，任何类型参数或关联类型具有 `Sized` 约束，除非使用 `?Sized` 放宽。

r[dynamic-sized.trait-impl]
* 可以为 <abbr title="dynamically sized
  types">DST</abbr> 实现 trait。与泛型类型参数不同，`Self: ?Sized` 在 trait 定义中默认生效。

r[dynamic-sized.struct-field]
* 结构体可以包含一个 <abbr title="dynamically sized type">DST</abbr> 作为最后一个字段；这使得结构体本身成为 <abbr title="dynamically sized type">DST</abbr>。

> [!NOTE]
> [变量][Variables]、函数参数、[const] 项和 [static] 项必须是 `Sized`。

[sized]: special-types-and-traits.md#sized
[Slices]: types/slice.md
[str]: types/str.md
[trait objects]: types/trait-object.md
[Pointer types]: types/pointer.md
[Variables]: variables.md
[const]: items/constant-items.md
[static]: items/static-items.md
