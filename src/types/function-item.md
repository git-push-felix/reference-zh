r[type.fn-item]
# 函数项类型

r[type.fn-item.intro]
当被引用时，函数项，或类元组结构体或枚举变体的构造函数，会产生一个其*函数项类型*的[零大小][zero-sized]值。

r[type.fn-item.unique]
该类型显式地标识了函数——其名称、类型参数和早期绑定生命周期参数（但不包括晚期绑定生命周期参数，它们仅在函数被调用时赋值）——因此该值不需要包含实际的函数指针，调用函数时也不需要间接调用。

r[type.fn-item.name]
没有直接引用函数项类型的语法，但编译器会在错误消息中将类型显示为类似 `fn(u32) -> i32 {fn_name}` 的形式。

由于函数项类型显式地标识了函数，不同函数的项类型——不同的项，或具有不同泛型的同一项——是不同的，混合它们将产生类型错误：

```rust,compile_fail,E0308
fn foo<T>() { }
let x = &mut foo::<i32>;
*x = foo::<u32>; //~ 错误：类型不匹配
```

r[type.fn-item.coercion]
不过，存在从函数项到具有相同签名的[函数指针][function pointers]的[强制转换][coercion]，这种转换不仅在函数项直接用于期望函数指针的位置时触发，也在具有相同签名的不同函数项类型出现在同一 `if` 或 `match` 的不同分支中时触发：

```rust
# let want_i32 = false;
# fn foo<T>() { }

// `foo_ptr_1` 在此处的类型为函数指针 `fn()`
let foo_ptr_1: fn() = foo::<i32>;

// ...而 `foo_ptr_2` 同样也是——类型检查通过。
let foo_ptr_2 = if want_i32 {
    foo::<i32>
} else {
    foo::<u32>
};
```

r[type.fn-item.traits]
所有函数项都实现了 [`Copy`]、[`Clone`]、[`Send`] 和 [`Sync`]。

[`Fn`]、[`FnMut`] 和 [`FnOnce`] 被实现，除非函数具有以下任何一项：

- [`unsafe`][unsafe.fn] 限定符
- [`target_feature` 属性][attributes.codegen.target_feature]
- 除 `"Rust"` 之外的 [ABI][items.fn.extern]

[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[coercion]: ../type-coercions.md
[function pointers]: function-pointer.md
[zero-sized]: glossary.zst
