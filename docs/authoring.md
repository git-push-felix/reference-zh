# 中文翻译编写规范

本文档为中文翻译的编写者和审阅者提供规范指南。

## 必须保留的内容

以下内容**绝对不能翻译或修改**，必须与英文原文保持完全一致：

- `r[...]` 标注（预处理器指令，例如 `r[lex.token.literal]`）
- 代码块——完整的围栏标记、内容、语言标签、rustdoc 属性
- `[!NOTE]`、`[!WARNING]`、`[!EDITION-2024]`、`[!EXAMPLE]`——警示框类型
- 文件底部参考链接定义的 key 和 URL——例如 `[path]: paths.md`（绝不能写成 `[路径]: paths.md`）

## 必须翻译的内容

- 正文段落、列表文本、表格内容、标题文字
- Rust 代码块中的行注释：`// This is hidden.` → `// 这是隐藏行。`

## 链接格式

这是构建警告的最常见来源。英文中使用快捷链接时，文本即 key：`[identifiers]` 对应 `[identifiers]: some.md`。

翻译时**必须**使用双括号格式：

```
英文：   [identifiers]          → 快捷链接，key = "identifiers"
中文：   [标识符][identifiers]  → 带标签的链接，显示 = 标识符，key = identifiers
错误：   [标识符]               → 查找中文 key "标识符"——不存在对应的定义
```

如果链接文本本身就是英文且与 key 相同，则无需修改：`[GitHub issues]` 保持原样即可。

## 锚点与标题

- 参考链接 URL 使用英文锚点：`#integer-literal-expressions`，绝不能使用 `#整数字面量`
- 当翻译后的标题是内部链接的目标时（如 `[Inhabited](#inhabited)`），添加显式锚点：`### 有居民的（Inhabited） {#inhabited}`
- 跨文件链接使用与英文原文相同的英文锚点

## Markdown 格式

- **LF 换行符**（`\n`）。CRLF 会导致 mdbook-spec 内部错误
- **UTF-8 无 BOM**。BOM 会破坏文件首行的 `r[...]` 标注
- **CJK 斜体使用星号**：用 `*中文*`，不要用 `_中文_`。下划线斜体依赖词边界，中文没有空格分隔，因此无法渲染
- 文件必须以换行符结尾
- 优先使用参考链接；参考链接定义置于文件底部，按键排序

## 术语

- 中文 Rust 术语以 [Rust 程序设计语言（中文版）](https://github.com/KaiserY/trpl-zh-cn) 为主要参考
- Rust 关键字在正文中保留英文：`trait`、`crate`、`unsafe`、`panic`
- 翻译力求自然——不要逐字翻译，将英文句式重构为符合中文习惯的表达
- 对于尚无公认翻译的术语，在自创翻译之前，先在互联网上广泛搜索（Rust 生态、其他语言的英译中翻译、计算机科学标准术语）

## 版本差异

正文描述最新的稳定版本。当不同版本（edition）之间存在差异时，使用单独的版本块标注，例如：

```markdown
r[foo.bar.edition2021]
> [!EDITION-2021]
> 描述 2021 版本中的变化。
```

## 目标平台

本参考手册不记录存在哪些目标平台或特定目标平台的属性。在语言要求时才可提及*平台*或*目标属性*，但应仅限于说明语言规则所需的最小范围。例如：

- 说明 `target_os` 等条件编译键存在，但不规定其具体值
- 说明 `windows_subsystem` 属性仅适用于 Windows 平台

## 增量翻译

当上游英文变更时，只翻译差异部分。尽量避免重译未变更的内容，但如果差异过大无法逐句定位中文对应位置，则翻译整个段落。如果文件超过一半已变更，考虑重译整个文件。

翻译完成后，使用 `SPEC_DENY_WARNINGS=1` 编译验证。按文件或逻辑批次单独提交。

## 格式检查

```powershell
# CI 构建模式（将警告视为错误）
$env:SPEC_DENY_WARNINGS="1"; mdbook build

# 样式检查
cargo xtask style-check

# 链接验证
cargo xtask linkcheck

# 内联代码测试
mdbook test
```
