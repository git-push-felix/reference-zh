r[type.slice]
# 切片类型

r[type.slice.syntax]
```grammar,types
SliceType -> `[` Type `]`
```

r[type.slice.intro]
切片是一种[动态大小类型][dynamically sized type]，表示类型为 `T` 的元素序列的一个"视图"。切片类型写作 `[T]`。

r[type.slice.unsized]
切片类型通常通过指针类型使用。例如：

* `&[T]`：一个"共享切片"，通常直接简称为"切片"。它不拥有所指向的数据；它只是借用。
* `&mut [T]`：一个"可变切片"。它可变地借用所指向的数据。
* `Box<[T]>`：一个"装箱切片"

示例：

```rust
// 堆分配的数组，强制转换为切片
let boxed_array: Box<[i32]> = Box::new([1, 2, 3]);

// 数组上的（共享）切片
let slice: &[i32] = &boxed_array[..];
```

r[type.slice.safe]
切片的所有元素始终是初始化的，并且在安全方法和运算符中访问切片时始终进行边界检查。

[dynamically sized type]: ../dynamically-sized-types.md
