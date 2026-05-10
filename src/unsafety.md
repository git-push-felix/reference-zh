r[safety]
# 不安全操作

r[safety.intro]
不安全操作是指那些可能违反 Rust 静态语义的内存安全保证的操作。

r[safety.unsafe-ops]
以下语言级特性不能在 Rust 的安全子集中使用：

r[safety.unsafe-deref]
- 解引用[裸指针][raw pointer]。

r[safety.unsafe-static]
- 读取或写入[可变][mutable]或不安全的[外部][external]静态变量。

r[safety.unsafe-union-access]
- 访问 [`union`] 的字段，除非是为了赋值。

r[safety.unsafe-call]
- 调用不安全的函数。

r[safety.unsafe-target-feature-call]
- 从一个没有启用相应 `target_feature` 特性的函数中，调用带有 [`target_feature`][attributes.codegen.target_feature] 标记的安全函数（参见 [attributes.codegen.target_feature.safety-restrictions]）。

r[safety.unsafe-impl]
- 实现[不安全 trait][unsafe trait]。

r[safety.unsafe-extern]
- 声明 [`extern`] 块[^extern-2024]。

r[safety.unsafe-attribute]
- 对项应用[不安全属性][unsafe attribute]。

[^extern-2024]: 在 2024 版之前，`extern` 块允许不带 `unsafe` 声明。

[`extern`]: items/external-blocks.md
[`union`]: items/unions.md
[mutable]: items/static-items.md#mutable-statics
[external]: items/external-blocks.md
[raw pointer]: types/pointer.md
[unsafe trait]: items/traits.md#unsafe-traits
[unsafe attribute]: attributes.md
