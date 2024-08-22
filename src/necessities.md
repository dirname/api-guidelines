# 必要性

<a id="c-stable"></a>
## 稳定版 Crate 的公共依赖必须稳定 (C-STABLE)

一个 Crate 如果要被标记为稳定版（>=1.0.0），则它所有的公共依赖也必须是稳定的。

公共依赖是指在当前 Crate 的公共 API 中使用的其他 Crate 的类型。

```rust
pub fn do_my_thing(arg: other_crate::TheirThing) { /* ... */ }
```

如果一个 Crate 包含了上述函数，那么 `other_crate` 也必须是稳定的，否则这个 Crate 无法被标记为稳定版。

要小心，因为公共依赖可能会在意想不到的地方出现。

```rust
pub struct Error {
    private: ErrorImpl,
}

enum ErrorImpl {
    Io(io::Error),
    // 即使 other_crate 不是稳定的，这里也没问题，
    // 因为 ErrorImpl 是私有的。
    Dep(other_crate::Error),
}

// 哦不！这使得 other_crate 出现在了当前 Crate 的公共 API 中。
impl From<other_crate::Error> for Error {
    fn from(err: other_crate::Error) -> Self {
        Error { private: ErrorImpl::Dep(err) }
    }
}
```

<a id="c-permissive"></a>
## Crate 及其依赖必须具有宽松的许可证 (C-PERMISSIVE)

Rust 项目生成的软件是双许可证的，既可以在 [MIT] 许可证下发布，也可以在 [Apache 2.0] 许可证下发布。为了与 Rust 生态系统保持最大的兼容性，建议 Crate 也采用相同的双许可证方式，具体方法如下所述。其他许可选项在下面描述。

这些 API 指南并不详细解释 Rust 的许可证，但在 [Rust FAQ] 中有一些相关说明。这里的指南主要关注与 Rust 的互操作性问题，并未涵盖所有的许可选项。

[MIT]: https://github.com/rust-lang/rust/blob/master/LICENSE-MIT
[Apache 2.0]: https://github.com/rust-lang/rust/blob/master/LICENSE-APACHE
[Rust FAQ]: https://github.com/dtolnay/rust-faq#why-a-dual-mitasl2-license

要将 Rust 许可证应用于你的项目，可以在 `Cargo.toml` 中定义 `license` 字段，如下所示：

```toml
[package]
name = "..."
version = "..."
authors = ["..."]
license = "MIT OR Apache-2.0"
```

然后在项目根目录下添加 `LICENSE-APACHE` 和 `LICENSE-MIT` 文件，文件内容应为许可证的文本（可以从 choosealicense.com 获取，例如 [Apache-2.0](https://choosealicense.com/licenses/apache-2.0/) 和 [MIT](https://choosealicense.com/licenses/mit/)）。

并在你的 README.md 的结尾添加：

```
## License

Licensed under either of

 * Apache License, Version 2.0
   ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license
   ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
```

除了双 MIT/Apache-2.0 许可证外，Rust Crate 作者使用的另一种常见的许可方式是采用单一宽松许可证，例如 MIT 或 BSD。这种许可方式与 Rust 的许可完全兼容，因为它遵循了 Rust MIT 许可证的最低限制。

不建议仅选择 Apache 许可证的 Crate，因为虽然 Apache 许可证是一种宽松的许可证，但它比 MIT 和 BSD 许可证附加了更多限制，可能会在某些场景下阻碍或阻止它们的使用，因此仅有 Apache 许可证的软件在某些情况下无法使用 Rust 的大部分运行时栈。

Crate 的依赖许可证可能会影响该 Crate 本身的分发限制，因此具有宽松许可证的 Crate 通常应该只依赖于具有宽松许可证的其他 Crate。