r[names.resolution]
# 名称解析

r[names.resolution.intro]
*名称解析*是将路径和其他标识符绑定到这些实体声明的过程。名称被隔离到不同的[命名空间][namespaces]中，允许不同命名空间中的实体共享相同的名称而不会冲突。每个名称在一个[作用域][scope]内有效，即源文本中可以引用该名称的区域。对名称的访问可能受其[可见性][visibility]的限制。

名称解析在整个编译过程中分为三个阶段。第一阶段，*展开时解析*，解析所有 [`use` 声明][`use` declarations]和[宏调用][macro invocations]。第二阶段，*主解析*，解析所有尚未解析且不依赖类型信息来解析的名称。最后一个阶段，*类型相关解析*，在类型信息可用后解析剩余的名称。

> [!NOTE]
> 展开时解析也称为*早期解析*。主解析也称为*晚期解析*。

r[names.resolution.general]
## 概述

r[names.resolution.general.intro]
本节中的规则适用于名称解析的所有阶段。

r[names.resolution.general.scopes]
### 作用域

r[names.resolution.general.scopes.intro]
> [!NOTE]
> 这是一个占位符，用于未来展开关于各种作用域内名称解析的内容。

r[names.resolution.expansion]
## 展开时名称解析

r[names.resolution.expansion.intro]
展开时名称解析是完成宏展开和完全生成 crate 的 [AST] 所必需的名称解析阶段。此阶段需要解析宏调用和 `use` 声明。解析 `use` 声明对于通过[基于路径的作用域][path-based scope]解析的宏调用是必需的。解析宏调用是为了将它们展开。

r[names.resolution.expansion.unresolved-invocations]
在展开时名称解析之后，AST 不得包含任何未展开的宏调用。每个宏调用都解析为存在于最终 AST 或外部 crate 中的有效定义。

```rust,compile_fail
m!(); // 错误：在此作用域中找不到宏 `m`。
```

r[names.resolution.expansion.expansion-order-stability]
名称的解析必须是稳定的。展开后，完全展开的 AST 中的名称必须解析到相同的定义，无论宏展开和导入解析的顺序如何。

r[names.resolution.expansion.speculation]
在宏展开期间选择的所有名称解析候选项都被认为是推测性的。一旦 crate 被完全展开，所有推测性的导入解析都会被验证，以确保宏展开没有引入任何新的歧义。

> [!NOTE]
> 由于宏展开的迭代性质，这会导致所谓的时间旅行歧义，例如当宏或 glob 导入引入了一个与其自身基路径有歧义的项时。
>
> ```rust,compile_fail,E0659
> # fn main() {}
> macro_rules! f {
>     () => {
>         mod m {
>             pub(crate) use f;
>         }
>     }
> }
> f!();
>
> const _: () = {
>     // 最初，我们推测性地将 `m` 解析为 crate 根中的模块。
>     //
>     // `f` 的展开在此函数体内引入了第二个 `m` 模块。
>     //
>     // 展开时解析通过重新解析所有导入和宏调用来最终确定解析结果，
>     // 看到引入的歧义并将其报告为错误。
>     m::f!(); // 错误：`m` 有歧义。
> };
> ```

r[names.resolution.expansion.imports]
### 导入
r[names.resolution.expansion.imports.intro]
所有 `use` 声明在此解析阶段完全解析。[类型相关路径][type-relative paths]在此阶段无法解析，将产生错误。

```rust,no_run
mod m {
    pub const C: () = ();
    pub enum E { V }
    pub type A = E;
    impl E {
        pub const C: () = ();
    }
}

// 在展开时解析的有效导入：
use m::C; // 正确。
use m::E; // 正确。
use m::A; // 正确。
use m::E::V; // 正确。

// 在类型相关解析期间解析的有效表达式：
let _ = m::A::V; // 正确。
let _ = m::E::C; // 正确。
```

```rust,compile_fail,E0432
# mod m {
#     pub const C: () = ();
#     pub enum E { V }
#     pub type A = E;
#     impl E {
#         pub const C: () = ();
#     }
# }
// 无法在展开时解析的无效类型相关导入：
use m::A::V; // 错误：未解析的导入 `m::A::V`。
use m::E::C; // 错误：未解析的导入 `m::E::C`。
```

r[names.resolution.expansion.imports.shadowing]
通过[外部作用域][outer scope]中的 `use` 声明引入的名称会被内部作用域中相同命名空间的同名候选项遮蔽，除非受到[名称解析歧义][name resolution ambiguities]的限制。

```rust,no_run
pub mod m1 {
    pub mod ambig {
        pub const C: u8 = 1;
    }
}

pub mod m2 {
    pub mod ambig {
        pub const C: u8 = 2;
    }
}

// 这在外部作用域中引入了名称 `ambig`。
use m1::ambig;
const _: () = {
    // 这在内部作用域中遮蔽了 `ambig`。
    use m2::ambig;
    // 此处选择内部候选项
    // 作为 `ambig` 的解析结果。
    use ambig::C;
    assert!(C == 2);
};
```

r[names.resolution.expansion.imports.shadowing.shared-scope]
在单个作用域内通过 `use` 声明引入的名称在以下情况下允许遮蔽：

- [`use` glob 遮蔽][`use` glob shadowing]
- [宏文本作用域遮蔽][Macro textual scope shadowing]

r[names.resolution.expansion.imports.ambiguity]
#### 歧义

r[names.resolution.expansion.imports.ambiguity.intro]
在展开时解析中存在某些情况，其中导入或宏调用的名称可能引用多个宏定义、`use` 声明或模块，而编译器无法一致地确定哪个候选项应遮蔽另一个。在这些情况下不允许遮蔽，编译器会发出歧义错误。

r[names.resolution.expansion.imports.ambiguity.glob-vs-glob]
名称不能通过有歧义的 glob 导入解析。只要名称未被使用，glob 导入允许导入相同命名空间中冲突的名称。来自有歧义的 glob 导入的冲突候选项名称仍可被非 glob 导入遮蔽并在不产生错误的情况下使用。错误发生在使用时，而非导入时。

```rust,compile_fail,E0659
mod m1 {
    pub struct Ambig;
}

mod m2 {
    pub struct Ambig;
}

// 正确：这在同一命名空间中引入了冲突的名称
// 但它们尚未被使用。
use m1::*;
use m2::*;

const _: () = {
    // 当使用具有冲突候选项的名称时发生错误。
    let x = Ambig; // 错误：`Ambig` 有歧义。
};
```

```rust,no_run
# mod m1 {
#     pub struct Ambig;
# }
#
# mod m2 {
#     pub struct Ambig;
# }
#
# use m1::*;
# use m2::*; // 正确：无名称冲突。
const _: () = {
    // 这是允许的，因为解析并非通过有歧义的 glob。
    struct Ambig;
    let x = Ambig; // 正确。
};
```

允许多个 glob 导入导入相同的名称，并且如果导入的是相同的项（跟随重新导出），则允许使用该名称。名称的可见性是导入的最大可见性。

```rust,no_run
mod m1 {
    pub struct Ambig;
}

mod m2 {
    // 这从第二个模块重新导出相同的 `Ambig` 项。
    pub use super::m1::Ambig;
}

mod m3 {
    // 这两者都导入相同的 `Ambig`。
    //
    // `Ambig` 的可见性是 `pub`，因为这是这两个
    // `use` 声明之间的最大可见性。
    pub use super::m1::*;
    use super::m2::*;
}

mod m4 {
    // `Ambig` 可以通过 `m3` 的 globs 使用，并且仍然具有
    // `pub` 可见性。
    pub use crate::m3::Ambig;
}

const _: () = {
    // 因此，我们可以在此处使用它。
    let _ = m4::Ambig; // 正确。
};
# fn main() {}
```

r[names.resolution.expansion.imports.ambiguity.glob-vs-outer]
当[外部作用域][outer scope]中存在另一个候选项时，导入和宏调用中的名称不能通过 glob 导入解析。

r[names.resolution.expansion.imports.ambiguity.panic-hack]
> [!NOTE]
> 当 [`core::panic!`] 或 [`std::panic!`] 之一由于[标准库预导入][standard library prelude]被带入作用域，而用户编写的 [glob 导入][glob import]将另一个带入作用域时，`rustc` 目前允许使用 `panic!`，即使它是有歧义的。用户编写的 glob 导入优先以解决此歧义。
>
> 在 Rust 2021 及更高版本中，[`core::panic!`] 和 [`std::panic!`] 行为相同。但在更早的版次中，它们有所不同；只有 [`std::panic!`] 接受 [`String`] 作为格式化参数。
>
> 例如，这是一个错误：
>
> ```rust,edition2018,compile_fail,E0308
> extern crate core;
> use ::core::prelude::v1::*;
> fn main() {
>     panic!(std::string::String::new()); // 错误。
> }
> ```
>
> 而这是被接受的：
>
> <!-- ignore: Can't test with `no_std`. -->
> ```rust,edition2018,ignore
> #![no_std]
> extern crate std;
> use ::std::prelude::v1::*;
> fn main() {
>     panic!(std::string::String::new()); // 正确。
> }
> ```
>
> 不要依赖此行为；计划将其移除。
>
> 详情请参见 [Rust issue #147319](https://github.com/rust-lang/rust/issues/147319)。

```rust,compile_fail,E0659
mod glob {
    pub mod ambig {
        pub struct Name;
    }
}

// 外部 `ambig` 候选项。
pub mod ambig {
    pub struct Name;
}

const _: () = {
    // 无法通过此 glob 解析 `ambig`，
    // 因为上面存在外部 `ambig` 候选项。
    use glob::*;
    use ambig::Name; // 错误：`ambig` 有歧义。
};
```

```rust,compile_fail,E0659
// 同上，但使用宏。
pub mod m {
    macro_rules! f {
        () => {};
    }
    pub(crate) use f;
}
pub mod glob {
    macro_rules! f {
        () => {};
    }
    pub(crate) use f as ambig;
}

use m::f as ambig;

const _: () = {
    use glob::*;
    ambig!(); // 错误：`ambig` 有歧义。
};
```

> [!NOTE]
> 这些歧义错误是展开时解析特有的。在后续解析阶段中，某个名称存在多个候选项不被视为错误。只要导入本身没有歧义，就始终会有一个唯一的无歧义的最接近解析。
>
> ```rust,no_run
> mod glob {
>     pub const AMBIG: u8 = 1;
> }
>
> mod outer {
>     pub const AMBIG: u8 = 2;
> }
>
> use outer::AMBIG;
>
> const C: () = {
>     use glob::*;
>     assert!(AMBIG == 1);
>     //      ^---- 此 `AMBIG` 在主解析期间解析。
> };
> ```

r[names.resolution.expansion.imports.ambiguity.path-vs-textual-macro]
名称不能通过有歧义的宏重新导出解析。当宏重新导出会遮蔽[外部作用域][outer scope]中相同名称的文本宏候选项时，宏重新导出是有歧义的。

```rust,compile_fail,E0659
// 文本宏候选项。
macro_rules! ambig {
    () => {}
}

// 基于路径的宏候选项。
macro_rules! path_based {
    () => {}
}

pub fn f() {
    // 将 `path_based` 宏定义重新导出为 `ambig`
    // 不能遮蔽通过文本宏作用域解析的
    // `ambig` 宏定义。
    use path_based as ambig;
    ambig!(); // 错误：`ambig` 有歧义。
}
```

> [!NOTE]
> 此限制是由于编译器中的实现细节，特别是当前的作用域访问逻辑和支持此行为的复杂性。此歧义错误可能在未来被移除。

r[names.resolution.expansion.macros]
### 宏

r[names.resolution.expansion.macros.intro]
宏通过遍历可用作用域以查找可用候选项来解析。宏分为两个子命名空间，一个用于类函数宏，另一个用于属性和派生宏。来自错误子命名空间的解析候选项将被忽略。

r[names.resolution.expansion.macros.visitation-order]
可用的作用域类型按以下顺序访问。每种作用域类型代表一个或多个作用域。

* [派生辅助属性][Derive helpers]
* [文本作用域宏][Textual scope macros]
* [基于路径的作用域宏][Path-based scope macros]
* [`macro_use` 预导入][`macro_use` prelude]
* [标准库预导入][Standard library prelude]
* [内置属性][Builtin attributes]

> [!NOTE]
> 编译器将尝试解析在其关联宏将其引入作用域之前使用的派生辅助属性。此作用域在解析正确位于作用域中的派生辅助属性候选项的作用域之后访问。此行为计划被移除。
>
> 更多信息请参见[派生辅助属性作用域][derive helper scope]。

> [!NOTE]
> 此访问顺序可能在未来改变，例如根据文本作用域和基于路径的作用域候选项的词法作用域交错访问它们。

> [!EDITION-2018]
> 从 2018 版次开始，当存在 [`#[no_implicit_prelude]`][names.preludes.no_implicit_prelude] 时，不会访问 `#[macro_use]` 预导入。

r[names.resolution.expansion.macros.reserved-names]
名称 `cfg` 和 `cfg_attr` 在宏属性[子命名空间][sub-namespace]中是保留的。

r[names.resolution.expansion.macros.ambiguity]
#### 歧义

r[names.resolution.expansion.macros.ambiguity.more-expanded-vs-outer]
名称不能通过宏展开内部的有歧义的候选项解析。当宏展开内部的候选项会遮蔽来自第一个候选项的宏展开外部的相同名称的候选项，并且被解析的名称的调用也来自第一个候选项的宏展开外部时，宏展开内部的候选项是有歧义的。

```rust,compile_fail,E0659
macro_rules! define_ambig {
    () => {
        macro_rules! ambig {
            () => {}
        }
    }
}

// 为 `ambig` 宏调用引入外部候选项定义。
macro_rules! ambig {
    () => {}
}

// 在宏展开内部为 `ambig` 引入第二个候选项定义。
define_ambig!();

// 来自第二次 `define_ambig` 调用的 `ambig` 定义
// 是最内层的候选项。
//
// 来自第一次 `define_ambig` 调用的 `ambig` 定义
// 是第二个候选项。
//
// 编译器检查第一个候选项是否位于宏展开内部，
// 第二个候选项是否不来自同一宏展开内部，
// 并且被解析的名称是否不来自同一宏展开内部。
ambig!(); // 错误：`ambig` 有歧义。
```

反过来不被视为歧义。

```rust,no_run
# macro_rules! define_ambig {
#     () => {
#         macro_rules! ambig {
#             () => {}
#         }
#     }
# }
// 交换定义顺序。
define_ambig!();
macro_rules! ambig {
    () => {}
}
// 最内层的候选项现在展开程度较低，因此它可以遮蔽
// 其上方的宏展开定义。
ambig!();
```

如果被解析的调用位于最内层候选项的展开内部，也不被视为歧义。

```rust,no_run
macro_rules! ambig {
    () => {}
}

macro_rules! define_and_invoke_ambig {
    () => {
        // 定义最内层候选项。
        macro_rules! ambig {
            () => {}
        }

        // `ambig` 的调用与最内层候选项位于同一展开中。
        ambig!(); // 正确
    }
}

define_and_invoke_ambig!();
```

两个定义是否来自同一宏的调用并不重要；最外层候选项仍被视为"展开程度较低"，因为它不在包含最内层候选项定义的展开内部。

```rust,compile_fail,E0659
# macro_rules! define_ambig {
#     () => {
#         macro_rules! ambig {
#             () => {}
#         }
#     }
# }
define_ambig!();
define_ambig!();
ambig!(); // 错误：`ambig` 有歧义。
```

这也适用于导入，只要名称的最内层候选项来自宏展开内部。

```rust,compile_fail,E0659
macro_rules! define_ambig {
    () => {
        mod ambig {
            pub struct Name;
        }
    }
}

mod ambig {
    pub struct Name;
}

const _: () = {
    // 在此宏展开中为 `ambig` mod 引入最内层候选项。
    define_ambig!();
    use ambig::Name; // 错误：`ambig` 有歧义。
};
```

r[names.resolution.expansion.macros.ambiguity.built-in-attr]
用户定义的属性或派生宏不能遮蔽内置的非宏属性（例如 inline）。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
// with-helper/src/lib.rs
# use proc_macro::TokenStream;
#[proc_macro_derive(WithHelperAttr, attributes(non_exhaustive))]
//                                             ^^^^^^^^^^^^^^
//                                   用户定义的属性候选项。
// ...
# pub fn derive_with_helper_attr(_item: TokenStream) -> TokenStream {
#     TokenStream::new()
# }
```

<!-- ignore: requires external crates -->
```rust,ignore
// src/lib.rs
#[derive(with_helper::WithHelperAttr)]
#[non_exhaustive] // 错误：`non_exhaustive` 有歧义。
struct S;
```

> [!NOTE]
> 无论内置属性是哪个名称的候选项，此规则都适用：
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> // with-helper/src/lib.rs
> # use proc_macro::TokenStream;
> #
> #[proc_macro_derive(WithHelperAttr, attributes(helper))]
> //                                             ^^^^^^
> //                                 用户定义的属性候选项。
> // ...
> # pub fn derive_with_helper_attr(_item: TokenStream) -> TokenStream {
> #     TokenStream::new()
> # }
> ```
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> // src/lib.rs
> use inline as helper;
> //            ^----- 通过重新导出的内置属性候选项。
>
> #[derive(with_helper::WithHelperAttr)]
> #[helper] // 错误：`helper` 有歧义。
> struct S;
> ```

r[names.resolution.primary]
## 主名称解析
> [!NOTE]
> 这是一个占位符，用于未来展开关于主名称解析的内容。

r[names.resolution.type-relative]
## 类型相关解析
> [!NOTE]
> 这是一个占位符，用于未来展开关于类型相关解析的内容。

[AST]: glossary.ast
[Builtin attributes]: ./preludes.md#r-names.preludes.lang
[Derive helpers]: ../procedural-macros.md#r-macro.proc.derive.attributes
[Macros]: ../macros.md
[Path-based scope macros]: ../macros.md#r-macro.invocation.name-resolution
[Standard library prelude]: ./preludes.md#r-names.preludes.std
[Textual scope macros]: ../macros-by-example.md#r-macro.decl.scope.textual
[`let` bindings]: ../statements.md#let-statements
[`macro_use` prelude]: ./preludes.md#r-names.preludes.macro_use
[`use` declarations]: ../items/use-declarations.md
[`use` glob shadowing]: ../items/use-declarations.md#r-items.use.glob.shadowing
[derive helper scope]: ../procedural-macros.md#r-macro.proc.derive.attributes.scope
[glob import]: items.use.glob
[item definitions]: ../items.md
[macro invocations]: ../macros.md#macro-invocation
[macro textual scope shadowing]: ../macros-by-example.md#r-macro.decl.scope.textual.shadow
[name resolution ambiguities]: #r-names.resolution.expansion.imports.ambiguity
[namespaces]: ../names/namespaces.md
[outer scope]: #r-names.resolution.general.scopes
[path-based scope]: ../macros.md#r-macro.invocation.name-resolution
[scope]: ../names/scopes.md
[sub-namespace]: ../names/namespaces.md#r-names.namespaces.sub-namespaces
[type-relative paths]: names.resolution.type-relative
[visibility]: ../visibility-and-privacy.md
