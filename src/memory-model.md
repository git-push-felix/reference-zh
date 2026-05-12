r[memory]
# 内存模型

> [!WARNING]
> Rust 的内存模型尚不完整，且未完全确定。

r[memory.bytes]
## 字节 {#bytes}

r[memory.bytes.intro]
Rust 中最基本的内存单元是字节。

> [!NOTE]
> 尽管字节通常被降低到硬件字节，Rust 使用一种"抽象"的字节概念，可以做出硬件中不存在的区分，例如未初始化或存储了指针的一部分。这些区分可能影响你的程序是否具有未定义行为，因此它们仍然对编译后的 Rust 程序的行为有实际影响。

r[memory.bytes.contents]
每个字节可能具有以下值之一：

r[memory.bytes.init]
* 一个已初始化字节，包含一个 `u8` 值和可选的[来源][std::ptr#provenance]，

r[memory.bytes.uninit]
* 一个未初始化字节。

> [!NOTE]
> 上述列表尚未保证是详尽无遗的。
