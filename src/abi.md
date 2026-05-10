r[abi]
# 应用程序二进制接口（ABI）

r[abi.intro]
本节记录了影响 crate 编译输出的 ABI 的特性。

相关信息请参阅 *[extern 函数][extern functions]* 以了解如何指定导出函数的 ABI，参阅 *[外部块][external blocks]* 以了解如何指定链接外部库的 ABI。

<!-- template:attributes -->
r[abi.used]
## `used` 属性

r[abi.used.intro]
*`used` [属性][attribute]* 强制将一个 [static] 保留在输出目标文件（.o、.rlib 等，但不包括最终二进制文件）中，即使它在 crate 中从未被其他程序项使用或引用。不过，链接器仍然可以自由地移除它。

> [!EXAMPLE]
> ```rust
> // lib.rs
>
> // 因为有 `#[used]`，此静态项被保留。
> #[used]
> static S1: u8 = 0;
>
> // 因为未使用，此静态项可被移除。
> #[allow(dead_code)]
> static S2: u8 = 0;
>
> // 因为可被公开访问，此静态项被保留。
> pub static S3: u8 = 0;
>
> // 因为被一个可公开访问的函数引用，此静态项被保留。
> static S4: u8 = 0;
> #[unsafe(no_mangle)] pub fn f4() -> &'static u8 { &S4 }
>
> // 因为只被一个私有的、未使用的（死）函数引用，此静态项可被移除。
> static S5: u8 = 0;
> #[allow(dead_code)]
> fn f5() -> &'static u8 { &S5 }
> ```
>
> ```console
> $ rustc -O --emit=obj --crate-type=rlib lib.rs
> $ LC_ALL=C nm -C lib.o
> 0000000000000000 R lib::S1
> 0000000000000000 R lib::S3
> 0000000000000000 r lib::S4
> 0000000000000000 T f4
> ```

r[abi.used.syntax]
`used` 属性使用 [MetaWord] 语法。

r[abi.used.allowed-positions]
`used` 属性只能应用于 [`static` 程序项][`static` items]。

r[abi.used.duplicates]
只有程序项上的第一次 `used` 使用才有效。

> [!NOTE]
> `rustc` 会对第一次之后的任何使用给出 lint 警告。

r[abi.no_mangle]
## `no_mangle` 属性

r[abi.no_mangle.intro]
*`no_mangle` 属性*可用于任何[程序项][item]，以禁用在符号名称上应用标准的名称修饰。该程序项的符号将是该程序项名称的标识符。

r[abi.no_mangle.publicly-exported]
此外，该程序项将从生成的库或目标文件中公开导出，类似于 [`used` 属性](#the-used-attribute)。

r[abi.no_mangle.unsafe]
此属性是不安全的，因为未修饰的符号可能与另一个同名符号（或已知符号）冲突，导致未定义行为。

```rust
#[unsafe(no_mangle)]
extern "C" fn foo() {}
```

r[abi.no_mangle.edition2024]
> [!EDITION-2024]
> 在 2024 版之前，允许在不使用 `unsafe` 限定的情况下使用 `no_mangle` 属性。

r[abi.link_section]
## `link_section` 属性

r[abi.link_section.intro]
*`link_section` 属性*指定将[函数][function]或 [static] 的内容放入目标文件的哪个节中。

r[abi.link_section.syntax]
`link_section` 属性使用 [MetaNameValueStr] 语法来指定节名称。

<!-- no_run: don't link. The format of the section name is platform-specific. -->
```rust,no_run
# #[cfg(target_os = "linux")] {
#[unsafe(no_mangle)]
#[unsafe(link_section = ".example_section")]
pub static VAR1: u32 = 1;
# }
```

r[abi.link_section.unsafe]
此属性是不安全的，因为它允许用户将数据和代码放入未预期它们的内存节中，例如将可变数据放入只读区域。

r[abi.link_section.duplicates]
只有程序项上的第一次 `link_section` 使用才有效。

> [!NOTE]
> `rustc` 会对第一次之后的任何使用给出未来兼容性警告。这在未来可能成为错误。

r[abi.link_section.edition2024]
> [!EDITION-2024]
> 在 2024 版之前，允许在不使用 `unsafe` 限定的情况下使用 `link_section` 属性。

r[abi.export_name]
## `export_name` 属性

r[abi.export_name.intro]
*`export_name` 属性*指定将在[函数][function]或 [static] 上导出的符号名称。

r[abi.export_name.syntax]
`export_name` 属性使用 [MetaNameValueStr] 语法来指定符号名称。

```rust
#[unsafe(export_name = "exported_symbol_name")]
pub fn name_in_rust() { }
```

r[abi.export_name.unsafe]
此属性是不安全的，因为具有自定义名称的符号可能与另一个同名符号（或已知符号）冲突，导致未定义行为。

r[abi.export_name.duplicates]
只有程序项上的第一次 `export_name` 使用才有效。

> [!NOTE]
> `rustc` 会对第一次之后的任何使用给出未来兼容性警告。这在未来可能成为错误。

r[abi.export_name.edition2024]
> [!EDITION-2024]
> 在 2024 版之前，允许在不使用 `unsafe` 限定的情况下使用 `export_name` 属性。

[attribute]: attributes.md
[extern functions]: items/functions.md#extern-function-qualifier
[external blocks]: items/external-blocks.md
[function]: items/functions.md
[item]: items.md
[`static` items]: items.static
[static]: items/static-items.md
