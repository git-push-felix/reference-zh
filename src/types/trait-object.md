r[type.trait-object]
# Trait 对象

r[type.trait-object.syntax]
```grammar,types
TraitObjectType -> `dyn`? Bounds

TraitObjectTypeOneBound -> `dyn`? TraitBound
```

r[type.trait-object.intro]
*trait 对象*是实现了某一组 trait 的另一种类型的不透明值。这组 trait 由一个 [dyn 兼容][dyn compatible]的*基础 trait* 加上任意数量的[自动 trait][auto traits]组成。

r[type.trait-object.impls]
Trait 对象实现了基础 trait、其自动 trait 以及基础 trait 的任何[超 trait][supertraits]。

r[type.trait-object.name]
Trait 对象写作关键字 `dyn` 后跟一组 trait 边界，但对 trait 边界有如下限制。

r[type.trait-object.constraint]
不能有超过一个非自动 trait，不能有超过一个生命周期，并且不允许使用 opt-out 边界（例如 `?Sized`）。此外，trait 路径可以用括号括起来。

例如，给定一个 trait `Trait`，以下所有都是 trait 对象：

* `dyn Trait`
* `dyn Trait + Send`
* `dyn Trait + Send + Sync`
* `dyn Trait + 'static`
* `dyn Trait + Send + 'static`
* `dyn Trait +`
* `dyn 'static + Trait`。
* `dyn (Trait)`

r[type.trait-object.syntax-edition2021]
> [!EDITION-2021]
> 在 2021 版本之前，`dyn` 关键字可以省略。

r[type.trait-object.syntax-edition2018]
> [!EDITION-2018]
> 在 2015 版本中，如果 trait 对象的第一个边界是一个以 `::` 开头的路径，则 `dyn` 将被视为路径的一部分。可以通过将第一个路径放在括号中来规避此问题。因此，如果你想要一个带有 trait `::your_module::Trait` 的 trait 对象，应该写作 `dyn (::your_module::Trait)`。
>
> 从 2018 版本开始，`dyn` 是一个真正的关键字，不允许出现在路径中，因此括号不是必需的。

r[type.trait-object.alias]
如果两个 trait 对象类型的基础 trait 互为别名，且自动 trait 集合相同，生命周期边界也相同，则这两个类型互为别名。例如，`dyn Trait + Send + UnwindSafe` 与 `dyn Trait + UnwindSafe + Send` 相同。

r[type.trait-object.unsized]
由于该值具体是哪个类型是不透明的，trait 对象是[动态大小类型]。与所有 <abbr title="动态大小类型">DST</abbr> 一样，trait 对象通过某种指针类型使用；例如 `&dyn SomeTrait` 或 `Box<dyn SomeTrait>`。指向 trait 对象的指针的每个实例包括：

 - 一个指向实现了 `SomeTrait` 的类型 `T` 的实例的指针
 - 一个*虚方法表*，通常简称为 *vtable*，对于 `SomeTrait` 及其 `T` 实现的[超 trait][supertraits] 的每个方法，包含一个指向 `T` 的实现（即函数指针）的指针。

trait 对象的目的是允许方法的"晚期绑定"。在 trait 对象上调用方法会导致运行时的虚派发：即从 trait 对象的 vtable 中加载函数指针并间接调用。每个 vtable 条目对应的实际实现可能因对象而异。

一个 trait 对象的示例：

```rust
trait Printable {
    fn stringify(&self) -> String;
}

impl Printable for i32 {
    fn stringify(&self) -> String { self.to_string() }
}

fn print(a: Box<dyn Printable>) {
    println!("{}", a.stringify());
}

fn main() {
    print(Box::new(10) as Box<dyn Printable>);
}
```

在此示例中，trait `Printable` 在 `print` 的类型签名和 `main` 中的类型转换表达式中均以 trait 对象的形式出现。

r[type.trait-object.lifetime-bounds]
## Trait 对象生命周期边界

由于 trait 对象可以包含引用，这些引用的生命周期需要作为 trait 对象的一部分来表达。此生命周期写作 `Trait + 'a`。有一些[默认规则][defaults]允许此生命周期通常被推断为一个合理的选择。

[auto traits]: ../special-types-and-traits.md#auto-traits
[defaults]: ../lifetime-elision.md#default-trait-object-lifetimes
[dyn compatible]: ../items/traits.md#dyn-compatibility
[动态大小类型]: ../dynamically-sized-types.md
[supertraits]: ../items/traits.md#supertraits
