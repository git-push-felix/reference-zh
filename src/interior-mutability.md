r[interior-mut]
# 内部可变性

r[interior-mut.intro]
有时一个类型需要在其存在多个别名的情况下被修改。在 Rust 中，这通过一种称为*内部可变性*的模式实现。

r[interior-mut.shared-ref]
如果一个类型的内部状态可以通过指向它的[共享引用][shared reference]来改变，则该类型具有内部可变性。

r[interior-mut.no-constraint]
这违背了通常的[要求][ub]，即共享引用指向的值不被修改。

r[interior-mut.unsafe-cell]
[`std::cell::UnsafeCell<T>`] 类型是禁用此要求的唯一允许方式。当 `UnsafeCell<T>` 被不可变地别名时，仍然可以安全地修改或获取对其包含的 `T` 的可变引用。

r[interior-mut.mut-unsafe-cell]
与所有其他类型一样，拥有多个 `&mut UnsafeCell<T>` 别名是未定义行为。

r[interior-mut.abstraction]
可以通过使用 `UnsafeCell<T>` 作为字段来创建其他具有内部可变性的类型。标准库提供了各种提供安全内部可变性 API 的类型。

r[interior-mut.ref-cell]
例如，[`std::cell::RefCell<T>`] 使用运行时借用检查来确保围绕多个引用的通常规则。

r[interior-mut.atomic]
[`std::sync::atomic`] 模块包含包装了仅通过原子操作访问的值的类型，允许该值在线程之间共享和修改。

[shared reference]: types/pointer.md#shared-references-
[ub]: behavior-considered-undefined.md
