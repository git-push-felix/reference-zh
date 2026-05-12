r[vis]
# 可见性和隐私 {#visibility-and-privacy}

r[vis.syntax]
```grammar,items
Visibility ->
      `pub`
    | `pub` `(` `crate` `)`
    | `pub` `(` `self` `)`
    | `pub` `(` `super` `)`
    | `pub` `(` `in` SimplePath `)`
```

r[vis.intro]
这两个术语经常互换使用，它们试图传达的是对"此项目能否在此位置使用？"这个问题的答案。

r[vis.name-hierarchy]
Rust 的名称解析在全局的命名空间层次结构上运行。层次结构中的每个级别可以被视为某个项。项是上述提到的那几种之一，但也包括外部 crate。声明或定义一个新模块可以被视为在定义位置的层次结构中插入一棵新树。

r[vis.privacy]
为了控制接口是否可以跨模块使用，Rust 检查对项的每次使用，以查看是否应该允许。这是隐私警告生成的地方，或者说"你使用了另一个模块的私有项并且不被允许"。

r[vis.default]
默认情况下，所有内容都是*私有的*，有两个例外：`pub` Trait 中的关联项默认是公共的；`pub` 枚举中的枚举变体也默认是公共的。当一个项被声明为 `pub` 时，它可以被认为对外部世界是可访问的。例如：

```rust
# fn main() {}
// 声明一个私有结构体
struct Foo;

// 声明一个带有私有字段的公共结构体
pub struct Bar {
    field: i32,
}

// 声明一个带有两个公共变体的公共枚举
pub enum State {
    PubliclyAccessibleState,
    PubliclyAccessibleState2,
}
```

r[vis.access]
有了项是公共还是私有的概念，Rust 在两种情况下允许项访问：

1. 如果一个项是公共的，那么如果你可以从某个模块 `m` 访问该项的所有祖先模块，就可以从 `m` 外部访问它。你还可以通过重新导出来潜在地命名该项。见下文。
2. 如果一个项是私有的，它可以被当前模块及其后代访问。

这两种情况对于创建暴露公共 API 同时隐藏内部实现细节的模块层次结构来说出奇地强大。为了帮助解释，这里有几个用例及其含义：

* 库开发者需要将功能暴露给链接到其库的 crate。作为第一种情况的推论，这意味着任何可从外部使用的内容必须从根到目标项都是 `pub`。链中的任何私有项都将禁止外部访问。

* 一个 crate 需要一个对其自身全局可用的"辅助模块"，但不想将辅助模块暴露为公共 API。为此，crate 层次结构的根将有一个私有模块，该模块内部具有"公共 API"。由于整个 crate 是根的后代，整个本地 crate 可以通过第二种情况访问此私有模块。

* 当为某个模块编写单元测试时，一个常见的惯用法是让一个名为 `mod test` 的模块直接作为待测试模块的子模块。此模块可以通过第二种情况访问父模块的任何项，这意味着内部实现细节也可以从子模块无缝测试。

在第二种情况下，它提到私有项"可以被"当前模块及其后代"访问"，但访问项的确切含义取决于该项是什么。

r[vis.use]
例如，访问一个模块意味着查看其内部（以导入更多项）。另一方面，访问一个函数意味着它被调用。此外，路径表达式和导入语句被视为访问项，这意味着导入/表达式仅在目标位于当前可见性作用域中时才有效。

以下是一个示例程序，展示了上述三种情况：

```rust
// 此模块是私有的，意味着没有外部 crate 可以访问此模块。
// 然而，因为它在当前 crate 的根部是私有的，
// crate 中的任何模块都可以访问此模块中的任何公开可见项。
mod crate_helper_module {

    // 此函数可以被当前 crate 中的任何内容使用
    pub fn crate_helper() {}

    // 此函数*不能*被 crate 中的任何其他内容使用。它在
    // `crate_helper_module` 外部不可公开访问，因此只有
    // 当前模块及其后代可以访问它。
    fn implementation_detail() {}
}

// 此函数是"对根公开的"，意味着它可用于链接到此 crate 的外部 crate。
pub fn public_api() {}

// 类似于 'public_api'，此模块是公共的，因此外部 crate 可以查看其内部。
pub mod submodule {
    use crate::crate_helper_module;

    pub fn my_method() {
        // 本地 crate 中的任何项都可以通过上述两条规则的组合
        // 调用辅助模块的公共接口。
        crate_helper_module::crate_helper();
    }

    // 此函数对不是 `submodule` 后代的任何模块隐藏
    fn my_implementation() {}

    #[cfg(test)]
    mod test {

        #[test]
        fn test_my_implementation() {
            // 因为此模块是 `submodule` 的后代，它被允许
            // 访问 `submodule` 内部的私有项而不会违反隐私。
            super::my_implementation();
        }
    }
}

# fn main() {}
```

为了让 Rust 程序通过隐私检查关，所有路径必须是给定上述两条规则的有效访问。这包括所有 use 语句、表达式、类型等。

r[vis.scoped]
## `pub(in path)`、`pub(crate)`、`pub(super)` 和 `pub(self)` {#pubin-path-pubcrate-pubsuper-and-pubself}

r[vis.scoped.intro]
除了 public 和 private 之外，Rust 允许用户将项声明为仅在给定作用域内可见。`pub` 限制的规则如下：

r[vis.scoped.in]
- `pub(in path)` 使项在提供的 `path` 内可见。`path` 必须是一个简单路径，解析为正在声明其可见性的项的祖先模块。`path` 中的每个标识符必须直接引用模块（而不是通过 `use` 语句引入的名称）。

r[vis.scoped.crate]
- `pub(crate)` 使项在当前 crate 内可见。

r[vis.scoped.super]
- `pub(super)` 使项对父模块可见。这等价于 `pub(in super)`。

r[vis.scoped.self]
- `pub(self)` 使项对当前模块可见。这等价于 `pub(in self)` 或根本不使用 `pub`。

r[vis.scoped.edition2018]
> [!EDITION-2018]
> 从 2018 版次开始，`pub(in path)` 的路径必须以 `crate`、`self` 或 `super` 开头。2015 版次还可以使用以 `::` 或来自 crate 根的模块开头的路径。

以下是一个示例：

```rust,edition2015
pub mod outer_mod {
    pub mod inner_mod {
        // 此函数在 `outer_mod` 内可见
        pub(in crate::outer_mod) fn outer_mod_visible_fn() {}
        // 与上面相同，这仅在 2015 版次中有效。
        pub(in outer_mod) fn outer_mod_visible_fn_2015() {}

        // 此函数对整个 crate 可见
        pub(crate) fn crate_visible_fn() {}

        // 此函数在 `outer_mod` 内可见
        pub(super) fn super_mod_visible_fn() {
            // 此函数可见，因为我们在同一个 `mod` 中
            inner_mod_visible_fn();
        }

        // 此函数仅在 `inner_mod` 内可见，
        // 这与保留私有相同。
        pub(self) fn inner_mod_visible_fn() {}
    }
    pub fn foo() {
        inner_mod::outer_mod_visible_fn();
        inner_mod::crate_visible_fn();
        inner_mod::super_mod_visible_fn();

        // 此函数不再可见，因为我们在 `inner_mod` 外部
        // 错误！`inner_mod_visible_fn` 是私有的
        //inner_mod::inner_mod_visible_fn();
    }
}

fn bar() {
    // 此函数仍然可见，因为我们在同一个 crate 中
    outer_mod::inner_mod::crate_visible_fn();

    // 此函数不再可见，因为我们在 `outer_mod` 外部
    // 错误！`super_mod_visible_fn` 是私有的
    //outer_mod::inner_mod::super_mod_visible_fn();

    // 此函数不再可见，因为我们在 `outer_mod` 外部
    // 错误！`outer_mod_visible_fn` 是私有的
    //outer_mod::inner_mod::outer_mod_visible_fn();

    outer_mod::foo();
}

fn main() { bar() }
```

> [!NOTE]
> 此语法仅对项的可见性增加了另一种限制。它不保证该项在指定作用域的所有部分都可见。要访问一个项，其所有父项直到当前作用域也必须仍然可见。

r[vis.reexports]
## 重新导出和可见性

r[vis.reexports.intro]
Rust 允许通过 `pub use` 指令公开重新导出项。因为这是一个公共指令，这允许通过上述规则在当前模块中使用该项。它本质上允许对重新导出的项进行公共访问。例如，此程序是有效的：

```rust
pub use self::implementation::api;

mod implementation {
    pub mod api {
        pub fn f() {}
    }
}

# fn main() {}
```

这意味着任何引用 `implementation::api::f` 的外部 crate 将收到隐私违规，而路径 `api::f` 将被允许。

r[vis.reexports.private-item]
当重新导出私有项时，可以认为允许通过重新导出"短路"隐私链，而不是像通常那样通过命名空间层次结构传递。
