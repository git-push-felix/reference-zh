r[lex.keywords]
# 关键字

Rust 将关键字分为三类：

* [严格关键字](#strict-keywords)
* [保留关键字](#reserved-keywords)
* [弱关键字](#weak-keywords)

r[lex.keywords.strict]
## 严格关键字 {#strict-keywords}

r[lex.keywords.strict.intro]
这些关键字只能在其正确的上下文中使用。它们不能用作以下名称：

* [程序项][Items]
* [变量][Variables]和函数参数
* 字段和[变体][variants]
* [类型参数][Type parameters]
* 生命周期参数或[循环标签][loop labels]
* [宏][Macros]或[属性][attributes]
* [宏占位符][Macro placeholders]
* [crate][Crates]

r[lex.keywords.strict.list]
以下关键字在所有版本中均存在：

- `_`
- `as`
- `async`
- `await`
- `break`
- `const`
- `continue`
- `crate`
- `dyn`
- `else`
- `enum`
- `extern`
- `false`
- `fn`
- `for`
- `if`
- `impl`
- `in`
- `let`
- `loop`
- `match`
- `mod`
- `move`
- `mut`
- `pub`
- `ref`
- `return`
- `self`
- `Self`
- `static`
- `struct`
- `super`
- `trait`
- `true`
- `type`
- `unsafe`
- `use`
- `where`
- `while`

r[lex.keywords.strict.edition2018]
> [!EDITION-2018]
> 以下关键字在 2018 版本中添加：
>
> - `async`
> - `await`
> - `dyn`

r[lex.keywords.reserved]
## 保留关键字 {#reserved-keywords}

r[lex.keywords.reserved.intro]
这些关键字尚未使用，但为将来使用而保留。它们与严格关键字具有相同的限制。这样做的原因是禁止程序使用这些关键字，从而使当前程序与未来的 Rust 版本保持向前兼容。

r[lex.keywords.reserved.list]
- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

r[lex.keywords.reserved.edition2018]
> [!EDITION-2018]
> `try` 关键字在 2018 版本中作为保留关键字添加。

r[lex.keywords.reserved.edition2024]
> [!EDITION-2024]
> `gen` 关键字在 2024 版本中作为保留关键字添加。

r[lex.keywords.weak]
## 弱关键字 {#weak-keywords}

r[lex.keywords.weak.intro]
这些关键字仅在特定上下文中具有特殊含义。例如，可以用 `union` 这个名称声明变量或方法。

- `'static`
- `macro_rules`
- `raw`
- `safe`
- `union`

r[lex.keywords.weak.macro_rules]
* `macro_rules` 用于创建自定义[宏][macros]。

r[lex.keywords.weak.union]
* `union` 用于声明[联合体][union]，且仅在联合体声明中才是关键字。

r[lex.keywords.weak.lifetime-static]
* `'static` 用于静态生命周期，不能用作[泛型生命周期参数][generic lifetime parameter]或[循环标签][loop label]

  ```compile_fail
  // 错误[E0262]: 无效的生存期参数名称: `'static`
  fn invalid_lifetime_parameter<'static>(s: &'static str) -> &'static str { s }
  ```

r[lex.keywords.weak.safe]
* `safe` 用于函数和静态项，在[外部块][external blocks]中具有含义。

r[lex.keywords.weak.raw]
* `raw` 用于[原始借用运算符][raw borrow operators]，且仅在匹配原始借用运算符形式（如 `&raw const expr` 或 `&raw mut expr`）时才是关键字。

r[lex.keywords.weak.dyn.edition2018]
> [!EDITION-2018]
> 在 2015 版本中，[`dyn`] 是用在类型位置并且后跟不以 `::` 或 `<` 开头的路径、生命周期、问号、`for` 关键字或左括号时的关键字。
>
> 从 2018 版本开始，`dyn` 被提升为严格关键字。

[items]: items.md
[Variables]: variables.md
[Type parameters]: types/parameters.md
[loop labels]: expressions/loop-expr.md#loop-labels
[Macros]: macros.md
[attributes]: attributes.md
[Macro placeholders]: macros-by-example.md
[Crates]: crates-and-source-files.md
[union]: items/unions.md
[variants]: items/enumerations.md
[`dyn`]: types/trait-object.md
[loop label]: expressions/loop-expr.md#loop-labels
[generic lifetime parameter]: items/generics.md
[external blocks]: items/external-blocks.md
[raw borrow operators]: expressions/operator-expr.md#raw-borrow-operators
