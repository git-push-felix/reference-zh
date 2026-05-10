r[subtype]
# 子类型与型变

r[subtype.intro]
子类型关系是隐式的，可以在类型检查或推断的任何阶段发生。

r[subtype.kinds]
子类型仅限于两种情况：关于生命周期的型变，以及具有高阶生命周期的类型之间的子类型关系。如果我们从类型中抹去生命周期，那么唯一的子类型关系将是基于类型相等的关系。

考虑以下示例：字符串字面量始终具有 `'static` 生命周期。尽管如此，我们可以将 `s` 赋值给 `t`：

```rust
fn bar<'a>() {
    let s: &'static str = "hi";
    let t: &'a str = s;
}
```

由于 `'static` 比生命周期参数 `'a` 存活得更久，`&'static str` 是 `&'a str` 的子类型。

r[subtype.higher-ranked]
[高阶][Higher-ranked]&#32;[函数指针][function pointers]和 [trait 对象][trait objects]有另一种子类型关系。它们是由高阶生命周期替换后得到的类型的子类型。一些示例：

```rust
// 这里 'a 被替换为 'static
let subtype: &(for<'a> fn(&'a i32) -> &'a i32) = &((|x| x) as fn(&_) -> &_);
let supertype: &(fn(&'static i32) -> &'static i32) = subtype;

// 这对 trait 对象同样有效
let subtype: &(dyn for<'a> Fn(&'a i32) -> &'a i32) = &|x| x;
let supertype: &(dyn Fn(&'static i32) -> &'static i32) = subtype;

// 我们也可以将一个高阶生命周期替换为另一个
let subtype: &(for<'a, 'b> fn(&'a i32, &'b i32)) = &((|x, y| {}) as fn(&_, &_));
let supertype: &for<'c> fn(&'c i32, &'c i32) = subtype;
```

r[subtyping.variance]
## 型变

r[subtyping.variance.intro]
型变是泛型类型相对于其参数所具有的一种性质。泛型类型在某个参数上的*型变*描述了该参数的子类型关系如何影响该类型的子类型关系。

r[subtyping.variance.covariant]
* 如果 `T` 是 `U` 的子类型意味着 `F<T>` 是 `F<U>` 的子类型，则称 `F<T>` 对 `T` 是*协变*的（子类型"穿透"）

r[subtyping.variance.contravariant]
* 如果 `T` 是 `U` 的子类型意味着 `F<U>` 是 `F<T>` 的子类型，则称 `F<T>` 对 `T` 是*逆变*的

r[subtyping.variance.invariant]
* 否则 `F<T>` 对 `T` 是*不变*的（不能推导出子类型关系）

r[subtyping.variance.builtin-types]
类型的型变按以下规则自动确定：

| 类型                          | `'a` 中的型变  | `T` 中的型变   |
|-------------------------------|----------------|----------------|
| `&'a T`                       | 协变           | 协变           |
| `&'a mut T`                   | 协变           | 不变           |
| `*const T`                    |                | 协变           |
| `*mut T`                      |                | 不变           |
| `[T]` 和 `[T; n]`             |                | 协变           |
| `fn() -> T`                   |                | 协变           |
| `fn(T) -> ()`                 |                | 逆变           |
| `std::cell::UnsafeCell<T>`    |                | 不变           |
| `std::marker::PhantomData<T>` |                | 协变           |
| `dyn Trait<T> + 'a`           | 协变           | 不变           |

r[subtyping.variance.user-composite-types]
其他 `struct`、`enum` 和 `union` 类型的型变通过其字段类型的型变来决定。如果参数被用在具有不同型变的位置上，则该参数是不变的。例如，以下结构体在 `'a` 和 `T` 上是协变的，在 `'b`、`'c` 和 `U` 上是不变的。

```rust
use std::cell::UnsafeCell;
struct Variance<'a, 'b, 'c, T, U: 'a> {
    x: &'a U,               // 这使得 `Variance` 在 'a 上协变，并且会使
                            // 它在 U 上协变，但 U 在后面被使用了
    y: *const T,            // 在 T 上协变
    z: UnsafeCell<&'b f64>, // 在 'b 上不变
    w: *mut U,              // 在 U 上不变，使得整个结构体不变

    f: fn(&'c ()) -> &'c () // 同时协变和逆变，使得 'c 在结构体中不变
}
```

r[subtyping.variance.builtin-composite-types]
当在 `struct`、`enum` 或 `union` 之外使用时，参数的型变在各个位置独立检查。

```rust
# use std::cell::UnsafeCell;
fn generic_tuple<'short, 'long: 'short>(
    // 'long 在元组中同时被用在协变和不变位置。
    x: (&'long u32, UnsafeCell<&'long u32>),
) {
    // 由于这些位置的型变是独立计算的，
    // 我们可以在协变位置自由缩短 'long。
    let _: (&'short u32, UnsafeCell<&'long u32>) = x;
}

fn takes_fn_ptr<'short, 'middle: 'short>(
    // 'middle 同时被用在协变和逆变位置。
    f: fn(&'middle ()) -> &'middle (),
) {
    // 由于这些位置的型变是独立计算的，
    // 我们可以在协变位置自由缩短 'middle，
    // 并在逆变位置扩展它。
    let _: fn(&'static ()) -> &'short () = f;
}
```

[函数指针]: types/function-pointer.md
[高阶]: ../nomicon/hrtb.html
[trait 对象]: types/trait-object.md
