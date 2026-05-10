# 语法索引

本附录提供了 token 和常见形式的索引，带有指向定义这些元素位置的链接。

## 关键字

| 关键字       | 使用 |
|---------------|-----|
| `_`           | [通配符模式][wildcard pattern]、[推断常量][inferred const]、[推断类型][inferred type]、[占位生命周期][placeholder lifetime]、[常量项][constant items]、[extern crate][extern crate]、[use 声明][use declarations]、[解构赋值][destructuring assignment] |
| `abstract`    | [保留关键字][reserved keyword] |
| `as`          | [extern crate][items.extern-crate.as]、[use 声明][items.use.forms.as]、[类型强制转换表达式][type cast expressions]、[限定路径][qualified paths] |
| `async`       | [async 函数][async functions]、[async 块][async blocks]、[async 闭包][async closures] |
| `await`       | [await 表达式][await expressions] |
| `become`      | [保留关键字][reserved keyword] |
| `box`         | [保留关键字][reserved keyword] |
| `break`       | [break 表达式][break expressions] |
| `const`       | [const 函数][const functions]、[const 项][const items]、[const 泛型][const generics]、[const 块][const blocks]、[原始借用运算符][raw borrow operator]、[原始指针类型][raw pointer type]、[const 汇编操作数][const assembly operands] |
| `continue`    | [continue 表达式][continue expressions] |
| `crate`       | [extern crate][extern crate]、[可见性][visibility]、[路径][paths] |
| `do`          | [保留关键字][reserved keyword] |
| `dyn`         | [trait 对象][trait objects] |
| `else`        | [let 语句][let statements]、[if 表达式][if expressions] |
| `enum`        | [枚举][enumerations] |
| `extern`      | [extern crate][extern crate]、[extern 函数限定符][extern function qualifier]、[外部块][external blocks]、[extern 函数指针类型][extern function pointer types] |
| `false`       | [布尔类型][boolean type]、[布尔表达式][boolean expressions]、[配置谓词][configuration predicates] |
| `final`       | [保留关键字][reserved keyword] |
| `fn`          | [函数][functions]、[函数指针类型][function pointer types] |
| `for`         | [trait 实现][trait implementations]、[迭代器循环][iterator loops]、[高阶 trait 约束][higher-ranked trait bounds] |
| `gen`         | [保留关键字][reserved keyword] |
| `if`          | [if 表达式][if expressions]、[match 守卫][match guards] |
| `impl`        | [固有 impl][inherent impls]、[trait impl][trait impls]、[impl trait 类型][impl trait types]、[匿名类型参数][anonymous type parameters] |
| `in`          | [可见性][visibility]、[迭代器循环][iterator loops]、[汇编操作数][assembly operands] |
| `let`         | [let 语句][let statements]、[`if let` 模式][`if let` patterns] |
| `loop`        | [无限循环][infinite loops] |
| `macro_rules` | [声明宏][macros by example] |
| `macro`       | [保留关键字][reserved keyword] |
| `match`       | [match 表达式][match expressions] |
| `mod`         | [模块][modules] |
| `move`        | [闭包表达式][closure expressions]、[async 块][async blocks] |
| `mut`         | [借用表达式][borrow expressions]、[标识符模式][identifier patterns]、[引用模式][reference patterns]、[结构体模式][struct patterns]、[引用类型][reference types]、[原始指针类型][raw pointer types]、[self 参数][self parameters]、[静态项][static items] |
| `override`    | [保留关键字][reserved keyword] |
| `priv`        | [保留关键字][reserved keyword] |
| `pub`         | [可见性][visibility] |
| `raw`         | [借用表达式][borrow expressions]、[原始汇编][raw assembly] |
| `ref`         | [标识符模式][identifier patterns]、[结构体模式][struct patterns] |
| `return`      | [return 表达式][return expressions] |
| `safe`        | [外部块函数][external block functions]、[外部块静态项][external block statics] |
| `self`        | [extern crate][items.extern-crate.self]、[self 参数][self parameters]、[可见性][visibility]、[`self` 路径][`self` paths] |
| `Self`        | [`Self` 类型路径][`Self` type paths]、[use 约束][use bounds] |
| `static`      | [静态项][static items]、[`'static` 生命周期][`'static` lifetimes] |
| `struct`      | [结构体][structs] |
| `super`       | [super 路径][super paths]、[可见性][visibility] |
| `trait`       | [trait 项][trait items] |
| `true`        | [布尔类型][boolean type]、[布尔表达式][boolean expressions]、[配置谓词][configuration predicates] |
| `try`         | [保留关键字][reserved keyword] |
| `type`        | [类型别名][type aliases] |
| `typeof`      | [保留关键字][reserved keyword] |
| `union`       | [联合体项][union items] |
| `unsafe`      | [unsafe 块][unsafe blocks]、[unsafe 属性][unsafe attributes]、[unsafe 模块][unsafe modules]、[unsafe 函数][unsafe functions]、[unsafe 外部块][unsafe external blocks]、[unsafe 外部函数][unsafe external functions]、[unsafe 外部静态项][unsafe external statics]、[unsafe trait][unsafe traits]、[unsafe trait 实现][unsafe trait implementations] |
| `unsized`     | [保留关键字][reserved keyword] |
| `use`         | [use 项][use items]、[use 约束][use bounds] |
| `virtual`     | [保留关键字][reserved keyword] |
| `where`       | [where 子句][where clauses] |
| `while`       | [谓词循环][predicate loops] |
| `yield`       | [保留关键字][reserved keyword] |

## 运算符和标点符号

| 符号 | 名称        | 使用 |
|--------|-------------|-----|
| `+`    | 加号        | [加法][arith]、[trait 约束][trait bounds]、[宏 Kleene 匹配器][macro Kleene matcher] |
| `-`    | 减号       | [减法][arith]、[取反][negation] |
| `*`    | 星号        | [乘法][arith]、[解引用][dereference]、[原始指针][raw pointers]、[宏 Kleene 匹配器][macro Kleene matcher]、[glob 导入][glob imports] |
| `/`    | 斜线       | [除法][arith] |
| `%`    | 百分号     | [取余][arith] |
| `^`    | 脱字符       | [按位与逻辑 XOR][arith] |
| `!`    | 非         | [按位与逻辑 NOT][negation]、[宏调用][macro calls]、[内部属性][inner attribute]、[never 类型][never type]、[否定 impl][negative impls] |
| `&`    | 与         | [按位与逻辑 AND][arith]、[借用][borrow]、[引用][references]、[引用模式][reference patterns] |
| `\|`   | 或 | [按位与逻辑 OR][arith]、[闭包][closures]、[或模式][or patterns]、[if let][if let]、[while let][while let] |
| `&&`   | 与与      | [惰性 AND][lazy-bool]、[借用][borrow]、[引用][references]、[引用模式][reference patterns] |
| `\|\|` | 或或 | [惰性 OR][lazy-bool]、[闭包][closures] |
| `<<`   | 左移         | [左移][arith]、[嵌套泛型][generics] |
| `>>`   | 右移         | [右移][arith]、[嵌套泛型][generics] |
| `+=`   | 加等      | [加法赋值][compound] |
| `-=`   | 减等     | [减法赋值][compound] |
| `*=`   | 星等      | [乘法赋值][compound] |
| `/=`   | 斜等     | [除法赋值][compound] |
| `%=`   | 百分等   | [取余赋值][compound] |
| `^=`   | 脱字符等     | [按位 XOR 赋值][compound] |
| `&=`   | 与等       | [按位 AND 赋值][compound] |
| `\|=`  | 或等 | [按位 OR 赋值][compound] |
| `<<=`  | 左移等       | [左移赋值][compound] |
| `>>=`  | 右移等       | [右移赋值][compound]、[嵌套泛型][generics] |
| `=`    | 等          | [赋值][assignment]、[let 语句][let statements]、[属性][attributes]、各种类型定义 |
| `==`   | 等等        | [相等][comparison] |
| `!=`   | 不等          | [不等于][comparison] |
| `>`    | 大于          | [大于][comparison]、[泛型][generics]、[路径][paths]、[use 约束][use bounds] |
| `<`    | 小于          | [小于][comparison]、[泛型][generics]、[路径][paths]、[use 约束][use bounds] |
| `>=`   | 大于等于          | [大于或等于][comparison]、[泛型][generics] |
| `<=`   | 小于等于          | [小于或等于][comparison] |
| `@`    | At          | [子模式绑定][subpattern binding] |
| `.`    | 点         | [字段访问][field]、[元组索引][tuple index]、[方法调用][method calls]、[await 表达式][await expressions] |
| `..`   | 点点      | [范围表达式][expr.range]、[结构体表达式][struct expressions]、[rest 模式][rest pattern]、[范围模式][range patterns]、[结构体模式][struct patterns] |
| `...`  | 点点点   | [可变参数函数][variadic functions]、[范围模式][range patterns] |
| `..=`  | 点点等    | [包含范围表达式][expr.range]、[范围模式][range patterns] |
| `,`    | 逗号       | 各种分隔符 |
| `;`    | 分号        | 各种项和语句的终止符、[数组表达式][array expressions]、[数组类型][array types] |
| `:`    | 冒号       | 各种分隔符 |
| `::`   | 路径分隔符     | [路径分隔符][paths] |
| `->`   | 右箭头      | [函数][functions]、[闭包][closures]、[函数指针类型][function pointer type] |
| `=>`   | 胖箭头    | [match 分支][match]、[宏][macros] |
| `<-`   | 左箭头      | 左箭头符号自 Rust 1.0 之前就一直未使用，但仍被视为单个 token。 |
| `#`    | 井号       | [属性][attributes]、[原始字符串字面量][raw string literals]、[原始字节字符串字面量][raw byte string literals]、[原始 C 字符串字面量][raw C string literals] |
| `$`    | 美元      | [宏][macros] |
| `?`    | 问号    | [try 传播表达式][question]、[放宽 trait 约束][relaxed trait bounds]、[宏 Kleene 匹配器][macro Kleene matcher] |
| `~`    | 波浪号       | 波浪号运算符自 Rust 1.0 之前就一直未使用，但其 token 仍可使用。 |

## 注释

| 注释  | 使用 |
|----------|-----|
| `//`     | [行注释][comments] |
| `//!`    | [内部行注释][comments] |
| `///`    | [外部行文档注释][comments] |
| `/*…*/`  | [块注释][comments] |
| `/*!…*/` | [内部块文档注释][comments] |
| `/**…*/` | [外部块文档注释][comments] |

## 其他 token

| Token        | 使用 |
|--------------|-----|
| `ident`      | [标识符][identifiers] |
| `r#ident`    | [原始标识符][raw identifiers] |
| `'ident`     | [生命周期和循环标签][lifetimes and loop labels] |
| `'r#ident`   | [原始生命周期和循环标签][raw lifetimes and loop labels] |
| `…u8`、`…i32`、`…f64`、`…usize`、… | [数字字面量][number literals] |
| `"…"`        | [字符串字面量][string literals] |
| `r"…"`、`r#"…"#`、`r##"…"##`、… | [原始字符串字面量][raw string literals] |
| `b"…"`       | [字节字符串字面量][byte string literals] |
| `br"…"`、`br#"…"#`、`br##"…"##`、… | [原始字节字符串字面量][raw byte string literals] |
| `'…'`        | [字符字面量][character literals] |
| `b'…'`       | [字节字面量][byte literals] |
| `c"…"`       | [C 字符串字面量][C string literals] |
| `cr"…"`、`cr#"…"#`、`cr##"…"##`、… | [原始 C 字符串字面量][raw C string literals] |

## 宏

| 语法                                     | 使用 |
|--------------------------------------------|-----|
| `ident!(…)`<br>`ident! {…}`<br>`ident![…]` | [宏调用][macro invocations] |
| `$ident`                                   | [宏元变量][macro metavariable] |
| `$ident:kind`                              | [宏匹配器片段指示符][macro matcher fragment specifier] |
| `$(…)…`                                    | [宏重复][macro repetition] |

## 属性

| 语法     | 使用 |
|------------|-----|
| `#[meta]`  | [外部属性][outer attribute] |
| `#![meta]` | [内部属性][inner attribute] |

## 表达式

| 表达式                | 使用 |
|---------------------------|-----|
| `\|…\| expr`<br>`\|…\| -> Type { … }` | [闭包][closures] |
| `ident::…`                | [路径][paths] |
| `::crate_name::…`         | [显式 crate 路径][explicit crate paths] |
| `crate::…`                | [crate 相对路径][crate-relative paths] |
| `self::…`                 | [模块相对路径][module-relative paths] |
| `super::…`                | [父模块路径][parent module paths] |
| `Type::…`<br>`<Type as Trait>::ident` | [关联项][associated items] |
| `<Type>::…`               | [限定路径][qualified paths]，可用于没有名称的类型，如 `<&T>::…`、`<[T]>::…` 等。 |
| `Trait::method(…)`<br>`Type::method(…)`<br>`<Type as Trait>::method(…)` | [消歧方法调用][disambiguated method calls] |
| `method::<…>(…)`<br>`path::<…>` | [泛型参数][generic arguments]，又名 turbofish |
| `()`                      | [单元][unit] |
| `(expr)`                  | [括号表达式][parenthesized expressions] |
| `(expr,)`                 | [单元素元组表达式][single-element tuple expressions] |
| `(expr, …)`               | [元组表达式][tuple expressions] |
| `expr(expr, …)`           | [调用表达式][call expressions] |
| `expr.0`、`expr.1`、…     | [元组索引表达式][tuple indexing expressions] |
| `expr.ident`              | [字段访问表达式][field access expressions] |
| `{…}`                     | [块表达式][block expressions] |
| `Type {…}`                | [结构体表达式][struct expressions] |
| `Type(…)`                 | [元组结构体构造函数][tuple struct constructors] |
| `[…]`                     | [数组表达式][array expressions] |
| `[expr; len]`             | [重复数组表达式][repeat array expressions] |
| `expr[..]`、`expr[a..]`、`expr[..b]`、`expr[a..b]`、`expr[a..=b]`、`expr[..=b]` | [数组和切片索引表达式][array and slice indexing expressions] |
| `if expr {…} else {…}`    | [if 表达式][if expressions] |
| `match expr { pattern => {…} }` | [match 表达式][match expressions] |
| `loop {…}`                | [无限循环表达式][infinite loop expressions] |
| `while expr {…}`          | [谓词循环表达式][predicate loop expressions] |
| `for pattern in expr {…}` | [迭代器循环][iterator loops] |
| `&expr`<br>`&mut expr`    | [借用表达式][borrow expressions] |
| `&raw const expr`<br>`&raw mut expr` | [原始借用表达式][raw borrow expressions] |
| `*expr`                   | [解引用表达式][dereference expressions] |
| `expr?`                   | [try 传播表达式][try propagation expressions] |
| `-expr`                   | [取反表达式][negation expressions] |
| `!expr`                   | [按位与逻辑 NOT 表达式][bitwise and logical NOT expressions] |
| `expr as Type`            | [类型强制转换表达式][type cast expressions] |

## 项

[程序项][Items] 是 crate 的组成部分。

| 项                          | 使用 |
|-------------------------------|-----|
| `mod ident;`<br>`mod ident {…}` | [模块][modules] |
| `use path;`                   | [use 声明][use declarations] |
| `fn ident(…) {…}`             | [函数][functions] |
| `type Type = Type;`           | [类型别名][type aliases] |
| `struct ident {…}`            | [结构体][structs] |
| `enum ident {…}`              | [枚举][enumerations] |
| `union ident {…}`             | [联合体][unions] |
| `trait ident {…}`             | [trait][traits] |
| `impl Type {…}`<br>`impl Type for Trait {…}` | [实现][implementations] |
| `const ident = expr;`         | [常量项][constant items] |
| `static ident = expr;`        | [静态项][static items] |
| `extern "C" {…}`              | [外部块][external blocks] |
| `fn ident<…>(…) …`<br>`struct ident<…> {…}`<br>`enum ident<…> {…}`<br>`impl<…> Type<…> {…}` | [泛型定义][generic definitions] |

## 类型表达式

[类型表达式][Type expressions] 用于引用类型。

| 类型                                  | 使用 |
|---------------------------------------|-----|
| `bool`、`u8`、`f64`、`str`、…         | [原始类型][primitive types] |
| `for<…>`                              | [高阶 trait 约束][higher-ranked trait bounds] |
| `T: TraitA + TraitB`                  | [trait 约束][trait bounds] |
| `T: 'a + 'b`                          | [生命周期约束][lifetime bounds] |
| `T: TraitA + 'a`                      | [trait 和生命周期约束][trait and lifetime bounds] |
| `T: ?Sized`                           | [放宽 trait 约束][relaxed trait bounds] |
| `[Type; len]`                         | [数组类型][array types] |
| `(Type, …)`                           | [元组类型][tuple types] |
| `[Type]`                              | [切片类型][slice types] |
| `(Type)`                              | [括号类型][parenthesized types] |
| `impl Trait`                          | [impl trait 类型][impl trait types]、[匿名类型参数][anonymous type parameters] |
| `dyn Trait`                           | [trait 对象类型][trait object types] |
| `ident`<br>`ident::…`                 | [类型路径][type paths]（可以引用[结构体][structs]、[枚举][enumerations]、[联合体][unions]、[类型别名][type aliases]、[trait][traits]、[泛型][generics]等） |
| `Type<…>`<br>`Trait<…>`               | [泛型参数][generic arguments]（例如 `Vec<u8>`） |
| `Trait<ident = Type>`                 | [关联类型绑定][associated type bindings]（例如 `Iterator<Item = T>`） |
| `Trait<ident: …>`                     | [关联类型约束][associated type bounds]（例如 `Iterator<Item: Send>`） |
| `&Type`<br>`&mut Type`                | [引用类型][reference types] |
| `*mut Type`<br>`*const Type`          | [原始指针类型][raw pointer types] |
| `fn(…) -> Type`                       | [函数指针类型][function pointer types] |
| `_`                                   | [推断类型][inferred type]、[推断常量][inferred const] |
| `'_`                                  | [占位生命周期][placeholder lifetime] |
| `!`                                   | [never 类型][never type] |

## 模式

[模式][Patterns] 用于匹配值。

| 模式                           | 使用 |
|-----------------------------------|-----|
| `"foo"`、`'a'`、`123`、`2.4`、…   | [字面量模式][literal patterns] |
| `ident`                           | [标识符模式][identifier patterns] |
| `_`                               | [通配符模式][wildcard pattern] |
| `..`                              | [rest 模式][rest pattern] |
| `a..`、`..b`、`a..b`、`a..=b`、`..=b` | [范围模式][range patterns] |
| `&pattern`<br>`&mut pattern`      | [引用模式][reference patterns] |
| `path {…}`                        | [结构体模式][struct patterns] |
| `path(…)`                         | [元组结构体模式][tuple struct patterns] |
| `(pattern, …)`                    | [元组模式][tuple patterns] |
| `(pattern)`                       | [分组模式][grouped patterns] |
| `[pattern, …]`                    | [切片模式][slice patterns] |
| `CONST`、`Enum::Variant`、…       | [路径模式][path patterns] |

[`'static` lifetimes]: bound
[`if let` patterns]: expr.if.let
[`self` paths]: paths.qualifiers.mod-self
[`Self` type paths]: paths.qualifiers.type-self
[anonymous type parameters]: type.impl-trait.param
[arith]: expr.arith-logic
[array and slice indexing expressions]: expr.array.index
[array expressions]: expr.array
[array types]: type.array
[assembly operands]: asm.operand-type.supported-operands.in
[assignment]: expr.assign
[associated items]: items.associated
[associated type bindings]: paths.expr
[associated type bounds]: paths.expr
[async blocks]: expr.block.async
[async closures]: expr.closure.async
[async functions]: items.fn.async
[await expressions]: expr.await
[bitwise and logical NOT expressions]: expr.negate
[block expressions]: expr.block
[boolean expressions]: expr.literal
[boolean type]: type.bool
[borrow expressions]: expr.operator.borrow
[borrow]: expr.operator.borrow
[break expressions]: expr.loop.break
[byte literals]: lex.token.byte
[byte string literals]: lex.token.str-byte
[C string literals]: lex.token.str-c
[call expressions]: expr.call
[character literals]: lex.token.literal.char
[closure expressions]: expr.closure
[closures]: expr.closure
[comparison]: expr.cmp
[compound]: expr.compound-assign
[configuration predicates]: cfg
[const assembly operands]: asm.operand-type.supported-operands.const
[const blocks]: expr.block.const
[const functions]: const-eval.const-fn
[const generics]: items.generics.const
[const items]: items.const
[constant items]: items.const
[continue expressions]: expr.loop.continue
[crate-relative paths]: paths.qualifiers.crate
[dereference expressions]: expr.deref
[dereference]: expr.deref
[destructuring assignment]: expr.placeholder
[disambiguated method calls]: expr.call.desugar
[enumerations]: items.enum
[explicit crate paths]: paths.qualifiers.global-root
[extern crate]: items.extern-crate
[extern function pointer types]: type.fn-pointer.qualifiers
[extern function qualifier]: items.fn.extern
[external block functions]: items.extern.fn
[external block statics]: items.extern.static
[external blocks]: items.extern
[field access expressions]: expr.field
[field]: expr.field
[function pointer type]: type.fn-pointer
[function pointer types]: type.fn-pointer
[functions]: items.fn
[generic arguments]: items.generics
[generic definitions]: items.generics
[generics]: items.generics
[glob imports]: items.use.glob
[grouped patterns]: patterns.paren
[higher-ranked trait bounds]: bound.higher-ranked
[identifier patterns]: patterns.ident
[identifiers]: ident
[if expressions]: expr.if
[if let]: expr.if.let
[impl trait types]: type.impl-trait.return
[implementations]: items.impl
[inferred const]: items.generics.const.inferred
[inferred type]: type.inferred
[infinite loop expressions]: expr.loop.infinite
[infinite loops]: expr.loop.infinite
[inherent impls]: items.impl.inherent
[inner attribute]: attributes.inner
[iterator loops]: expr.loop.for
[lazy-bool]: expr.bool-logic
[let statements]: statement.let
[lifetime bounds]: bound.lifetime
[lifetimes and loop labels]: lex.token.life
[literal patterns]: patterns.literal
[macro calls]: macro.invocation
[macro invocations]: macro.invocation
[macro Kleene matcher]: macro.decl.repetition
[macro matcher fragment specifier]: macro.decl.meta.specifier
[macro metavariable]: macro.decl.meta
[macro repetition]: macro.decl.repetition
[macros by example]: macro.decl
[macros]: macro.decl
[match expressions]: expr.match
[match guards]: expr.match.guard
[match]: expr.match
[method calls]: expr.method
[module-relative paths]: paths.qualifiers.mod-self
[modules]: items.mod
[negation expressions]: expr.negate
[negation]: expr.negate
[negative impls]: items.impl
[never type]: type.never
[number literals]: lex.token.literal.num
[or patterns]: patterns.or
[outer attribute]: attributes.outer
[parent module paths]: paths.qualifiers.super
[parenthesized expressions]: expr.paren
[parenthesized types]: type.name.parenthesized
[path patterns]: patterns.path
[placeholder lifetime]: lifetime-elision.function.explicit-placeholder
[predicate loop expressions]: expr.loop.while
[predicate loops]: expr.loop.while
[primitive types]: type.kinds
[qualified paths]: paths.qualified
[question]: expr.try
[range patterns]: patterns.range
[raw assembly]: asm.options.supported-options.raw
[raw borrow expressions]: expr.borrow.raw
[raw borrow operator]: expr.borrow.raw
[raw byte string literals]: lex.token.str-byte-raw
[raw C string literals]: lex.token.str-c-raw
[raw identifiers]: ident.raw
[raw lifetimes and loop labels]: lex.token.life
[raw pointer type]: type.pointer.raw
[raw pointer types]: type.pointer.raw
[raw pointers]: type.pointer.raw
[raw string literals]: lex.token.literal.str-raw
[reference patterns]: patterns.ref
[reference types]: type.pointer.reference
[references]: type.pointer.reference
[relaxed trait bounds]: bound.sized
[repeat array expressions]: expr.array
[reserved keyword]: lex.keywords.reserved
[rest pattern]: patterns.rest
[return expressions]: expr.return
[self parameters]: items.fn.params.self-pat
[single-element tuple expressions]: expr.tuple
[slice patterns]: patterns.slice
[slice types]: type.slice
[static items]: items.static
[string literals]: lex.token.literal.str
[struct expressions]: expr.struct
[struct patterns]: patterns.struct
[structs]: items.struct
[subpattern binding]: patterns.ident.scrutinized
[super paths]: paths.qualifiers.super
[trait and lifetime bounds]: bound
[trait bounds]: bound
[trait implementations]: items.impl.trait
[trait impls]: items.impl.trait
[trait items]: items.traits
[trait object types]: type.trait-object
[trait objects]: type.trait-object
[traits]: items.traits
[try propagation expressions]: expr.try
[tuple expressions]: expr.tuple
[tuple index]: expr.tuple-index
[tuple indexing expressions]: expr.tuple-index
[tuple patterns]: patterns.tuple
[tuple struct constructors]: items.struct.tuple
[tuple struct patterns]: patterns.tuple-struct
[tuple types]: type.tuple
[type aliases]: items.type
[type cast expressions]: expr.as
[Type expressions]: type.name
[type paths]: type.name.path
[union items]: items.union
[unions]: items.union
[unit]: type.tuple.unit
[unsafe attributes]: attributes.safety
[unsafe blocks]: expr.block.unsafe
[unsafe external blocks]: unsafe.extern
[unsafe external functions]: items.extern.fn.safety
[unsafe external statics]: items.extern.static.safety
[unsafe functions]: unsafe.fn
[unsafe modules]: items.mod.unsafe
[unsafe trait implementations]: items.impl.trait.safety
[unsafe traits]: items.traits.safety
[use bounds]: bound.use
[use declarations]: items.use
[use items]: items.use
[variadic functions]: items.extern.variadic
[visibility]: vis
[where clauses]: items.generics.where
[while let]: expr.loop.while.let
[wildcard pattern]: patterns.wildcard
