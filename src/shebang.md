r[shebang]
# Shebang

r[shebang.intro]
*[Shebang]* 是一个可选行，通常用于类 Unix 系统中，用来指定执行该文件的解释器。

> [!EXAMPLE]
> <!-- ignore: tests don't like shebang -->
> ```rust,ignore
> #!/usr/bin/env rustx
>
> fn main() {
>     println!("Hello!");
> }
> ```

r[shebang.syntax]
```grammar,lexer
@root SHEBANG ->
    `#!` !((WHITESPACE | LINE_COMMENT | BLOCK_COMMENT)* `[`)
    ~LF* (LF | EOF)
```

r[shebang.syntax-description]
Shebang 以字符 `#!` 开头，并延伸到第一个 `U+000A`（LF）处，如果没有 LF 则延伸到 EOF。如果 `#!` 字符后面跟随 `[`（忽略中间的[注释][comments]或[空白字符][whitespace]），则该行不被视为 shebang（以避免与[内部属性][inner attribute]产生歧义）。

r[shebang.position]
Shebang 可以紧接在文件开头出现，也可以出现在可选的[字节顺序标记][byte order mark]之后。

[byte order mark]: https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
[comments]: comments.md
[inner attribute]: attributes.md
[shebang]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[whitespace]: whitespace.md
