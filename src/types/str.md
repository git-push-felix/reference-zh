r[type.str]
# 字符串切片类型

r[type.str.intro]
字符串切片（`str`）类型表示一个字符序列。

```rust
let greeting1: &str = "Hello, world!";
let greeting2: &str = "你好，世界";
```

> [!NOTE]
> 有关 `str` 类型的实现信息，请参阅[标准库文档][`str`]。

r[type.str.value]
`str` 类型的值以与 `[u8]`（8 位无符号字节切片）相同的方式表示。

> [!NOTE]
> 标准库对 `str` 有额外的假定：操作 `str` 的方法假定并确保其中包含的数据是有效的 UTF-8。使用非 UTF-8 缓冲区调用 `str` 方法可能现在或将来引发[未定义行为][undefined behavior]。

r[type.str.unsized]
`str` 是[动态大小类型][dynamically sized type]。它只能通过指针类型（如 `&str`）进行实例化。`&str` 的布局与 `&[u8]` 的布局相同。

[undefined behavior]: ../behavior-considered-undefined.md
[dynamically sized type]: ../dynamically-sized-types.md
