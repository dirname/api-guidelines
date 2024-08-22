# Rust API 指南

这是关于如何设计和展示 Rust 编程语言的 API 的一系列推荐。这些指南主要由 Rust 库团队撰写，基于构建 Rust 标准库和 Rust 生态系统中其他 crate 的经验。

这些只是指南，其中一些更为严格，而其他的则比较模糊且仍在开发中。Rust crate 作者应将它们视为开发惯用和可互操作的 Rust 库时需要考虑的一系列重要因素，可以根据需要使用它们。这些指南不应被视为 crate 作者必须遵循的强制性规定，尽管他们可能会发现，与这些指南高度兼容的 crate 比那些不兼容的 crate 更好地集成到现有的 crate 生态系统中。

本书分为两部分：适合在 crate 评审期间快速浏览的所有单独指南的简明 [清单]；以及包含详细指南解释的专题章节。

如果你有兴趣为 API 指南做出贡献，请查阅 [contributing.md] 并加入我们的 [Gitter 频道]。

[checklist]: checklist.html
[contributing.md]: https://github.com/rust-lang/api-guidelines/blob/master/CONTRIBUTING.md
[Gitter channel]: https://gitter.im/rust-impl-period/WG-libs-guidelines