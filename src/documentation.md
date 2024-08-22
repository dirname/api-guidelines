# 文档

<a id="c-crate-doc"></a>
## crate 级别的文档应详尽并包含示例 (C-CRATE-DOC)

请参阅 [RFC 1687]。

[RFC 1687]: https://github.com/rust-lang/rfcs/pull/1687

<a id="c-example"></a>
## 所有条目都应有一个 rustdoc 示例 (C-EXAMPLE)

每个公共模块、trait、struct、enum、函数、方法、宏和类型定义都应有一个示例来展示其功能。

该指南应在合理的范围内应用。

链接到另一个条目的相关示例可能是足够的。例如，如果只有一个函数使用了某个特定类型，那么在该函数或类型上编写一个示例并从另一个条目中链接到它可能是合适的。

示例的目的并不总是为了展示*如何使用*该条目。读者可以预期能够理解如何调用函数、匹配枚举以及其他基础任务。相反，示例通常是为了展示*为什么有人会想要使用*该条目。

```rust
// 这是一个使用 clone() 的糟糕示例。它机械地展示了*如何*调用 clone()，但没有展示*为什么*有人会需要这样做。
fn main() {
    let hello = "hello";

    hello.clone();
}
```

<a id="c-question-mark"></a>
## 示例应使用 `?`，而不是 `try!`，也不是 `unwrap` (C-QUESTION-MARK)

无论喜不喜欢，用户通常会逐字逐句地复制示例代码。解包错误（unwrap）应是用户需要自行做出的有意识决策。

编写可能失败的示例代码的一个常见结构如下所示。以 `#` 开头的行在构建示例时由 `cargo test` 编译，但不会出现在用户可见的 rustdoc 中。

```
/// ```rust
/// # use std::error::Error;
/// #
/// # fn main() -> Result<(), Box<dyn Error>> {
/// your;
/// example?;
/// code;
/// #
/// #     Ok(())
/// # }
/// ```
```

<a id="c-failure"></a>
## 函数文档应包含错误、panic 和安全性考虑 (C-FAILURE)

错误情况应在“Errors”部分中记录。这也适用于 trait 方法——对于那些允许或预期返回错误的实现，应该在“Errors”部分中进行记录。

例如，在标准库中，某些 [`std::io::Read::read`] trait 方法的实现可能会返回错误。

[`std::io::Read::read`]: https://doc.rust-lang.org/std/io/trait.Read.html#tymethod.read

```
/// 从该源中读取一些字节到指定的缓冲区中，并返回读取的字节数。
///
/// ... 更多信息 ...
///
/// # Errors
///
/// 如果此函数遇到任何形式的 I/O 或其他错误，将返回一个错误变体。如果返回错误，则必须确保没有字节被读取。
```

panic 情况应在“Panics”部分中记录。这也适用于 trait 方法——对于那些允许或预期会 panic 的实现，应在“Panics”部分中进行记录。

在标准库中，[`Vec::insert`] 方法可能会 panic。

[`Vec::insert`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.insert

```
/// 在向量中的 `index` 位置插入一个元素，并将其后的所有元素向右移动。
///
/// # Panics
///
/// 如果 `index` 超出范围，将会 panic。
```

没有必要记录所有可能的 panic 情况，尤其是在 panic 发生在调用者提供的逻辑中。例如，记录以下代码中的 `Display` panic 似乎是多余的。但在不确定的情况下，宁可多记录 panic 情况。

```rust
/// # Panics
///
/// 如果 `T` 的 `Display` 实现 panic，该函数将 panic。
pub fn print<T: Display>(t: T) {
    println!("{}", t.to_string());
}
```

不安全函数应在“Safety”部分中记录，解释调用者在正确使用该函数时需要遵守的所有不变性。

不安全的 [`std::ptr::read`] 要求调用者遵守以下规则。

[`std::ptr::read`]: https://doc.rust-lang.org/std/ptr/fn.read.html

```
/// 从 `src` 读取值，而不移动它。这将保持 `src` 中的内存不变。
///
/// # Safety
///
/// 除了接受一个原始指针外，这个函数是不安全的，因为它在语义上将值从 `src` 中移出，而不阻止 `src` 的进一步使用。
/// 如果 `T` 不是 `Copy`，则必须确保在数据被再次覆盖之前（例如使用 `write`、`zero_memory` 或 `copy_memory`），不要使用 `src` 中的值。请注意，`*src = foo` 也算作使用，因为它将尝试删除之前在 `*src` 处的值。
///
/// 指针必须对齐；如果不是这种情况，请使用 `read_unaligned`。
```

<a id="c-link"></a>
## 文章中应包含到相关内容的超链接 (C-LINK)

常规链接可以使用通常的 Markdown 语法 `[text](url)` 内联添加。可以通过使用 ``[`text`]`` 标记的方式添加到其他类型的链接，然后在文档字符串末尾用 ``[`text`]: <target>`` 添加链接目标，其中 `<target>` 如下所述。

指向同一类型中的方法的链接目标通常如下所示：

```md
[`serialize_struct`]: #method.serialize_struct
```

指向其他类型的链接目标通常如下所示：

```md
[`Deserialize`]: trait.Deserialize.html
```

链接目标也可以指向父模块或子模块：

```md
[`Value`]: ../enum.Value.html
[`DeserializeOwned`]: de/trait.DeserializeOwned.html
```

此指南由 RFC 1574 正式推荐，标题为 ["Link all the things"]。

["Link all the things"]: https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md#link-all-the-things

<a id="c-metadata"></a>
## Cargo.toml 应包含所有常见的元数据 (C-METADATA)

`Cargo.toml` 的 `[package]` 部分应包括以下值：

- `authors`
- `description`
- `license`
- `repository`
- `keywords`
- `categories`

此外，还有两个可选的元数据字段：

- `documentation`
- `homepage`

默认情况下，*crates.io* 会链接到 [*docs.rs*] 上的 crate 文档。只有当文档托管在 *docs.rs* 以外的地方时，才需要设置 `documentation` 元数据，例如因为 crate 链接到 *docs.rs* 构建环境中不可用的共享库。

[*docs.rs*]: https://docs.rs

只有当 crate 有一个独立于源代码仓库或 API 文档的独特网站时，才应设置 `homepage` 元数据。不要让 `homepage` 与 `documentation` 或 `repository` 值重复。例如，serde 将 `homepage` 设置为 *https://serde.rs*，这是一个专用网站。

<a id="c-relnotes"></a>
## 发布说明应记录所有重要更改 (C-RELNOTES)

crate 的用户可以阅读发布说明以找到每个发布版本中的更改摘要。发布说明的链接或发布说明本身应包含在 crate 级文档和/或 Cargo.toml 中链接的仓库中。

发布说明中应清楚标识破坏性更改（根据 [RFC 1105] 定义）。

如果使用 Git 来跟踪 crate 的源代码，则发布到 *crates.io* 的每个版本都应有一个对应的标签，标识已发布的提交。对于非 Git 的版本控制工具，也应采用类似的流程。

```bash
# 标记当前提交
GIT_COMMITTER_DATE=$(git log -n1 --pretty=%aD) git tag -a -m "Release 0.3.0" 0.3.0
git push --tags
```

推荐使用带注释的标签，因为如果存在带注释的标签，有些 Git 命令会忽略未注释的标签。

[RFC 1105]: https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md

### 示例

- [Serde 1.0.0 发布说明](https://github.com/serde-rs/serde/releases/tag/v1.0.0)
- [Serde 0.9.8 发布说明](https://github.com/serde-rs/serde/releases/tag/v0.9.8)
- [Serde 0.9.0 发布说明](https://github.com/serde-rs/serde/releases/tag/v0.9.0)
- [Diesel 变更日志](https://github.com/diesel-rs/diesel/blob/master/CHANGELOG.md)

<a id="c-hidden"></a>
## Rustdoc 不应显示无用的实现细节 (C-HIDDEN)

Rustdoc 应该包含用户完整使用 crate 所需的所有内容，但不应包含更多内容