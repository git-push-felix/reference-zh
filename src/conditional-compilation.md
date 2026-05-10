r[cfg]
# 条件编译

r[cfg.syntax]
```grammar,configuration
ConfigurationPredicate ->
      ConfigurationOption
    | ConfigurationAll
    | ConfigurationAny
    | ConfigurationNot
    | `true`
    | `false`

ConfigurationOption ->
    IDENTIFIER ( `=` ( STRING_LITERAL | RAW_STRING_LITERAL ) )?

ConfigurationAll ->
    `all` `(` ConfigurationPredicateList? `)`

ConfigurationAny ->
    `any` `(` ConfigurationPredicateList? `)`

ConfigurationNot ->
    `not` `(` ConfigurationPredicate `)`

ConfigurationPredicateList ->
    ConfigurationPredicate (`,` ConfigurationPredicate)* `,`?
```

r[cfg.intro]
*条件编译的源代码*是仅在特定条件下才被编译的源代码。

r[cfg.attributes-macro]
源代码可以通过 [`cfg`] 和 [`cfg_attr`] [属性][attributes]以及内置的 [`cfg!`] 和 [`cfg_select!`] [宏][macros]来进行条件编译。

r[cfg.conditional]
是否编译可以取决于被编译 crate 的目标架构、传递给编译器的任意值，以及下文所述的其他因素。

r[cfg.predicate]
每种条件编译形式都接受一个求值为 true 或 false 的*配置谓词*。谓词可以是以下之一：

r[cfg.predicate.option]
* 一个配置选项。如果该选项被设置，则谓词为 true；如果未设置，则为 false。

r[cfg.predicate.all]
* `all()`，后跟逗号分隔的配置谓词列表。如果所有给定谓词都为 true，或列表为空，则结果为 true。

r[cfg.predicate.any]
* `any()`，后跟逗号分隔的配置谓词列表。如果至少有一个给定谓词为 true，则结果为 true。如果没有谓词，则结果为 false。

r[cfg.predicate.not]
* `not()`，后跟一个配置谓词。如果其谓词为 false，则结果为 true；如果其谓词为 true，则结果为 false。

r[cfg.predicate.literal]
* `true` 或 `false` 字面量，分别始终为 true 或始终为 false。

r[cfg.option-spec]
*配置选项*可以是名称或键值对，它们要么被设置，要么未被设置。

r[cfg.option-name]
名称写作单个标识符，如 `unix`。

r[cfg.option-key-value]
键值对写作一个标识符、`=` 和一个字符串，如 `target_arch = "x86_64"`。

> [!NOTE]
> `=` 周围的空白会被忽略，因此 `foo="bar"` 和 `foo = "bar"` 是等价的。

r[cfg.option-key-uniqueness]
键不需要唯一。例如，`feature = "std"` 和 `feature = "serde"` 可以同时被设置。

r[cfg.options.set]
## 已设置的配置选项

r[cfg.options.intro]
哪些配置选项被设置在 crate 编译期间静态确定。

r[cfg.options.target]
有些选项是*编译器设置的*，基于关于编译的数据。

r[cfg.options.other]
其他选项是*任意设置的*，基于在代码之外传递给编译器的输入。

r[cfg.options.crate]
无法从正在编译的 crate 的源代码内部设置配置选项。

> [!NOTE]
> 对于 `rustc`，任意设置的配置选项使用 [`--cfg`] 标志来设置。可以通过 `rustc --print cfg --target $TARGET` 显示指定目标的配置值。

> [!NOTE]
> 键为 `feature` 的配置选项是 [Cargo][cargo-feature] 用于指定编译时选项和可选依赖的一项约定。

r[cfg.target_arch]
### `target_arch`

r[cfg.target_arch.def]
键值选项，设置一次，值为目标的 CPU 架构。该值类似于平台目标三元组的第一个元素，但不完全相同。

r[cfg.target_arch.values]
示例值：

* `"x86"`
* `"x86_64"`
* `"mips"`
* `"powerpc"`
* `"powerpc64"`
* `"arm"`
* `"aarch64"`

r[cfg.target_feature]
### `target_feature`

r[cfg.target_feature.def]
键值选项，为当前编译目标可用的每个平台特性设置。

r[cfg.target_feature.values]
示例值：

* `"avx"`
* `"avx2"`
* `"crt-static"`
* `"rdrand"`
* `"sse"`
* `"sse2"`
* `"sse4.1"`

参见 [`target_feature` 属性][`target_feature` attribute]了解更多关于可用特性的详细信息。

r[cfg.target_feature.crt_static]
`target_feature` 选项还有一个额外特性 `crt-static`，表示可用的[静态 C 运行时][static C runtime]。

r[cfg.target_os]
### `target_os`

r[cfg.target_os.def]
键值选项，设置一次，值为目标的操作系统。该值类似于平台目标三元组的第二个和第三个元素。

r[cfg.target_os.values]
示例值：

* `"windows"`
* `"macos"`
* `"ios"`
* `"linux"`
* `"android"`
* `"freebsd"`
* `"dragonfly"`
* `"openbsd"`
* `"netbsd"`
* `"none"`（典型用于嵌入式目标）

r[cfg.target_family]
### `target_family`

r[cfg.target_family.def]
键值选项，提供目标的更通用描述，例如目标通常属于的操作系统或架构家族。可以设置任意数量的 `target_family` 键值对。

r[cfg.target_family.values]
示例值：

* `"unix"`
* `"windows"`
* `"wasm"`
* 同时 `"unix"` 和 `"wasm"`

r[cfg.target_family.unix]
### `unix` 和 `windows`

如果设置了 `target_family = "unix"`，则 `unix` 被设置。

r[cfg.target_family.windows]
如果设置了 `target_family = "windows"`，则 `windows` 被设置。

r[cfg.target_env]
### `target_env`

r[cfg.target_env.def]
键值选项，设置关于目标平台的进一步消歧信息，包含关于所用 ABI 或 `libc` 的信息。由于历史原因，此值仅在需要消歧时才定义为非空字符串。因此，例如在许多 GNU 平台上，此值将为空。该值类似于平台目标三元组的第四个元素。一个不同之处在于，像 `gnueabihf` 这样的嵌入式 ABI 将简单地将 `target_env` 定义为 `"gnu"`。

r[cfg.target_env.values]
示例值：

* `""`
* `"gnu"`
* `"msvc"`
* `"musl"`
* `"sgx"`
* `"sim"`
* `"macabi"`

r[cfg.target_abi]
### `target_abi`

r[cfg.target_abi.def]
键值选项，设置为进一步消歧目标，包含关于目标 ABI 的信息。

r[cfg.target_abi.disambiguation]
由于历史原因，此值仅在需要消歧时才定义为非空字符串。因此，例如在许多 GNU 平台上，此值将为空。

r[cfg.target_abi.values]
示例值：

* `""`
* `"llvm"`
* `"eabihf"`
* `"abi64"`

r[cfg.target_endian]
### `target_endian`

键值选项，设置一次，值为 `"little"` 或 `"big"`，取决于目标 CPU 的字节序。

r[cfg.target_pointer_width]
### `target_pointer_width`

r[cfg.target_pointer_width.def]
键值选项，设置一次，值为目标的指针宽度（以位为单位）。

r[cfg.target_pointer_width.values]
示例值：

* `"16"`
* `"32"`
* `"64"`

r[cfg.target_vendor]
### `target_vendor`

r[cfg.target_vendor.def]
键值选项，设置一次，值为目标的供应商。

r[cfg.target_vendor.values]
示例值：

* `"apple"`
* `"fortanix"`
* `"pc"`
* `"unknown"`

r[cfg.target_has_atomic]
### `target_has_atomic`

r[cfg.target_has_atomic.def]
键值选项，为目标支持原子加载、存储和比较并交换操作的每个位宽设置。

r[cfg.target_has_atomic.stdlib]
当此 cfg 存在时，所有稳定的 [`core::sync::atomic`] API 对相应的原子位宽都可用。

r[cfg.target_has_atomic.values]
可能的值：

* `"8"`
* `"16"`
* `"32"`
* `"64"`
* `"128"`
* `"ptr"`

r[cfg.test]
### `test`

在编译测试框架时启用。通过使用 [`--test`] 标志与 `rustc` 一起完成。更多关于测试支持的信息，请参见[测试][Testing]。

r[cfg.debug_assertions]
### `debug_assertions`

在不进行优化编译时默认启用。这可以用于在开发中启用额外的调试代码，而在生产中不启用。例如，它控制标准库的 [`debug_assert!`] 宏的行为。

r[cfg.proc_macro]
### `proc_macro`

当正在编译的 crate 以 `proc_macro` [crate 类型][crate type]编译时设置。

r[cfg.panic]
### `panic`

r[cfg.panic.def]
键值选项，根据 [panic 策略][panic strategy]设置。注意将来可能会添加更多值。

r[cfg.panic.values]
示例值：

* `"abort"`
* `"unwind"`

[panic strategy]: panic.md#panic-strategy

## 条件编译的形式

<!-- template:attributes -->
r[cfg.attr]
### `cfg` 属性 {#the-cfg-attribute}

r[cfg.attr.intro]
*`cfg` [属性][attribute]* 根据配置谓词有条件地包含它所附加的形式。

> [!EXAMPLE]
> ```rust
> // 该函数仅在为 macOS 编译时包含在构建中
> #[cfg(target_os = "macos")]
> fn macos_only() {
>   // ...
> }
>
> // 该函数仅在 foo 或 bar 被定义时包含
> #[cfg(any(foo, bar))]
> fn needs_foo_or_bar() {
>   // ...
> }
>
> // 该函数仅在为类 Unix 操作系统且 32 位架构编译时包含
> #[cfg(all(unix, target_pointer_width = "32"))]
> fn on_32bit_unix() {
>   // ...
> }
>
> // 该函数仅在 foo 未被定义时包含
> #[cfg(not(foo))]
> fn needs_not_foo() {
>   // ...
> }
>
> // 该函数仅在 panic 策略设置为 unwind 时包含
> #[cfg(panic = "unwind")]
> fn when_unwinding() {
>   // ...
> }
> ```

r[cfg.attr.syntax]
`cfg` 属性的语法为：

```grammar,configuration
@root CfgAttribute -> `cfg` `(` ConfigurationPredicate `)`
```

r[cfg.attr.allowed-positions]
`cfg` 属性可以用于任何允许属性的地方。

r[cfg.attr.duplicates]
`cfg` 属性可以在同一个形式上使用任意次数。如果任何 `cfg` 谓词为 false，则属性所附加的形式将不会被包含，除非 [cfg.attr.crate-level-attrs] 中所述的情况。

r[cfg.attr.effect]
如果谓词为 true，则该形式被重写为不带 `cfg` 属性。如果任何谓词为 false，则该形式从源代码中移除。

r[cfg.attr.crate-level-attrs]
当 crate 级别的 `cfg` 具有 false 谓词时，crate 本身仍然存在。`cfg` 之前的任何 crate 属性会被保留，而 `cfg` 之后的任何 crate 属性会被移除，同时移除所有后续的 crate 内容。

> [!EXAMPLE]
> 不移除前置属性的行为允许你做诸如包含 `#![no_std]` 来避免链接 `std` 的事情，即使 `#![cfg(...)]` 已经移除了 crate 的内容。例如：
>
> <!-- ignore: test infrastructure can't handle no_std -->
> ```rust,ignore
> // 即使 crate 级别的 `cfg` 属性为 false，此 `no_std` 属性也会被保留。
> #![no_std]
> #![cfg(false)]
>
> // 此函数不会被包含。
> pub fn example() {}
> ```

<!-- template:attributes -->
r[cfg.cfg_attr]
### `cfg_attr` 属性 {#the-cfg_attr-attribute}

r[cfg.cfg_attr.intro]
*`cfg_attr` [属性][attribute]* 根据配置谓词有条件地包含属性。

> [!EXAMPLE]
> 以下模块将根据目标在 `linux.rs` 或 `windows.rs` 中找到。
>
> <!-- ignore: `mod` needs multiple files -->
> ```rust,ignore
> #[cfg_attr(target_os = "linux", path = "linux.rs")]
> #[cfg_attr(windows, path = "windows.rs")]
> mod os;
> ```

r[cfg.cfg_attr.syntax]
`cfg_attr` 属性的语法为：

```grammar,configuration
@root CfgAttrAttribute -> `cfg_attr` `(` ConfigurationPredicate `,` CfgAttrs? `)`

CfgAttrs -> Attr (`,` Attr)* `,`?
```

r[cfg.cfg_attr.allowed-positions]
`cfg_attr` 属性可以用于任何允许属性的地方。

r[cfg.cfg_attr.duplicates]
`cfg_attr` 属性可以在同一个形式上使用任意次数。

r[cfg.cfg_attr.attr-restriction]
[`crate_type`] 和 [`crate_name`] 属性不能与 `cfg_attr` 一起使用。

r[cfg.cfg_attr.behavior]
当配置谓词为 true 时，`cfg_attr` 展开为谓词之后列出的属性。

r[cfg.cfg_attr.attribute-list]
可以列出零个、一个或多个属性。多个属性将被分别展开为单独的属性。

> [!EXAMPLE]
> <!-- ignore: fake attributes -->
> ```rust,ignore
> #[cfg_attr(feature = "magic", sparkles, crackles)]
> fn bewitched() {}
>
> // 当 `magic` 特性标志启用时，以上代码将展开为：
> #[sparkles]
> #[crackles]
> fn bewitched() {}
> ```

> [!NOTE]
> `cfg_attr` 可以展开为另一个 `cfg_attr`。例如，`#[cfg_attr(target_os = "linux", cfg_attr(feature = "multithreaded", some_other_attribute))]` 是有效的。此示例将等价于 `#[cfg_attr(all(target_os = "linux", feature = "multithreaded"), some_other_attribute)]`。

r[cfg.macro]
### `cfg` 宏 {#the-cfg-macro}

内置的 `cfg` 宏接受单个配置谓词，当谓词为 true 时求值为 `true` 字面量，为 false 时求值为 `false` 字面量。

例如：

```rust
let machine_kind = if cfg!(unix) {
  "unix"
} else if cfg!(windows) {
  "windows"
} else {
  "unknown"
};

println!("I'm running on a {} machine!", machine_kind);
```

r[cfg.cfg_select]
### `cfg_select` 宏 {#the-cfg_select-macro}

r[cfg.cfg_select.intro]
内置的 [`cfg_select!`][std::cfg_select] 宏可用于在编译期根据多个配置谓词选择代码。

> [!EXAMPLE]
> ```rust
> cfg_select! {
>     unix => {
>         fn foo() { /* unix 特定功能 */ }
>     }
>     target_pointer_width = "32" => {
>         fn foo() { /* 非 unix、32 位功能 */ }
>     }
>     _ => {
>         fn foo() { /* 回退实现 */ }
>     }
> }
>
> let is_unix_str = cfg_select! {
>     unix => "unix",
>     _ => "not unix",
> };
> ```

r[cfg.cfg_select.syntax]
```grammar,configuration
@root CfgSelect -> CfgSelectArms?

CfgSelectArms ->
    CfgSelectConfigurationPredicate `=>`
    (
        `{` ^ TokenTree `}` `,`? CfgSelectArms?
      | ExpressionWithBlockNoAttrs `,`? CfgSelectArms?
      | ExpressionWithoutBlockNoAttrs ( `,` CfgSelectArms? )?
    )

CfgSelectConfigurationPredicate ->
    ConfigurationPredicate | `_`
```

r[cfg.cfg_select.first-arm]
`cfg_select` 展开为第一个其配置谓词求值为 true 的分支的载荷。

r[cfg.cfg_select.braces]
如果整个载荷被花括号包裹，展开时会移除花括号。

r[cfg.cfg_select.wildcard]
配置谓词 `_` 始终求值为 true。

r[cfg.cfg_select.fallthrough]
如果没有任何谓词求值为 true，则是编译错误。

r[cfg.cfg_select.well-formed]
每个右侧必须是对于宏调用所在位置语法有效的展开。

[Testing]: attributes/testing.md
[`--cfg`]: ../rustc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[`--test`]: ../rustc/command-line-arguments.html#--test-build-a-test-harness
[`cfg`]: #the-cfg-attribute
[`cfg!`]: #the-cfg-macro
[`cfg_attr`]: #the-cfg_attr-attribute
[`cfg_select!`]: #the-cfg_select-macro
[`crate_name`]: crates-and-source-files.md#the-crate_name-attribute
[`crate_type`]: linkage.md
[`target_feature` attribute]: attributes/codegen.md#the-target_feature-attribute
[attribute]: attributes.md
[attributes]: attributes.md
[cargo-feature]: ../cargo/reference/features.html
[crate type]: linkage.md
[macros]: macros.md
[static C runtime]: linkage.md#static-and-dynamic-c-runtimes
