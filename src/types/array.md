r[type.array]
# 数组类型

r[type.array.syntax]
```grammar,types
ArrayType -> `[` Type `;` Expression `]`
```

r[type.array.intro]
数组是包含 `N` 个类型为 `T` 的元素的固定大小序列。数组类型写作 `[T; N]`。

r[type.array.constraint]
大小是一个求值为 [`usize`] 的[常量表达式][constant expression]。

示例：

```rust
// 栈分配的数组
let array: [i32; 3] = [1, 2, 3];

// 堆分配的数组，强制转换为切片
let boxed_array: Box<[i32]> = Box::new([1, 2, 3]);
```

r[type.array.index]
数组的所有元素始终是初始化的，并且在安全方法和运算符中访问数组时始终进行边界检查。

> [!NOTE]
> [`Vec<T>`] 标准库类型提供了一种堆分配的可变大小数组类型。

[`usize`]: numeric.md#machine-dependent-integer-types
[constant expression]: ../const_eval.md#constant-expressions
