r[attributes.codegen]
# 代码生成属性

以下[属性][attributes]用于控制代码生成。

<!-- template:attributes -->
r[attributes.codegen.inline]
### `inline` 属性

r[attributes.codegen.inline.intro]
*`inline` [属性][attribute]* 建议是将带属性函数的代码副本放置在调用者中，还是生成对函数的调用。

> [!EXAMPLE]
> ```rust
> #[inline]
> pub fn example1() {}
>
> #[inline(always)]
> pub fn example2() {}
>
> #[inline(never)]
> pub fn example3() {}
> ```

> [!NOTE]
> `rustc` 在认为值得时会自动内联函数。请谨慎使用此属性，因为关于内联哪些内容的不当决定可能会降低程序速度。

r[attributes.codegen.inline.syntax]
`inline` 属性的语法如下：

```grammar,attributes
@root InlineAttribute ->
      `inline` `(` `always` `)`
    | `inline` `(` `never` `)`
    | `inline`
```

r[attributes.codegen.inline.allowed-positions]
`inline` 属性只能应用于具有[函数体][bodies]的函数 --- [闭包][closures]、[async 块][async blocks]、[自由函数][free functions]、[固有 impl][inherent impl] 或 [trait impl][trait impl] 中的[关联函数][associated functions]，以及具有[默认定义][default definition]时 [trait 定义][trait definition]中的关联函数。

> [!NOTE]
> `rustc` 忽略其他位置的用法但会发出 lint 警告。这可能在将来成为错误。

> [!NOTE]
> 尽管该属性可以应用于[闭包][closures]和 [async 块][async blocks]，但这一点实用性有限，因为我们尚不支持表达式上的属性。

r[attributes.codegen.inline.duplicates]
只有第一次在函数上使用 `inline` 才有效。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。这可能在将来成为错误。

r[attributes.codegen.inline.modes]
`inline` 属性支持以下模式：

- `#[inline]` *建议*执行内联展开。
- `#[inline(always)]` *建议*始终执行内联展开。
- `#[inline(never)]` *建议*绝不执行内联展开。

> [!NOTE]
> 在每种形式中，该属性都是一个提示。编译器可能会忽略它。

r[attributes.codegen.inline.trait]
当 `inline` 应用于 [trait] 中的函数时，它仅适用于[默认定义][default definition]的代码。

r[attributes.codegen.inline.async]
当 `inline` 应用于 [async 函数][async function] 或 [async 闭包][async closure] 时，它仅适用于生成的 `poll` 函数的代码。

> [!NOTE]
> 更多细节，请参见 [Rust issue #129347](https://github.com/rust-lang/rust/issues/129347)。

r[attributes.codegen.inline.externally-exported]
如果函数通过 [`no_mangle`] 或 [`export_name`] 外部导出，则 `inline` 属性被忽略。

<!-- template:attributes -->
r[attributes.codegen.cold]
### `cold` 属性

r[attributes.codegen.cold.intro]
*`cold` [属性][attribute]* 建议带属性的函数不太可能被调用，这可能帮助编译器生成更好的代码。

> [!EXAMPLE]
> ```rust
> #[cold]
> pub fn example() {}
> ```

r[attributes.codegen.cold.syntax]
`cold` 属性使用 [MetaWord] 语法。

r[attributes.codegen.cold.allowed-positions]
`cold` 属性只能应用于具有[函数体][bodies]的函数。

r[attributes.codegen.cold.duplicates]
只有第一次在函数上使用 `cold` 才有效。

r[attributes.codegen.cold.trait]
当 `cold` 应用于 [trait] 中的函数时，它仅适用于[默认定义][default definition]的代码。

r[attributes.codegen.naked]
## `naked` 属性 {#the-naked-attribute}

r[attributes.codegen.naked.intro]
*`naked` [属性][attribute]* 阻止编译器为带属性的函数生成函数序言和尾声。

r[attributes.codegen.naked.body]
[函数体][function body]必须恰好由一个 [`naked_asm!`] 宏调用组成。

r[attributes.codegen.naked.prologue-epilogue]
不会为带属性的函数生成函数序言或尾声。`naked_asm!` 块中的汇编代码构成裸函数的完整函数体。

r[attributes.codegen.naked.unsafe-attribute]
`naked` 属性是一个 [unsafe 属性][unsafe attribute]。使用 `#[unsafe(naked)]` 标注函数附带的安全性义务是：函数体必须遵守函数的调用约定、履行其签名，并且要么返回要么发散（即不越过汇编代码的末尾而掉落）。

r[attributes.codegen.naked.call-stack]
汇编代码可以假设在入口时调用栈和寄存器状态根据函数的签名和调用约定是有效的。

r[attributes.codegen.naked.no-duplication]
汇编代码不能被编译器复制，除非在单态化多态函数时。

> [!NOTE]
> 保证汇编代码何时可能被复制或不被复制对于定义符号的裸函数很重要。

r[attributes.codegen.naked.unused-variables]
[`unused_variables`] lint 在裸函数中被抑制。

r[attributes.codegen.naked.inline]
[`inline`](#inline-属性) 属性不能应用于裸函数。

r[attributes.codegen.naked.track_caller]
[`track_caller`](#track_caller-属性) 属性不能应用于裸函数。

r[attributes.codegen.naked.testing]
[测试属性](testing.md)不能应用于裸函数。

<!-- template:attributes -->
r[attributes.codegen.no_builtins]
## `no_builtins` 属性

r[attributes.codegen.no_builtins.intro]
*`no_builtins` [属性][attribute]* 禁用与调用假定存在的库函数相关的某些代码模式的优化。

> [!EXAMPLE]
> ```rust
> #![no_builtins]
> ```

r[attributes.codegen.no_builtins.syntax]
`no_builtins` 属性使用 [MetaWord] 语法。

r[attributes.codegen.no_builtins.allowed-positions]
`no_builtins` 属性只能应用于 crate 根。

r[attributes.codegen.no_builtins.duplicates]
只有第一次使用 `no_builtins` 属性才有效。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。

r[attributes.codegen.target_feature]
## `target_feature` 属性

r[attributes.codegen.target_feature.intro]
*`target_feature` [属性][attribute]* 可以应用于函数，以为特定平台架构特性启用该函数的代码生成。它使用 [MetaListNameValueStr] 语法，带有单个键 `enable`，其值是一个以逗号分隔的要启用的特性名称字符串。

```rust
# #[cfg(target_feature = "avx2")]
#[target_feature(enable = "avx2")]
fn foo_avx2() {}
```

r[attributes.codegen.target_feature.arch]
每个[目标架构][target architecture]都有一组可以启用的特性。为 crate 未编译的目标架构指定特性是错误的。

r[attributes.codegen.target_feature.closures]
在 `target_feature` 标注的函数内定义的闭包从外围函数继承该属性。

r[attributes.codegen.target_feature.target-ub]
调用使用当前平台不支持的特定平台特性编译的函数是[未定义行为][undefined behavior]，*除非*平台明确文档说明这是安全的。

r[attributes.codegen.target_feature.safety-restrictions]
除非以下平台规则另有说明，否则适用以下限制：

- 安全的 `#[target_feature]` 函数（以及继承该属性的闭包）只能在启用被调用者启用的所有 `target_feature` 的调用者中安全调用。此限制不适用于 `unsafe` 上下文。
- 安全的 `#[target_feature]` 函数（以及继承该属性的闭包）只能在启用被强制转换者启用的所有 `target_feature` 的上下文中被强制转换为*安全的*函数指针。此限制不适用于 `unsafe` 函数指针。

隐式启用的特性包含在此规则中。例如，一个 `sse2` 函数可以调用标记为 `sse` 的函数。

```rust
# #[cfg(target_feature = "sse2")] {
#[target_feature(enable = "sse")]
fn foo_sse() {}

fn bar() {
    // 在此处调用 `foo_sse` 是不安全的，因为我们必须首先确保 SSE 可用，
    // 即使目标平台默认启用了 `sse` 或作为编译器标志手动启用。
    unsafe {
        foo_sse();
    }
}

#[target_feature(enable = "sse")]
fn bar_sse() {
    // 在此处调用 `foo_sse` 是安全的。
    foo_sse();
    || foo_sse();
}

#[target_feature(enable = "sse2")]
fn bar_sse2() {
    // 在此处调用 `foo_sse` 是安全的，因为 `sse2` 隐含 `sse`。
    foo_sse();
}
# }
```

r[attributes.codegen.target_feature.fn-traits]
具有 `#[target_feature]` 属性的函数*从不*实现 `Fn` trait 族，尽管从外围函数继承特性的闭包会实现。

r[attributes.codegen.target_feature.allowed-positions]
`#[target_feature]` 属性不允许出现在以下位置：

- [`main` 函数][crate.main]
- [`panic_handler` 函数][panic.panic_handler]
- 安全的 trait 方法
- trait 中的安全默认函数

r[attributes.codegen.target_feature.inline]
标记为 `target_feature` 的函数不会内联到不支持给定特性的上下文中。`#[inline(always)]` 属性不能与 `target_feature` 属性一起使用。

r[attributes.codegen.target_feature.availability]
### 可用特性

以下是可用特性名称的列表。

r[attributes.codegen.target_feature.x86]
#### `x86` 或 `x86_64`

在此平台上执行不支持的特性的代码是未定义行为。因此在此平台上使用 `#[target_feature]` 函数遵循[上述限制][attributes.codegen.target_feature.safety-restrictions]。

特性     | 隐式启用 | 描述
------------|--------------------|-------------------
`adx`       |          | [ADX] --- 多精度加法进位指令扩展
`aes`       | `sse2`   | [AES] --- 高级加密标准
`avx`       | `sse4.2` | [AVX] --- 高级向量扩展
`avx2`      | `avx`    | [AVX2] --- 高级向量扩展 2
`avx512bf16`        | `avx512bw`           | [AVX512-BF16] --- 高级向量扩展 512 位 - Bfloat16 扩展
`avx512bitalg`      | `avx512bw`           | [AVX512-BITALG] --- 高级向量扩展 512 位 - 位算法
`avx512bw`          | `avx512f`            | [AVX512-BW] --- 高级向量扩展 512 位 - 字节和字指令
`avx512cd`          | `avx512f`            | [AVX512-CD] --- 高级向量扩展 512 位 - 冲突检测指令
`avx512dq`          | `avx512f`            | [AVX512-DQ] --- 高级向量扩展 512 位 - 双字和四字指令
`avx512f`           | `avx2`, `fma`, `f16c`| [AVX512-F] --- 高级向量扩展 512 位 - 基础
`avx512fp16`        | `avx512bw`           | [AVX512-FP16] --- 高级向量扩展 512 位 - Float16 扩展
`avx512ifma`        | `avx512f`            | [AVX512-IFMA] --- 高级向量扩展 512 位 - 整数融合乘加
`avx512vbmi`        | `avx512bw`           | [AVX512-VBMI] --- 高级向量扩展 512 位 - 向量字节操作指令
`avx512vbmi2`       | `avx512bw`           | [AVX512-VBMI2] --- 高级向量扩展 512 位 - 向量字节操作指令 2
`avx512vl`          | `avx512f`            | [AVX512-VL] --- 高级向量扩展 512 位 - 向量长度扩展
`avx512vnni`        | `avx512f`            | [AVX512-VNNI] --- 高级向量扩展 512 位 - 向量神经网络指令
`avx512vp2intersect`| `avx512f`            | [AVX512-VP2INTERSECT] --- 高级向量扩展 512 位 - 向量对交集到一对掩码寄存器
`avx512vpopcntdq`   | `avx512f`            | [AVX512-VPOPCNTDQ] --- 高级向量扩展 512 位 - 向量人口计数指令
`avxifma`           | `avx2`               | [AVX-IFMA] --- 高级向量扩展 - 整数融合乘加
`avxneconvert`      | `avx2`               | [AVX-NE-CONVERT] --- 高级向量扩展 - 无异常浮点转换指令
`avxvnni`           | `avx2`               | [AVX-VNNI] --- 高级向量扩展 - 向量神经网络指令
`avxvnniint16`      | `avx2`               | [AVX-VNNI-INT16] --- 高级向量扩展 - 16 位整数向量神经网络指令
`avxvnniint8`       | `avx2`               | [AVX-VNNI-INT8] --- 高级向量扩展 - 8 位整数向量神经网络指令
`bmi1`      |          | [BMI1] --- 位操作指令集
`bmi2`      |          | [BMI2] --- 位操作指令集 2
`cmpxchg16b`|          | [`cmpxchg16b`] --- 原子地比较和交换 16 字节（128 位）数据
`f16c`      | `avx`    | [F16C] --- 16 位浮点转换指令
`fma`       | `avx`    | [FMA3] --- 三操作数融合乘加
`fxsr`      |          | [`fxsave`] 和 [`fxrstor`] --- 保存和恢复 x87 FPU、MMX 技术和 SSE 状态
`gfni`      | `sse2`   | [GFNI] --- 伽罗瓦域新指令
`kl`        | `sse2`   | [KEYLOCKER] --- Intel Key Locker 指令
`lzcnt`     |          | [`lzcnt`] --- 前导零计数
`movbe`     |          | [`movbe`] --- 交换字节后移动数据
`pclmulqdq` | `sse2`   | [`pclmulqdq`] --- 打包无进位乘法四字
`popcnt`    |          | [`popcnt`] --- 设置为 1 的位数计数
`rdrand`    |          | [`rdrand`] --- 读取随机数
`rdseed`    |          | [`rdseed`] --- 读取随机种子
`sha`       | `sse2`   | [SHA] --- 安全哈希算法
`sha512`    | `avx2`   | [SHA512] --- 512 位摘要的安全哈希算法
`sm3`       | `avx`    | [SM3] --- 商密 3 哈希算法
`sm4`       | `avx2`   | [SM4] --- 商密 4 密码算法
`sse`       |          | [SSE] --- 流式 <abbr title="Single Instruction Multiple Data">SIMD</abbr> 扩展
`sse2`      | `sse`    | [SSE2] --- 流式 SIMD 扩展 2
`sse3`      | `sse2`   | [SSE3] --- 流式 SIMD 扩展 3
`sse4.1`    | `ssse3`  | [SSE4.1] --- 流式 SIMD 扩展 4.1
`sse4.2`    | `sse4.1` | [SSE4.2] --- 流式 SIMD 扩展 4.2
`sse4a`     | `sse3`   | [SSE4a] --- 流式 SIMD 扩展 4a
`ssse3`     | `sse3`   | [SSSE3] --- 补充流式 SIMD 扩展 3
`tbm`       |          | [TBM] --- 尾部位操作
`vaes`      | `avx2`, `aes`     | [VAES] --- 向量 AES 指令
`vpclmulqdq`| `avx`, `pclmulqdq`| [VPCLMULQDQ] --- 向量无进位四字乘法
`widekl`    | `kl`     | [KEYLOCKER_WIDE] --- Intel Wide Keylocker 指令
`xsave`     |          | [`xsave`] --- 保存处理器扩展状态
`xsavec`    |          | [`xsavec`] --- 以压缩保存处理器扩展状态
`xsaveopt`  |          | [`xsaveopt`] --- 优化保存处理器扩展状态
`xsaves`    |          | [`xsaves`] --- 以管理员模式保存处理器扩展状态

<!-- Keep links near each table to make it easier to move and update. -->

[ADX]: https://en.wikipedia.org/wiki/Intel_ADX
[AES]: https://en.wikipedia.org/wiki/AES_instruction_set
[AVX]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions
[AVX2]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX2
[AVX512-BF16]: https://en.wikipedia.org/wiki/AVX-512#BF16
[AVX512-BITALG]: https://en.wikipedia.org/wiki/AVX-512#VPOPCNTDQ_and_BITALG
[AVX512-BW]: https://en.wikipedia.org/wiki/AVX-512#BW,_DQ_and_VBMI
[AVX512-CD]: https://en.wikipedia.org/wiki/AVX-512#Conflict_detection
[AVX512-DQ]: https://en.wikipedia.org/wiki/AVX-512#BW,_DQ_and_VBMI
[AVX512-F]: https://en.wikipedia.org/wiki/AVX-512
[AVX512-FP16]: https://en.wikipedia.org/wiki/AVX-512#FP16
[AVX512-IFMA]: https://en.wikipedia.org/wiki/AVX-512#IFMA
[AVX512-VBMI]: https://en.wikipedia.org/wiki/AVX-512#BW,_DQ_and_VBMI
[AVX512-VBMI2]: https://en.wikipedia.org/wiki/AVX-512#VBMI2
[AVX512-VL]: https://en.wikipedia.org/wiki/AVX-512
[AVX512-VNNI]: https://en.wikipedia.org/wiki/AVX-512#VNNI
[AVX512-VP2INTERSECT]: https://en.wikipedia.org/wiki/AVX-512#VP2INTERSECT
[AVX512-VPOPCNTDQ]:https://en.wikipedia.org/wiki/AVX-512#VPOPCNTDQ_and_BITALG
[AVX-IFMA]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[AVX-NE-CONVERT]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[AVX-VNNI]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[AVX-VNNI-INT16]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[AVX-VNNI-INT8]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[BMI1]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets
[BMI2]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets#BMI2
[`cmpxchg16b`]: https://www.felixcloutier.com/x86/cmpxchg8b:cmpxchg16b
[F16C]: https://en.wikipedia.org/wiki/F16C
[FMA3]: https://en.wikipedia.org/wiki/FMA_instruction_set
[`fxsave`]: https://www.felixcloutier.com/x86/fxsave
[`fxrstor`]: https://www.felixcloutier.com/x86/fxrstor
[GFNI]: https://en.wikipedia.org/wiki/AVX-512#GFNI
[KEYLOCKER]: https://en.wikipedia.org/wiki/List_of_x86_cryptographic_instructions#Intel_Key_Locker_instructions
[KEYLOCKER_WIDE]: https://en.wikipedia.org/wiki/List_of_x86_cryptographic_instructions#Intel_Key_Locker_instructions
[`lzcnt`]: https://www.felixcloutier.com/x86/lzcnt
[`movbe`]: https://www.felixcloutier.com/x86/movbe
[`pclmulqdq`]: https://www.felixcloutier.com/x86/pclmulqdq
[`popcnt`]: https://www.felixcloutier.com/x86/popcnt
[`rdrand`]: https://en.wikipedia.org/wiki/RdRand
[`rdseed`]: https://en.wikipedia.org/wiki/RdRand
[SHA]: https://en.wikipedia.org/wiki/Intel_SHA_extensions
[SHA512]: https://en.wikipedia.org/wiki/Intel_SHA_extensions
[SM3]: https://en.wikipedia.org/wiki/List_of_x86_cryptographic_instructions#Intel_SHA_and_SM3_instructions
[SM4]: https://en.wikipedia.org/wiki/List_of_x86_cryptographic_instructions#Intel_SHA_and_SM3_instructions
[SSE]: https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions
[SSE2]: https://en.wikipedia.org/wiki/SSE2
[SSE3]: https://en.wikipedia.org/wiki/SSE3
[SSE4.1]: https://en.wikipedia.org/wiki/SSE4#SSE4.1
[SSE4.2]: https://en.wikipedia.org/wiki/SSE4#SSE4.2
[SSE4a]: https://en.wikipedia.org/wiki/SSE4#SSE4a
[SSSE3]: https://en.wikipedia.org/wiki/SSSE3
[TBM]: https://en.wikipedia.org/wiki/X86_Bit_manipulation_instruction_set#TBM_(Trailing_Bit_Manipulation)
[VAES]: https://en.wikipedia.org/wiki/AVX-512#VAES
[VPCLMULQDQ]: https://en.wikipedia.org/wiki/AVX-512#VPCLMULQDQ
[`xsave`]: https://www.felixcloutier.com/x86/xsave
[`xsavec`]: https://www.felixcloutier.com/x86/xsavec
[`xsaveopt`]: https://www.felixcloutier.com/x86/xsaveopt
[`xsaves`]: https://www.felixcloutier.com/x86/xsaves

r[attributes.codegen.target_feature.aarch64]
#### `aarch64`

在此平台上使用 `#[target_feature]` 函数遵循[上述限制][attributes.codegen.target_feature.safety-restrictions]。

有关这些特性的更多文档可以在 [ARM 架构参考手册][ARM Architecture Reference Manual] 或 [developer.arm.com] 上找到。

[ARM Architecture Reference Manual]: https://developer.arm.com/documentation/ddi0487/latest
[developer.arm.com]: https://developer.arm.com

> [!NOTE]
> 以下特性对应如果使用，应同时标记为启用或禁用：
> - `paca` 和 `pacg`，LLVM 目前将它们实现为一个特性。

特性        | 隐式启用 | 特性名称
-------        | ------------------ | ------------
`aes`          | `neon`             | FEAT_AES & FEAT_PMULL --- 高级 <abbr title="Single Instruction Multiple Data">SIMD</abbr> AES & PMULL 指令
`bf16`         |                    | FEAT_BF16 --- BFloat16 指令
`bti`          |                    | FEAT_BTI --- 分支目标识别
`crc`          |                    | FEAT_CRC --- CRC32 校验和指令
`dit`          |                    | FEAT_DIT  --- 数据独立时序指令
`dotprod`      | `neon`             | FEAT_DotProd --- 高级 SIMD Int8 点积指令
`dpb`          |                    | FEAT_DPB --- 数据缓存清理到持久化点
`dpb2`         | `dpb`              | FEAT_DPB2 --- 数据缓存清理到深度持久化点
`f32mm`        | `sve`              | FEAT_F32MM --- SVE 单精度 FP 矩阵乘法指令
`f64mm`        | `sve`              | FEAT_F64MM --- SVE 双精度 FP 矩阵乘法指令
`fcma`         | `neon`             | FEAT_FCMA --- 浮点复数支持
`fhm`          | `fp16`             | FEAT_FHM --- 半精度 FP FMLAL 指令
`flagm`        |                    | FEAT_FLAGM --- 条件标志操作
`fp16`         | `neon`             | FEAT_FP16 --- 半精度 FP 数据处理
`frintts`      |                    | FEAT_FRINTTS --- 浮点到整数辅助指令
`i8mm`         |                    | FEAT_I8MM --- Int8 矩阵乘法
`jsconv`       | `neon`             | FEAT_JSCVT --- JavaScript 转换指令
`lor`          |                    | FEAT_LOR --- 有限排序区域扩展
`lse`          |                    | FEAT_LSE --- 大型系统扩展
`mte`          |                    | FEAT_MTE & FEAT_MTE2 --- 内存标记扩展
`neon`         |                    | FEAT_AdvSimd & FEAT_FP --- 浮点和高级 SIMD 扩展
`paca`         |                    | FEAT_PAUTH --- 指针认证（地址认证）
`pacg`         |                    | FEAT_PAUTH --- 指针认证（通用认证）
`pan`          |                    | FEAT_PAN --- 特权访问禁止扩展
`pmuv3`        |                    | FEAT_PMUv3 --- 性能监视器扩展（v3）
`rand`         |                    | FEAT_RNG --- 随机数生成器
`ras`          |                    | FEAT_RAS & FEAT_RASv1p1 --- 可靠性、可用性和可维护性扩展
`rcpc`         |                    | FEAT_LRCPC --- 释放一致处理器一致
`rcpc2`        | `rcpc`             | FEAT_LRCPC2 --- 带立即偏移的 RcPc
`rdm`          | `neon`             | FEAT_RDM --- 舍入双倍乘累加
`sb`           |                    | FEAT_SB --- 推测屏障
`sha2`         | `neon`             | FEAT_SHA1 & FEAT_SHA256 --- 高级 SIMD SHA 指令
`sha3`         | `sha2`             | FEAT_SHA512 & FEAT_SHA3 --- 高级 SIMD SHA 指令
`sm4`          | `neon`             | FEAT_SM3 & FEAT_SM4 --- 高级 SIMD SM3/4 指令
`spe`          |                    | FEAT_SPE --- 统计性能分析扩展
`ssbs`         |                    | FEAT_SSBS & FEAT_SSBS2 --- 推测存储绕过安全
`sve`          | `neon`             | FEAT_SVE --- 可扩展向量扩展
`sve2`         | `sve`              | FEAT_SVE2 --- 可扩展向量扩展 2
`sve2-aes`     | `sve2`, `aes`      | FEAT_SVE_AES & FEAT_SVE_PMULL128 --- SVE AES 指令
`sve2-bitperm` | `sve2`             | FEAT_SVE2_BitPerm --- SVE 位排列
`sve2-sha3`    | `sve2`, `sha3`     | FEAT_SVE2_SHA3 --- SVE SHA3 指令
`sve2-sm4`     | `sve2`, `sm4`      | FEAT_SVE2_SM4 --- SVE SM4 指令
`tme`          |                    | FEAT_TME --- 事务内存扩展
`vh`           |                    | FEAT_VHE --- 虚拟化主机扩展

<!-- Continue similarly for loongarch, riscv, wasm, s390x tables... -->

r[attributes.codegen.target_feature.loongarch]
#### `loongarch`

在此平台上使用 `#[target_feature]` 函数遵循[上述限制][attributes.codegen.target_feature.safety-restrictions]。

特性     | 隐式启用  | 描述
------------|---------------------|-------------------
`f`         |                     | [F][la-f] --- 单精度浮点指令
`d`         | `f`                 | [D][la-d] --- 双精度浮点指令
`frecipe`   |                     | [FRECIPE][la-frecipe] --- 倒数近似指令
`lasx`      | `lsx`               | [LASX][la-lasx] --- 256 位向量指令
`lbt`       |                     | [LBT][la-lbt] --- 二进制翻译指令
`lsx`       | `d`                 | [LSX][la-lsx] --- 128 位向量指令
`lvz`       |                     | [LVZ][la-lvz] --- 虚拟化指令
`div32`     |                     | [DIV32][la-div32] --- 接受非符号扩展 32 位操作数的除法指令
`lam-bh`    |                     | [LAM-BH][la-lam-bh] --- 字节和半字的原子交换和加法指令
`lamcas`    |                     | [LAMCAS][la-lamcas] --- 字节、半字、字和双字的原子比较并交换指令
`ld-seq-sa` |                     | [LD-SEQ-SA][la-ld-seq-sa] --- 对同一地址的加载操作的顺序排序
`scq`       |                     | [SCQ][la-scq] --- 存储条件四字指令

<!-- Keep links near each table to make it easier to move and update. -->

[la-f]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-fp_sp
[la-d]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-fp_dp
[la-frecipe]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-frecipe
[la-lasx]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lasx
[la-lbt]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lbt_x86
[la-lsx]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lsx
[la-lvz]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lvz
[la-div32]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-div32
[la-lam-bh]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lam_bh
[la-lamcas]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lamcas
[la-ld-seq-sa]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-ld_seq_sa
[la-scq]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-scq

r[attributes.codegen.target_feature.riscv]
#### `riscv32` 或 `riscv64`

在此平台上使用 `#[target_feature]` 函数遵循[上述限制][attributes.codegen.target_feature.safety-restrictions]。

rest of RISC-V table kept same as English source

r[attributes.codegen.target_feature.wasm]
#### `wasm32` 或 `wasm64`

在 Wasm 平台上，安全的 `#[target_feature]` 函数始终可以在安全上下文中使用。无法通过 `#[target_feature]` 属性导致未定义行为，因为尝试使用 Wasm 引擎不支持的指令将在加载时失败，而不会有被以不同于编译器预期的方式解释的风险。

rest of wasm table kept same as English source

r[attributes.codegen.target_feature.s390x]
#### `s390x`

在 `s390x` 目标上，使用 `#[target_feature]` 属性的函数遵循[上述限制][attributes.codegen.target_feature.safety-restrictions]。

rest of s390x table kept same as English source

r[attributes.codegen.target_feature.info]
### 附加信息

r[attributes.codegen.target_feature.remark-cfg]
请参见 [`target_feature` 条件编译选项][`target_feature` conditional compilation option] 以根据编译时设置选择性地启用或禁用代码编译。请注意，此选项不受 `target_feature` 属性的影响，仅由对整个 crate 启用的特性驱动。

r[attributes.codegen.target_feature.remark-rt]
可以使用标准库中的平台特定宏在运行时检查特性是否启用，例如 [`is_x86_feature_detected`] 或 [`is_aarch64_feature_detected`]。

> [!NOTE]
> `rustc` 为每个目标和 CPU 设置了一组默认启用的特性。CPU 可以通过 [`-C target-cpu`] 标志选择。可以通过 [`-C target-feature`] 标志为整个 crate 启用或禁用各个特性。

r[attributes.codegen.track_caller]
## `track_caller` 属性

r[attributes.codegen.track_caller.allowed-positions]
`track_caller` 属性可以应用于任何具有 [`"Rust"` ABI][rust-abi] 的函数，但入口点 `fn main` 除外。

r[attributes.codegen.track_caller.traits]
当应用于 trait 声明中的函数和方法时，该属性适用于所有实现。如果 trait 提供了具有该属性的默认实现，则该属性也适用于覆盖实现。

r[attributes.codegen.track_caller.extern]
当应用于 `extern` 块中的函数时，该属性也必须应用于任何链接的实现，否则会导致未定义行为。当应用于对 `extern` 块可用的函数时，`extern` 块中的声明也必须具有该属性，否则会导致未定义行为。

r[attributes.codegen.track_caller.behavior]
### 行为

将属性应用于函数 `f` 允许 `f` 内的代码获取导致 `f` 被调用的"最顶层"跟踪调用的 [`Location`] 的提示。在观察点，实现的行为如同从 `f` 的帧向上遍历栈以查找最近的*未标注*函数 `outer` 的帧，并返回 `outer` 中跟踪调用的 [`Location`]。

```rust
#[track_caller]
fn f() {
    println!("{}", std::panic::Location::caller());
}
```

> [!NOTE]
> `core` 提供 [`core::panic::Location::caller`] 用于观察调用者位置。它包装了由 `rustc` 实现的 [`core::intrinsics::caller_location`] 内部函数。

> [!NOTE]
> 因为生成的 `Location` 是一个提示，实现可以提前停止向上遍历栈。请参见[限制](#限制)了解重要注意事项。

#### 示例

当 `f` 被 `calls_f` 直接调用时，`f` 中的代码观察到其在 `calls_f` 中的调用点。

当 `f` 被另一个带属性的函数 `g` 调用，而 `g` 又被 `calls_g` 调用时，`f` 和 `g` 中的代码观察到 `g` 在 `calls_g` 中的调用点。

r[attributes.codegen.track_caller.limits]
### 限制

r[attributes.codegen.track_caller.hint]
此信息是一个提示，实现不需要保留它。

r[attributes.codegen.track_caller.decay]
特别地，将带有 `#[track_caller]` 的函数强制转换为函数指针会创建一个垫片，观察者看来它是在带属性的函数定义点被调用的，从而在虚调用中丢失实际的调用者信息。一个常见的强制转换示例是创建方法带有属性的 trait 对象。

> [!NOTE]
> 函数指针的上述垫片是必需的，因为 `rustc` 通过将隐式参数附加到函数 ABI 在代码生成上下文中实现 `track_caller`，但这对于间接调用将是不健全的，因为该参数不是函数类型的一部分，并且给定的函数指针类型可能引用或不引用具有该属性的函数。创建垫片对函数指针的调用者隐藏了隐式参数，保持了健全性。

<!-- template:attributes -->
r[attributes.codegen.instruction_set]
## `instruction_set` 属性

r[attributes.codegen.instruction_set.intro]
*`instruction_set` [属性][attribute]* 指定函数在代码生成期间将使用的指令集。这允许在单个程序中混合多个指令集。

> [!EXAMPLE]
> <!-- ignore: arm-only -->
> ```rust,ignore
> #[instruction_set(arm::a32)]
> fn arm_code() {}
>
> #[instruction_set(arm::t32)]
> fn thumb_code() {}
> ```

r[attributes.codegen.instruction_set.syntax]
`instruction_set` 属性使用 [MetaListPaths] 语法来指定由架构族名称和指令集名称组成的单个路径。

r[attributes.codegen.instruction_set.allowed-positions]
`instruction_set` 属性只能应用于具有[函数体][bodies]的函数。

r[attributes.codegen.instruction_set.duplicates]
`instruction_set` 属性在一个函数上只能使用一次。

r[attributes.codegen.instruction_set.target-limits]
`instruction_set` 属性只能用于支持给定值的目标。

r[attributes.codegen.instruction_set.inline-asm]
当使用 `instruction_set` 属性时，函数中的任何内联汇编必须使用指定的指令集而不是目标默认的指令集。

r[attributes.codegen.instruction_set.arm]
### ARM 上的 `instruction_set`

当面向 `ARMv4T` 和 `ARMv5te` 架构时，`instruction_set` 的支持值为：

- `arm::a32` --- 将函数生成为 A32 "ARM" 代码。
- `arm::t32` --- 将函数生成为 T32 "Thumb" 代码。

如果函数的地址被取为函数指针，地址的低位将取决于所选的指令集：

- 对于 `arm::a32`（"ARM"），将为 0。
- 对于 `arm::t32`（"Thumb"），将为 1。

[`-C target-cpu`]: ../../rustc/codegen-options/index.html#target-cpu
[`-C target-feature`]: ../../rustc/codegen-options/index.html#target-feature
[`export_name`]: abi.export_name
[`is_aarch64_feature_detected`]: ../../std/arch/macro.is_aarch64_feature_detected.html
[`is_x86_feature_detected`]: ../../std/arch/macro.is_x86_feature_detected.html
[`Location`]: core::panic::Location
[`naked_asm!`]: ../inline-assembly.md
[`no_mangle`]: abi.no_mangle
[`target_feature` conditional compilation option]: ../conditional-compilation.md#target_feature
[`unused_variables`]: ../../rustc/lints/listing/warn-by-default.html#unused-variables
[associated functions]: items.associated.fn
[async blocks]: expr.block.async
[async closure]: expr.closure.async
[async function]: items.fn.async
[attribute]: ../attributes.md
[attributes]: ../attributes.md
[bodies]: items.fn.body
[closures]: expr.closure
[default definition]: items.traits.associated-item-decls
[free functions]: items.fn
[function body]: ../items/functions.md#function-body
[functions]: ../items/functions.md
[inherent impl]: items.impl.inherent
[rust-abi]: ../items/external-blocks.md#abi
[target architecture]: ../conditional-compilation.md#target_arch
[trait]: items.traits
[trait definition]: items.traits
[trait impl]: items.impl.trait
[undefined behavior]: ../behavior-considered-undefined.md
[unsafe attribute]: ../attributes.md#r-attributes.safety
