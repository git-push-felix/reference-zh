# 属性

属性应使用以下模板。示例给出了你*应该*使用的措辞，但如果属性不符合任何示例或者示例妨碍了清晰性，你可以偏离模板。

当添加一个属性（或语法中新的属性位置）时，请确保更新所有"在...上的属性"小节，这些小节列出了可以在各种位置上使用的属性。

----

<!-- template:attributes -->
r[PARENT.example]
## `example` 属性

r[PARENT.example.intro]
*`example` [属性][attributes]* ...给出高层次的描述。

> [!EXAMPLE]
> ```rust
> // 这应该是一个非常基本的示例，展示该属性
> // 以某种方式被使用。
> #[example]
> fn some_meaningful_name() {}
> ```

r[PARENT.example.syntax]
描述此属性接受的语法。你可以说明它使用某种预先存在的语法，如 `MetaWord`，或者定义一个显式的语法。如果有不同的形式，这里简要描述语法并链接到下面解释不同形式行为的相应规则。示例：

----

`example` 属性使用 [MetaWord] 语法。

----

`example` 属性使用 [MetaListPaths] 语法来指定一个...的列表。

----

`example` 属性使用 [MetaWord] 和 [MetaNameValueStr] 语法。

----

`example` 属性使用 [MetaWord]、[MetaListPaths] 和 [MetaNameValueStr] 语法。

----

`example` 属性使用 [MetaNameValueStr] 语法。接受的值是 `"X"` 和 `"Y"`。

----

`example` 属性使用 [MetaNameValueStr] 语法。字符串中的值必须是...

----

`example` 属性有以下形式：

- [MetaWord]
  > [!EXAMPLE]
  > ```rust
  > #[example]
  > fn f() {}
  > ```

- [MetaNameValueStr] --- 给定的字符串必须...
  > [!EXAMPLE]
  > ```rust
  > #[example = "example"]
  > fn f() {}
  > ```

- [MetaListNameValueStr] --- 与 [MetaNameValueStr] 语法相同，给定的字符串必须...
  > [!EXAMPLE]
  > ```rust
  > #[example(inner = "example")]
  > fn f() {}
  > ```

----

`example` 属性的语法是：

```grammar,attributes
@root ExampleAttribute -> `example` `(` ... `)`
```
----

r[PARENT.example.syntax.foo]
`example` 属性的 [MetaNameValueStr] 形式提供了一种指定 foo 的方式。

> [!EXAMPLE]
> ```rust
> #[example = "example"]
> fn some_meaningful_name() {}
> ```

r[PARENT.example.allowed-positions]
解释可以使用此属性的有效位置。

请参阅编译器中的 [`check_attr`](https://github.com/rust-lang/rust/blob/HEAD/compiler/rustc_passes/src/check_attr.rs) 和 [`builtin_attrs.rs`](https://github.com/rust-lang/rust/blob/HEAD/compiler/rustc_feature/src/builtin_attrs.rs)。别忘了某些属性只能作为内部属性或外部属性使用。示例：

----

`example` 属性只能应用于...

----

`example` 属性只能应用于 crate 根。

----

`example` 属性可以在允许使用属性的任何地方使用。

----

如果有未使用属性的警告，或者如果 `rustc` 错误地接受某些位置，请包含关于这些情况的说明。

> [!NOTE]
> `rustc` 在其他位置使用时忽略但会发出 lint 警告。这在未来可能变为错误。

----

r[PARENT.example.duplicates]
解释当属性在同一个元素上多次使用时的行为。请参阅编译器中的 [`AttributeDuplicates`](https://github.com/rust-lang/rust/blob/40d2563ea200f9327a8cb8b99a0fb82f75a7365c/compiler/rustc_feature/src/builtin_attrs.rs#L143)。示例：

----

`example` 属性可以在一个形式上使用任意次数。

----

在一个形式上多次使用 `example` 与使用一次效果相同。

----

`example` 属性只能在...上使用一次。

----

只有首次在程序项上使用 `example` 才有效。

> [!NOTE]
> `rustc` 对首次使用之后的任何使用都会发出 lint 警告。这在未来可能变为错误。

> [!NOTE]
> `rustc` 对首次使用之后的任何使用都会发出向前兼容性 lint 警告。这在未来可能变为错误。

----

只有最后一次在程序项上使用 `example` 才有效。

> [!NOTE]
> `rustc` 对最后一次使用之前的任何使用都会发出 lint 警告。这在未来可能变为错误。

----

只有最后一次在程序项上使用 `example` 才用于...

----

如果 `example` 属性在程序项上使用超过一次，则所有列出值的组合用于...解释它们如何合并。

----

r[PARENT.example.ATTR_NAME]
如果此属性不能与另一个属性一起使用，请指明每个冲突的属性。对两个属性都要这样做。示例：

----

`example` 属性不能与 [`foo`] 属性一起使用。

----

r[PARENT.example.unsafe]
如果这是一个 `unsafe` 属性，请解释其必须满足的安全条件。否则不要包含此小节。当添加新的 unsafe 属性时，请确保也更新 `attributes.safety`。示例：

----

`example` 属性必须标记为 [`unsafe`][attributes.safety]，因为...

----

r[PARENT.example.stdlib]
此规则说明该属性是否在标准库中导出。如果没有，请跳过此小节。示例：

----

`example` 属性在标准库 prelude 中导出为 [`core::prelude::v1::example`]。

----

r[PARENT.example.foo]
从这里开始，添加解释该属性所有行为的规则。如果属性非常简单，你可以只有一个名为 `.behavior` 的规则来解释其行为。更复杂的属性（如有多种输入类型或不同模式的属性）应将每一种作为一个单独的规则来描述。
