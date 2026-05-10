r[type.pointer]
# 指针类型

r[type.pointer.intro]
所有指针都是显式的一等值。它们可以被移动或复制，存储在数据结构中，并从函数返回。

r[type.pointer.reference]
## 引用（`&` 和 `&mut`）

r[type.pointer.reference.syntax]
```grammar,types
ReferenceType -> `&` Lifetime? `mut`? TypeNoBounds
```

r[type.pointer.reference.shared]
### 共享引用（`&`）

r[type.pointer.reference.shared.intro]
共享引用指向由某个其他值拥有的内存。

r[type.pointer.reference.shared.constraint-mutation]
当创建对值的共享引用时，它会阻止该值的直接修改。[内部可变性][Interior mutability]在某些情况下提供了对此的例外。顾名思义，可以对一个值存在任意数量的共享引用。共享引用类型写作 `&type`，或者当你需要指定显式生命周期时写作 `&'a type`。

r[type.pointer.reference.shared.copy]
复制引用是一种"浅"操作：它只涉及复制指针本身，也就是说，指针是 `Copy` 的。释放引用对其指向的值没有影响，但对[临时值][temporary value]的引用会在引用自身的作用域期间保持该临时值的存活。

r[type.pointer.reference.mut]
### 可变引用（`&mut`）

r[type.pointer.reference.mut.intro]
可变引用指向由某个其他值拥有的内存。可变引用类型写作 `&mut type` 或 `&'a mut type`。

r[type.pointer.reference.mut.copy]
可变引用（未被借出的）是访问其指向值的唯一方式，因此不是 `Copy` 的。

r[type.pointer.raw]
## 裸指针（`*const` 和 `*mut`）

r[type.pointer.raw.syntax]
```grammar,types
RawPointerType -> `*` ( `mut` | `const` ) TypeNoBounds
```

r[type.pointer.raw.intro]
裸指针是没有安全或活性保证的指针。裸指针写作 `*const T` 或 `*mut T`。例如，`*const i32` 表示指向 32 位整数的裸指针。

r[type.pointer.raw.copy]
复制或丢弃裸指针不会对任何其他值的生命周期产生任何影响。

r[type.pointer.raw.safety]
解引用裸指针是 [`unsafe` 操作][`unsafe` operation]。

这也可以用来通过重新借用（`&*` 或 `&mut *`）将裸指针转换为引用。通常不鼓励使用裸指针；它们存在是为了支持与外部代码的互操作，以及编写性能关键或底层函数。

r[type.pointer.raw.cmp]
比较裸指针时，按地址进行比较，而不是按其指向的内容进行比较。比较指向[动态大小类型][dynamically sized types]的裸指针时，还会比较它们的附加数据。

r[type.pointer.raw.constructor]
裸指针可以直接使用 `&raw const`（用于 `*const` 指针）和 `&raw mut`（用于 `*mut` 指针）来创建。

r[type.pointer.smart]
## 智能指针

标准库包含了引用和裸指针之外的额外"智能指针"类型。

r[type.pointer.validity]
## 位有效性

r[type.pointer.validity.pointer-fragment]
尽管在大多数平台上生成的机器码中，指针和引用与 `usize` 类似，但将引用或指针类型转换为非指针类型的语义目前尚未确定。因此，将指针或引用类型 `P` 转换为 `[u8; size_of::<P>()]` 可能不是有效的。

r[type.pointer.validity.raw]
对于瘦裸指针（即对于 `T: Sized` 的 `P = *const T` 或 `P = *mut T`），反向（从整数或整数数组转换为 `P`）始终是有效的。然而，通过这样的转换产生的指针可能不能被解引用（即使 `T` 具有[零大小][size zero]也不行）。

[Interior mutability]: ../interior-mutability.md
[`unsafe` operation]: ../unsafety.md
[dynamically sized types]: ../dynamically-sized-types.md
[size zero]: glossary.zst
[temporary value]: ../expressions.md#temporaries
