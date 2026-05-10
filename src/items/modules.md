r[items.mod]
# 模块

r[items.mod.syntax]
```grammar,items
Module ->
      `unsafe`? `mod` IDENTIFIER `;`
    | `unsafe`? `mod` IDENTIFIER `{`
        InnerAttribute*
        Item*
      `}`
```

r[items.mod.intro]
模块是零个或多个[程序项][items]的容器。

r[items.mod.def]
*模块项*是一个用花括号括起来、命名并以关键字 `mod` 为前缀的模块。模块项在构成 crate 的模块树中引入一个新的命名模块。

r[items.mod.nesting]
模块可以任意嵌套。

模块示例：

```rust
mod math {
    type Complex = (f64, f64);
    fn sin(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
    fn cos(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
    fn tan(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
}
```

r[items.mod.namespace]
模块定义在其所在模块或块的[类型命名空间][type namespace]中。

r[items.mod.multiple-items]
在模块的同一命名空间中定义多个同名程序项是错误的。有关限制和遮蔽行为的更多详细信息，请参见[作用域章节][scopes chapter]。

r[items.mod.unsafe]
语法上允许 `unsafe` 关键字出现在 `mod` 关键字之前，但在语义层面会被拒绝。这允许宏消费该语法并利用 `unsafe` 关键字，然后将其从 token 流中移除。

r[items.mod.outlined]
## 模块源文件名 {#module-source-filenames}

r[items.mod.outlined.intro]
没有主体的模块从外部文件加载。当模块没有 `path` 属性时，该文件的路径与逻辑[模块路径][module path]相对应。

r[items.mod.outlined.search]
祖先模块路径的各级组件是目录，模块的内容以模块名加上 `.rs` 扩展名命名的文件形式存在。例如，以下模块结构可以有以下对应的文件系统结构：

模块路径               | 文件系统路径  | 文件内容
------------------------- | ---------------  | -------------
`crate`                   | `lib.rs`         | `mod util;`
`crate::util`             | `util.rs`        | `mod config;`
`crate::util::config`     | `util/config.rs` |

r[items.mod.outlined.search-mod]
模块文件名也可以是模块名作为目录，其内容放在该目录下一个名为 `mod.rs` 的文件中。上面的示例也可以将 `crate::util` 的内容放在文件 `util/mod.rs` 中。不允许同时存在 `util.rs` 和 `util/mod.rs`。

> [!NOTE]
> 在 `rustc` 1.30 之前，使用 `mod.rs` 文件是加载带有嵌套子模块的模块的方式。鼓励使用新的命名约定，因为它更一致，并且可以避免项目中存在许多名为 `mod.rs` 的文件。

r[items.mod.outlined.path]
### `path` 属性 {#the-path-attribute}

r[items.mod.outlined.path.intro]
用于加载外部文件模块的目录和文件可以通过 `path` 属性来影响。

r[items.mod.outlined.path.search]
对于不在内联模块块中的模块上的 `path` 属性，文件路径相对于源文件所在的目录。例如，以下代码片段将根据其位置使用如下所示的路径：

<!-- ignore: requires external files -->
```rust,ignore
#[path = "foo.rs"]
mod c;
```

源文件    | `c` 的文件位置 | `c` 的模块路径
-------------- | ------------------- | ----------------------
`src/a/b.rs`   | `src/a/foo.rs`      | `crate::a::b::c`
`src/a/mod.rs` | `src/a/foo.rs`      | `crate::a::c`

r[items.mod.outlined.path.search-nested]
对于内联模块块中的 `path` 属性，文件路径的相对位置取决于 `path` 属性所在的源文件类型。"mod-rs" 源文件是根模块（如 `lib.rs` 或 `main.rs`）和文件名为 `mod.rs` 的模块。"non-mod-rs" 源文件是所有其他模块文件。在 mod-rs 文件中的内联模块块中，`path` 属性的路径相对于 mod-rs 文件的目录，包括作为目录的内联模块组件。对于 non-mod-rs 文件，情况相同，只是路径以 non-mod-rs 模块名称的目录开头。例如，以下代码片段将根据其位置使用如下所示的路径：

<!-- ignore: requires external files -->
```rust,ignore
mod inline {
    #[path = "other.rs"]
    mod inner;
}
```

源文件    | `inner` 的文件位置   | `inner` 的模块路径
-------------- | --------------------------| ----------------------------
`src/a/b.rs`   | `src/a/b/inline/other.rs` | `crate::a::b::inline::inner`
`src/a/mod.rs` | `src/a/inline/other.rs`   | `crate::a::inline::inner`

一个结合了上述 `path` 属性在内联模块和嵌套模块上规则的示例（适用于 mod-rs 和 non-mod-rs 文件）：

<!-- ignore: requires external files -->
```rust,ignore
#[path = "thread_files"]
mod thread {
    // 从 thread_files/tls.rs 加载 `local_data` 模块，相对于
    // 此源文件所在的目录。
    #[path = "tls.rs"]
    mod local_data;
}
```

r[items.mod.attributes]
## 模块上的属性 {#attributes-on-modules}

r[items.mod.attributes.intro]
模块和所有程序项一样，接受外部属性。它们也接受内部属性：对于有主体的模块，在 `{` 之后；或者对于源文件，在可选的 BOM 和 shebang 之后的文件开头。

r[items.mod.attributes.supported]
对模块有意义的内置属性有 [`cfg`]、[`deprecated`]、[`doc`]、[lint 检查属性][the lint check attributes]、[`path`] 和 [`no_implicit_prelude`]。模块也接受宏属性。

[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../rustdoc/the-doc-attribute.html
[`no_implicit_prelude`]: ../names/preludes.md#the-no_implicit_prelude-attribute
[`path`]: #the-path-attribute
[attribute]: ../attributes.md
[items]: ../items.md
[module path]: ../paths.md
[scopes chapter]: ../names/scopes.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[type namespace]: ../names/namespaces.md
