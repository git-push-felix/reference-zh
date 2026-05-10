r[items.union]
# 联合体

r[items.union.syntax]
```grammar,items
Union ->
    `union` IDENTIFIER GenericParams? WhereClause? `{` StructFields? `}`
```

r[items.union.intro]
联合体声明使用与结构体声明相同的语法，只是用 `union` 代替了 `struct`。

r[items.union.namespace]
联合体声明在其所在模块或块的[类型命名空间][type namespace]中定义给定的名称。

```rust
#[repr(C)]
union MyUnion {
    f1: u32,
    f2: f32,
}
```

r[items.union.common-storage]
联合体的关键属性是联合体的所有字段共享公共存储。因此，对联合体一个字段的写入会覆盖其其他字段，并且联合体的大小由其最大字段的大小决定。

r[items.union.field-restrictions]
联合体字段类型限制为以下类型的子集：

r[items.union.field-copy]
- `Copy` 类型

r[items.union.field-references]
- 引用（任意 `T` 的 `&T` 和 `&mut T`）

r[items.union.field-manually-drop]
- `ManuallyDrop<T>`（任意 `T`）

r[items.union.field-tuple]
- 仅包含允许的联合体字段类型的元组和数组

r[items.union.drop]
此限制特别确保了联合体字段永远不需要被释放。和结构体与枚举一样，可以为联合体 `impl Drop` 来手动定义其被释放时的行为。

r[items.union.fieldless]
没有任何字段的联合体不被编译器接受，但可以被宏接受。

r[items.union.init]
## 联合体的初始化 {#initialization-of-a-union}

r[items.union.init.intro]
联合体类型的值可以使用与结构体类型相同的语法创建，但必须指定恰好一个字段：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
let u = MyUnion { f1: 1 };
```

r[items.union.init.result]
上面的表达式创建了一个 `MyUnion` 类型的值，并使用字段 `f1` 初始化存储。可以使用与结构体字段相同的语法访问联合体：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
# let u = MyUnion { f1: 1 };
let f = unsafe { u.f1 };
```

r[items.union.fields]
## 读写联合体字段 {#reading-and-writing-union-fields}

r[items.union.fields.intro]
联合体没有"活动字段"的概念。相反，每次联合体访问只是将存储解释为用于访问的字段的类型。

r[items.union.fields.read]
读取联合体字段会以该字段的类型读取联合体的位。

r[items.union.fields.offset]
字段可能有非零偏移（除非使用了 [C 表示法][the C representation]）；在这种情况下，会读取从字段偏移处开始的位。

r[items.union.fields.validity]
程序员有责任确保数据在该字段的类型下是有效的。未能做到这一点会导致[未定义行为][undefined behavior]。例如，从[布尔类型][boolean type]的字段中读取值 `3` 是未定义行为。实际上，使用 [C 表示法][the C representation]对联合体进行写入然后读取，类似于从用于写入的类型到用于读取的类型的 [`transmute`]。

r[items.union.fields.read-safety]
因此，所有对联合体字段的读取都必须放在 `unsafe` 块中：

```rust
# union MyUnion { f1: u32, f2: f32 }
# let u = MyUnion { f1: 1 };
#
unsafe {
    let f = u.f1;
}
```

通常，使用联合体的代码会围绕不安全的联合体字段访问提供安全的包装器。

r[items.union.fields.write-safety]
相比之下，对联合体字段的写入是安全的，因为它们只是覆盖任意数据，但不会导致未定义行为。（请注意，联合体字段类型永远不会有 drop 胶水代码，因此联合体字段写入永远不会隐式地释放任何东西。）

r[items.union.pattern]
## 联合体上的模式匹配 {#pattern-matching-on-unions}

r[items.union.pattern.intro]
访问联合体字段的另一种方式是使用模式匹配。

r[items.union.pattern.one-field]
联合体字段上的模式匹配使用与结构体模式相同的语法，只是模式必须指定恰好一个字段。

r[items.union.pattern.safety]
由于模式匹配类似于用特定字段读取联合体，因此也必须放在 `unsafe` 块中。

```rust
# union MyUnion { f1: u32, f2: f32 }
#
fn f(u: MyUnion) {
    unsafe {
        match u {
            MyUnion { f1: 10 } => { println!("ten"); }
            MyUnion { f2 } => { println!("{}", f2); }
        }
    }
}
```

r[items.union.pattern.subpattern]
模式匹配可以将联合体作为更大结构的字段进行匹配。特别是，当通过 FFI 使用 Rust 联合体实现 C 标记联合体时，这允许同时匹配标记和相应字段：

```rust
#[repr(u32)]
enum Tag { I, F }

#[repr(C)]
union U {
    i: i32,
    f: f32,
}

#[repr(C)]
struct Value {
    tag: Tag,
    u: U,
}

fn is_zero(v: Value) -> bool {
    unsafe {
        match v {
            Value { tag: Tag::I, u: U { i: 0 } } => true,
            Value { tag: Tag::F, u: U { f: num } } if num == 0.0 => true,
            _ => false,
        }
    }
}
```

r[items.union.ref]
## 联合体字段的引用 {#references-to-union-fields}

r[items.union.ref.intro]
由于联合体字段共享公共存储，获得联合体一个字段的写入访问权限可能会获得其所有其余字段的写入访问权限。

r[items.union.ref.borrow]
借用检查规则必须进行调整以考虑这一事实。因此，如果联合体的一个字段被借用了，其所有其余字段也会在同一生命周期内被借用。

```rust,compile_fail
# union MyUnion { f1: u32, f2: f32 }
// 错误：不能多次可变借用 `u`（通过 `u.f2`）
fn test() {
    let mut u = MyUnion { f1: 1 };
    unsafe {
        let b1 = &mut u.f1;
//                    ---- 第一次可变借用发生在这里（通过 `u.f1`）
        let b2 = &mut u.f2;
//                    ^^^^ 第二次可变借用发生在这里（通过 `u.f2`）
        *b1 = 5;
    }
//  - 第一次借用在此结束
    assert_eq!(unsafe { u.f1 }, 5);
}
```

r[items.union.ref.use]
如你所见，在许多方面（除了布局、安全性和所有权外）联合体的行为与结构体完全相同，这很大程度上是因为它们从结构体继承了语法形状。对于 Rust 语言的许多未提及的方面也是如此（如隐私、名称解析、类型推断、泛型、trait 实现、固有实现、一致性、模式检查等等）。

[`transmute`]: std::mem::transmute
[boolean type]: ../types/boolean.md
[the C representation]: ../type-layout.md#reprc-unions
[type namespace]: ../names/namespaces.md
[undefined behavior]: ../behavior-considered-undefined.md
