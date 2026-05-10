r[type.inferred]
# 推断类型

r[type.inferred.syntax]
```grammar,types
InferredType -> `_`
```

r[type.inferred.intro]
推断类型要求编译器在可能的情况下根据可用的周围信息来推断类型。

> [!EXAMPLE]
> 推断类型常用于泛型参数中：
>
> ```rust
> let x: Vec<_> = (0..10).collect();
> ```

r[type.inferred.constraint]
推断类型不能用于项签名中。

<!--
  这里还应写些什么？
  我所知的唯一文档是 https://rustc-dev-guide.rust-lang.org/type-inference.html
  应该在某个地方对类型推断有更广泛的讨论。
-->
