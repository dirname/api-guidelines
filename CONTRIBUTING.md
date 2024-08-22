# 贡献 API 指南

Rust API 指南项目欢迎每个人以建议、bug 报告、pull request 和反馈的形式进行贡献。如果你想帮助我们，本文档将提供一些指导。

如果你在贡献过程中有任何问题，可以通过 GitHub issue 或者我们的 [Gitter 频道] 联系我们，我们会尽力为你提供帮助。

[Gitter 频道]: https://gitter.im/rust-impl-period/WG-libs-guidelines

## 提交新的指南想法

我们一直在寻找高质量 Rust 库中可以学习的经验。如果你发现某个 crate 的 API 某方面做得特别好，且其他 crate 也可以从中受益，欢迎 [发起讨论] 告诉我们。

如果你想提出对现有指南的具体修改建议，也可以 [提交 issue]。

[发起讨论]: https://github.com/rust-lang/api-guidelines/discussions/new  
[提交 issue]: https://github.com/rust-lang/api-guidelines/issues/new

## 为指南撰写内容

指南的内容存放在 [`src`] 目录下的一系列 Markdown 文件中。当你进行更改时，可以使用 [mdBook] 来预览渲染后的内容。

[`src`]: https://github.com/rust-lang/api-guidelines/tree/master/src  
[mdBook]: https://github.com/azerupi/mdBook

```sh
cargo install mdbook

# 在 api-guidelines 目录下运行
mdbook serve
```

`mdbook serve` 命令会将渲染后的 API 指南作为一个网页，在 http://localhost:3000/ 上提供访问。

## 指南的书写规范

我们遵循一些基本的语法规则，以确保指南的检查清单保持一致性和可理解性。

指南应该是关于假设 crate 的 **陈述句**。

```diff
  不是命令句，比如
- "为二进制数字类型实现 Hex、Octal、Binary"
  而是：
+ "二进制数字类型提供 Hex、Octal、Binary 格式化"

  不是义务性句子，比如
- "宏应该与属性很好地组合"
  而是：
+ "宏与属性很好地组合"
```

指南应当有明确的 **主语** 和 **动词**。

```diff
  不是隐含主语，比如
- "包括所有常见的 Cargo.toml 元数据"
  而是：
+ "Cargo.toml 包含所有常见的元数据"

  不是隐含动词，比如
- "用示例进行详细记录"
  而是：
+ "crate 级别的文档详尽并且包含示例"

  不是形而上的句子，比如
- "没有 out 参数"
  而是：
+ "函数不接受 out 参数"
```

指南应该使用 **主动语态**。

```diff
  不是被动语态，比如
- "函数参数已被验证"
  而是：
+ "函数验证它们的参数"
```

## 行为准则

和所有 Rust 相关的空间一样，我们遵守 [Rust 行为准则]。如果有任何升级或管理问题，请联系 Rust 管理团队，邮箱是 rust-mods@rust-lang.org。

[Rust 行为准则]: https://www.rust-lang.org/policies/code-of-conduct