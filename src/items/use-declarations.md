r[items.use]
# use 声明

r[items.use.syntax]
```grammar,items
UseDeclaration -> `use` UseTree `;`

UseTree ->
      (SimplePath? `::`)? `*`
    | (SimplePath? `::`)? `{` (UseTree ( `,`  UseTree )* `,`?)? `}`
    | SimplePath ( `as` ( IDENTIFIER | `_` ) )?
```

r[items.use.intro]
*use 声明*创建一个或多个与其他[路径][path]同义的本地名称绑定。通常，`use` 声明用于缩短引用模块程序项所需的路径。这些声明可以出现在[模块][modules]和[块][blocks]中，通常位于顶部。`use` 声明有时也称为*导入*，如果它是公开的，则称为*重导出*。

[path]: ../paths.md
[modules]: modules.md
[blocks]: ../expressions/block-expr.md

r[items.use.forms]
use 声明支持许多便捷的快捷方式：

r[items.use.forms.multiple]
* 使用花括号语法同时绑定具有公共前缀的路径列表：`use a::b::{c, d, e::f, g::h::i};`

r[items.use.forms.self]
* 使用 `self` 关键字同时绑定具有公共前缀的路径列表及其公共父模块：`use a::b::{self, c, d::e};`

r[items.use.forms.as]
* 使用语法 `use p::q::r as x;` 将目标名称重新绑定为新的本地名称。这也可以与前两个特性一起使用：`use a::b::{self as ab, c as abc}`。

r[items.use.forms.glob]
* 使用星号通配符语法绑定匹配给定前缀的所有路径：`use a::b::*;`。

r[items.use.forms.nesting]
* 多次嵌套前述特性的分组：`use a::b::{self as ab, c, d::{*, e::f}};`

`use` 声明示例：

```rust
use std::collections::hash_map::{self, HashMap};

fn foo<T>(_: T){}
fn bar(map1: HashMap<String, usize>, map2: hash_map::HashMap<String, usize>){}

fn main() {
    // use 声明也可以存在于函数内部
    use std::option::Option::{Some, None};

    // 等价于 'foo(vec![std::option::Option::Some(1.0f64),
    // std::option::Option::None]);'
    foo(vec![Some(1.0f64), None]);

    // `hash_map` 和 `HashMap` 都在作用域中。
    let map1 = HashMap::new();
    let map2 = hash_map::HashMap::new();
    bar(map1, map2);
}
```

r[items.use.visibility]
## `use` 可见性 {#use-visibility}

r[items.use.visibility.intro]
与程序项一样，`use` 声明默认对包含它的模块是私有的。也与程序项一样，如果被 `pub` 关键字限定，`use` 声明可以是公开的。这样的 `use` 声明用于*重导出*一个名称。因此，公开的 `use` 声明可以将某个公开名称*重定向*到不同的目标定义：即使是一个位于不同模块内、具有私有规范路径的定义。

r[items.use.visibility.unambiguous]
如果这样一系列重定向形成循环或无法无歧义地解析，则它们表示编译时错误。

重导出示例：

```rust
mod quux {
    pub use self::foo::{bar, baz};
    pub mod foo {
        pub fn bar() {}
        pub fn baz() {}
    }
}

fn main() {
    quux::bar();
    quux::baz();
}
```

在此示例中，模块 `quux` 重导出了 `foo` 中定义的两个公开名称。

r[items.use.path]
## `use` 路径 {#use-paths}

r[items.use.path.intro]
`use` 程序项中允许的[路径][paths]遵循 [SimplePath] 语法，并且类似于表达式中可以使用的路径。它们可以为以下内容创建绑定：

* 可命名的[程序项][items]
* [枚举变体][Enum variants]
* [内置类型][Built-in types]
* [属性][Attributes]
* [Derive 宏][Derive macros]
* [`macro_rules`]

r[items.use.path.disallowed]
它们不能导入[关联程序项][associated items]、[泛型参数][generic parameters]、[局部变量][local variables]、带有 [`Self`] 的路径或[工具属性][tool attributes]。更多限制如下所述。

r[items.use.path.namespace]
`use` 将为导入实体的所有[命名空间][namespaces]创建绑定，但 `self` 导入只从类型命名空间导入（如下所述）为例外。例如，以下示例展示了在两个命名空间中为同一个名称创建绑定：

```rust
mod stuff {
    pub struct Foo(pub i32);
}

// 导入 `Foo` 类型和 `Foo` 构造器。
use stuff::Foo;

fn example() {
    let ctor = Foo; // 使用值命名空间中的 `Foo`。
    let x: Foo = ctor(123); // 使用类型命名空间中的 `Foo`。
}
```

r[items.use.path.edition2018]
> [!EDITION-2018]
> 在 2015 版本中，`use` 路径相对于 crate 根。例如：
>
> ```rust,edition2015
> mod foo {
>     pub mod example { pub mod iter {} }
>     pub mod baz { pub fn foobaz() {} }
> }
> mod bar {
>     // 从 crate 根解析 `foo`。
>     use foo::example::iter;
>     // `::` 前缀显式地从 crate 根
>     // 解析 `foo`。
>     use ::foo::baz::foobaz;
> }
>
> # fn main() {}
> ```
>
> 2015 版本不允许 use 声明引用[外部预导入][extern prelude]。因此，在 2015 中仍然需要 [`extern crate`] 声明才能在 `use` 声明中引用外部 crate。从 2018 版本开始，`use` 声明可以像 `extern crate` 一样指定外部 crate 依赖项。

r[items.use.as]
## `as` 重命名 {#as-renames}

`as` 关键字可用于更改导入实体的名称。例如：

```rust
// 为函数 `foo` 创建非公开别名 `bar`。
use inner::foo as bar;

mod inner {
    pub fn foo() {}
}
```

r[items.use.multiple-syntax]
## 花括号语法 {#brace-syntax}

r[items.use.multiple-syntax.intro]
花括号可以用在路径的最后一段，以从前一段导入多个实体，或者，如果没有前一段，则从当前作用域导入。花括号可以嵌套，创建路径树，其中每个段分组都与父级逻辑组合以创建完整路径。

```rust
// 创建以下内容的绑定：
// - `std::collections::BTreeSet`
// - `std::collections::hash_map`
// - `std::collections::hash_map::HashMap`
use std::collections::{BTreeSet, hash_map::{self, HashMap}};
```

r[items.use.multiple-syntax.empty]
空花括号不导入任何内容，尽管会验证前导路径是否可访问。
<!-- 这不太对，参见: https://github.com/rust-lang/rust/issues/61826 -->

r[items.use.multiple-syntax.edition2018]
> [!EDITION-2018]
> 在 2015 版本中，路径相对于 crate 根，因此 `use {foo, bar};` 这样的导入将从 crate 根导入名称 `foo` 和 `bar`，而从 2018 开始，这些名称相对于当前作用域。

r[items.use.self]
## `self` 导入 {#self-imports}

r[items.use.self.intro]
关键字 `self` 可以在[花括号语法][brace syntax]中使用，以在其自身名称下创建父实体的绑定。

```rust
mod stuff {
    pub fn foo() {}
    pub fn bar() {}
}
mod example {
    // 创建 `stuff` 和 `foo` 的绑定。
    use crate::stuff::{self, foo};
    pub fn baz() {
        foo();
        stuff::bar();
    }
}
# fn main() {}
```

> [!NOTE]
> `self` 也可以用作路径的第一段。将 `self` 用作第一段和在 `use` 花括号中使用 `self` 在逻辑上是相同的；它表示父段的当前模块，或者如果没有父段，则表示当前模块。有关前导 `self` 含义的更多信息，请参见路径章节中的 [`self`]。

r[items.use.self.trailing]
`self` 可以作为 `use` 路径的最后一段出现，前面加上 `::`。`P::self` 形式的路径等价于 `P::{self}`，`P::self as name` 等价于 `P::{self as name}`。

```rust
mod m {
    pub enum E { V1, V2 }
}
use m::self as _; // 等价于 `use m::{self as _};`。
use m::E::self; // 等价于 `use m::E::{self};`。
# fn main() {}
```

> [!NOTE]
> 有关前导路径的限制，请参见 [paths.qualifiers.mod-self.trailing]。

r[items.use.self.module]
当 `self` 在[花括号语法][brace syntax]中使用时，花括号组前面的路径必须解析为[模块][module]、[枚举][enumeration]或 [trait]。

```rust
mod m {
    pub enum E { V1, V2 }
    pub trait Tr { fn f(&self); }
}
use m::{self as _}; // OK：模块可以是 `self` 的父级。
use m::E::{self, V1}; // OK：枚举可以是 `self` 的父级。
use m::Tr::{self}; // OK：trait 可以是 `self` 的父级。
# fn main() {}
```

```rust,compile_fail,E0432
struct S {}
use S::{self as _}; // 错误：结构体不能是 `self` 的父级。
# fn main() {}
```

r[items.use.self.namespace]
`self` 仅从父实体的[类型命名空间][type namespace]创建绑定。例如，在以下代码中，仅导入了 `foo` 模块：

```rust,compile_fail
mod bar {
    pub mod foo {}
    pub fn foo() {}
}

// 这仅导入模块 `foo`。函数 `foo` 位于
// 值命名空间中，没有被导入。
use bar::foo::{self};

fn main() {
    foo(); //~ 错误：`foo` 是一个模块
}
```

r[items.use.glob]
## 通配符导入 {#glob-imports}

r[items.use.glob.intro]
`*` 字符可以作为 `use` 路径的最后一段，以从前一段的实体中导入所有可导入的实体。例如：

```rust
// 为 `bar` 创建非公开别名。
use foo::*;

mod foo {
    fn i_am_private() {}
    enum Example {
        V1,
        V2,
    }
    pub fn bar() {
        // 创建 `Example` 枚举的 `V1` 和 `V2`
        // 的本地别名。
        use Example::*;
        let x = V1;
    }
}
```

r[items.use.glob.shadowing]
程序项和命名导入允许遮蔽来自同一[命名空间][namespace]中通配符导入的名称。也就是说，如果同一命名空间中已存在由另一个程序项定义的名称，则通配符导入将被遮蔽。例如：

```rust
// 这创建了对 `clashing::Foo` 元组结构体
// 构造器的绑定，但不会导入其类型，因为
// 这与此处定义的 `Foo` 结构体冲突。
//
// 请注意，此处的定义顺序并不重要。
use clashing::*;
struct Foo {
    field: f32,
}

fn do_stuff() {
    // 使用 `clashing::Foo` 的构造器。
    let f1 = Foo(123);
    // 结构体表达式使用上面定义的
    // `Foo` 结构体的类型。
    let f2 = Foo { field: 1.0 };
    // `Bar` 也因通配符导入而在作用域中。
    let z = Bar {};
}

mod clashing {
    pub struct Foo(pub i32);
    pub struct Bar {}
}
```

> [!NOTE]
> 对于不允许遮蔽的区域，请参见[名称解析歧义][name resolution ambiguities]。

r[items.use.glob.last-segment-only]
`*` 不能用作第一段或中间段。

r[items.use.glob.self-import]
`*` 不能用于将模块的内容导入自身（如 `use self::*;`）。

r[items.use.glob.edition2018]
> [!EDITION-2018]
> 在 2015 版本中，路径相对于 crate 根，因此 `use *;` 这样的导入是有效的，它表示从 crate 根导入所有内容。这不能在 crate 根本身中使用。

r[items.use.as-underscore]
## 下划线导入 {#underscore-imports}

r[items.use.as-underscore.intro]
可以使用下划线形式 `use path as _` 导入程序项而不绑定到名称。这对于导入 trait 以便使用其方法而不导入 trait 的符号特别有用，例如如果 trait 的符号可能与另一个符号冲突。另一个例子是链接外部 crate 而不导入其名称。

r[items.use.as-underscore.glob]
星号通配符导入将以其不可命名形式导入以 `_` 导入的程序项。

```rust
mod foo {
    pub trait Zoo {
        fn zoo(&self) {}
    }

    impl<T> Zoo for T {}
}

use self::foo::Zoo as _;
struct Zoo;  // 下划线导入避免了与此程序项的名称冲突。

fn main() {
    let z = Zoo;
    z.zoo();
}
```

r[items.use.as-underscore.macro]
唯一的、不可命名的符号在宏展开之后创建，因此宏可以安全地多次发出对 `_` 导入的引用。例如，以下不应产生错误：

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(use std as _;);
// 这会展开为：
// use std as _;
// use std as _;
```

r[items.use.restrictions]
## 限制 {#restrictions}

以下规则是有效 `use` 声明的限制。

r[items.use.restrictions.crate-alias]
当使用 `crate` 导入当前 crate 时，必须使用 `as` 来定义绑定名称。

> [!EXAMPLE]
> ```rust
> use crate as root;
> use crate::{self as root2};
>
> // 不允许：
> // use crate;
> // use crate::{self};
> ```

r[items.use.restrictions.macro-crate-alias]
当在宏转录器中使用 [`$crate`] 导入当前 crate 时，必须使用 `as` 来定义绑定名称。

> [!EXAMPLE]
> ```rust
> macro_rules! import_crate_root {
>     () => {
>         use $crate as my_crate;
>         use $crate::{self as my_crate2};
>     };
> }
> ```

r[items.use.restrictions.self-alias]
当使用 `self` 导入当前模块时，必须使用 `as` 来定义绑定名称。

> [!EXAMPLE]
> ```rust
> use {self as this_module};
> use self as this_module2;
> use self::{self as this_module3};
>
> // 不允许：
> // use {self};
> // use self;
> // use self::{self};
> ```

r[items.use.restrictions.super-alias]
当使用 `super` 导入父模块时，必须使用 `as` 来定义绑定名称。

> [!EXAMPLE]
> ```rust
> mod a {
>     mod b {
>         use super as parent;
>         use super::{self as parent2};
>         use self::super as parent3;
>         use super::super as grandparent;
>         use super::super::{self as grandparent2};
>
>         // 不允许：
>         // use super;
>         // use super::{self};
>         // use self::super;
>         // use super::super;
>         // use super::super::{self};
>     }
> }
> ```

r[items.use.restrictions.extern-prelude]
`::` 作为[外部预导入][extern prelude]不能被导入。

> [!EXAMPLE]
> ```rust,edition2018,compile_fail
> use ::{self as root}; //~ 错误
> ```

> [!EDITION-2018]
> 在 2015 版本中，前缀 `::` 指向 crate 根，因此 `use ::{self as root};` 是允许的，因为它与 `use crate::{self as root};` 相同。从 2018 版本开始，`::` 前缀指向外部预导入，不能直接导入。
>
> ```rust,edition2015
> use ::{self as root}; //~ OK
> ```

r[items.use.restrictions.duplicate-name]
与任何程序项定义一样，`use` 导入不能在模块或块的同一命名空间中创建同名的重复绑定。

r[items.use.restrictions.variant]
`use` 路径不能通过[类型别名][type alias]引用枚举变体。

> [!EXAMPLE]
> ```rust,compile_fail
> enum MyEnum {
>   MyVariant
> }
> type TypeAlias = MyEnum;
>
> use MyEnum::MyVariant; //~ OK
> use TypeAlias::MyVariant; //~ 错误
> ```

[`$crate`]: paths.qualifiers.macro-crate
[Attributes]: ../attributes.md
[brace syntax]: items.use.multiple-syntax
[Built-in types]: ../types.md
[Derive macros]: macro.proc.derive
[Enum variants]: enumerations.md
[enumeration]: items.enum
[`extern crate`]: extern-crates.md
[`macro_rules`]: ../macros-by-example.md
[`self`]: ../paths.md#self
[associated items]: associated-items.md
[extern prelude]: ../names/preludes.md#extern-prelude
[generic parameters]: generics.md
[items]: ../items.md
[local variables]: ../variables.md
[module]: items.mod
[name resolution ambiguities]: names.resolution.expansion.imports.ambiguity
[namespace]: ../names/namespaces.md
[namespaces]: ../names/namespaces.md
[paths]: ../paths.md
[tool attributes]: ../attributes.md#tool-attributes
[trait]: items.traits
[type alias]: type-aliases.md
[type namespace]: ../names/namespaces.md
