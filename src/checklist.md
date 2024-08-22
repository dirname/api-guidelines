# Rust API 指南检查清单

<!-- 在编写新指南之前，请阅读 CONTRIBUTING.md -->

- **命名** *(crate 符合 Rust 命名约定)*
  - [ ] 命名符合 RFC 430 ([C-CASE])
  - [ ] 临时转换遵循 `as_`, `to_`, `into_` 约定 ([C-CONV])
  - [ ] Getter 名称遵循 Rust 约定 ([C-GETTER])
  - [ ] 集合上的方法生成迭代器遵循 `iter`, `iter_mut`, `into_iter` ([C-ITER])
  - [ ] 迭代器类型名称与生成它们的方法匹配 ([C-ITER-TY])
  - [ ] 功能名称没有占位符词语 ([C-FEATURE])
  - [ ] 名称使用一致的词序 ([C-WORD-ORDER])
- **互操作性** *(crate 与其他库功能良好交互)*
  - [ ] 类型积极实现常见特性 ([C-COMMON-TRAITS])
    - `Copy`, `Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`, `Hash`, `Debug`,
      `Display`, `Default`
  - [ ] 转换使用标准特性 `From`, `AsRef`, `AsMut` ([C-CONV-TRAITS])
  - [ ] 集合实现 `FromIterator` 和 `Extend` ([C-COLLECT])
  - [ ] 数据结构实现 Serde 的 `Serialize`, `Deserialize` ([C-SERDE])
  - [ ] 类型在可能的情况下是 `Send` 和 `Sync` ([C-SEND-SYNC])
  - [ ] 错误类型有意义且行为良好 ([C-GOOD-ERR])
  - [ ] 二进制数类型提供 `Hex`, `Octal`, `Binary` 格式化 ([C-NUM-FMT])
  - [ ] 通用读/写函数按值接收 `R: Read` 和 `W: Write` ([C-RW-VALUE])
- **宏** *(crate 提供行为良好的宏)*
  - [ ] 输入语法唤起输出 ([C-EVOCATIVE])
  - [ ] 宏与属性良好组合 ([C-MACRO-ATTR])
  - [ ] 项目宏在允许项目的任何地方工作 ([C-ANYWHERE])
  - [ ] 项目宏支持可见性说明符 ([C-MACRO-VIS])
  - [ ] 类型片段灵活 ([C-MACRO-TY])
- **文档** *(crate 文档丰富)*
  - [ ] crate 级别文档详尽并包含示例 ([C-CRATE-DOC])
  - [ ] 所有项目都有 rustdoc 示例 ([C-EXAMPLE])
  - [ ] 示例使用 `?`，而不是 `try!` 或 `unwrap` ([C-QUESTION-MARK])
  - [ ] 函数文档包含错误、恐慌和安全性考虑 ([C-FAILURE])
  - [ ] 文章包含相关内容的超链接 ([C-LINK])
  - [ ] Cargo.toml 包含所有常见元数据 ([C-METADATA])
    - 作者、描述、许可证、主页、文档、仓库、关键词、类别
  - [ ] 发行说明记录所有重大变化 ([C-RELNOTES])
  - [ ] Rustdoc 不显示无用的实现细节 ([C-HIDDEN])
- **可预测性** *(crate 使代码易读且行为如表面所示)*
  - [ ] 智能指针不添加固有方法 ([C-SMART-PTR])
  - [ ] 转换存在于所涉及的最具体类型上 ([C-CONV-SPECIFIC])
  - [ ] 具有明确接收者的函数是方法 ([C-METHOD])
  - [ ] 函数不接收输出参数 ([C-NO-OUT])
  - [ ] 操作符重载不令人惊讶 ([C-OVERLOAD])
  - [ ] 只有智能指针实现 `Deref` 和 `DerefMut` ([C-DEREF])
  - [ ] 构造函数是静态的固有方法 ([C-CTOR])
- **灵活性** *(crate 支持多样的现实使用场景)*
  - [ ] 函数暴露中间结果以避免重复工作 ([C-INTERMEDIATE])
  - [ ] 调用者决定在哪里复制和放置数据 ([C-CALLER-CONTROL])
  - [ ] 函数通过使用泛型最小化对参数的假设 ([C-GENERIC])
  - [ ] 如果特性作为特性对象可能有用，则它们是对象安全的 ([C-OBJECT])
- **类型安全** *(crate 有效利用类型系统)*
  - [ ] 新类型提供静态区分 ([C-NEWTYPE])
  - [ ] 参数通过类型而不是 `bool` 或 `Option` 传达意义 ([C-CUSTOM-TYPE])
  - [ ] 一组标志的类型是 `bitflags`，而不是枚举 ([C-BITFLAG])
  - [ ] 构建器使复杂值的构建成为可能 ([C-BUILDER])
- **可靠性** *(crate 不太可能做错事)*
  - [ ] 函数验证其参数 ([C-VALIDATE])
  - [ ] 析构函数从不失败 ([C-DTOR-FAIL])
  - [ ] 可能阻塞的析构函数有替代方案 ([C-DTOR-BLOCK])
- **可调试性** *(crate 便于调试)*
  - [ ] 所有公共类型实现 `Debug` ([C-DEBUG])
  - [ ] `Debug` 表示永不为空 ([C-DEBUG-NONEMPTY])
- **未来保障** *(crate 可以在不破坏用户代码的情况下改进)*
  - [ ] 密封特性防止下游实现 ([C-SEALED])
  - [ ] 结构体有私有字段 ([C-STRUCT-PRIVATE])
  - [ ] 新类型封装实现细节 ([C-NEWTYPE-HIDE])
  - [ ] 数据结构不重复派生特性边界 ([C-STRUCT-BOUNDS])
- **必要性** *(对相关人员而言，它们非常重要)*
  - [ ] 稳定 crate 的公共依赖是稳定的 ([C-STABLE])
  - [ ] crate 及其依赖项具有宽松的许可证 ([C-PERMISSIVE])


[C-CASE]: naming.html#c-case
[C-CONV]: naming.html#c-conv
[C-GETTER]: naming.html#c-getter
[C-ITER]: naming.html#c-iter
[C-ITER-TY]: naming.html#c-iter-ty
[C-FEATURE]: naming.html#c-feature
[C-WORD-ORDER]: naming.html#c-word-order

[C-COMMON-TRAITS]: interoperability.html#c-common-traits
[C-CONV-TRAITS]: interoperability.html#c-conv-traits
[C-COLLECT]: interoperability.html#c-collect
[C-SERDE]: interoperability.html#c-serde
[C-SEND-SYNC]: interoperability.html#c-send-sync
[C-GOOD-ERR]: interoperability.html#c-good-err
[C-NUM-FMT]: interoperability.html#c-num-fmt
[C-RW-VALUE]: interoperability.html#c-rw-value

[C-EVOCATIVE]: macros.html#c-evocative
[C-MACRO-ATTR]: macros.html#c-macro-attr
[C-ANYWHERE]: macros.html#c-anywhere
[C-MACRO-VIS]: macros.html#c-macro-vis
[C-MACRO-TY]: macros.html#c-macro-ty

[C-CRATE-DOC]: documentation.html#c-crate-doc
[C-EXAMPLE]: documentation.html#c-example
[C-QUESTION-MARK]: documentation.html#c-question-mark
[C-FAILURE]: documentation.html#c-failure
[C-LINK]: documentation.html#c-link
[C-METADATA]: documentation.html#c-metadata
[C-HTML-ROOT]: documentation.html#c-html-root
[C-RELNOTES]: documentation.html#c-relnotes
[C-HIDDEN]: documentation.html#c-hidden

[C-SMART-PTR]: predictability.html#c-smart-ptr
[C-CONV-SPECIFIC]: predictability.html#c-conv-specific
[C-METHOD]: predictability.html#c-method
[C-NO-OUT]: predictability.html#c-no-out
[C-OVERLOAD]: predictability.html#c-overload
[C-DEREF]: predictability.html#c-deref
[C-CTOR]: predictability.html#c-ctor

[C-INTERMEDIATE]: flexibility.html#c-intermediate
[C-CALLER-CONTROL]: flexibility.html#c-caller-control
[C-GENERIC]: flexibility.html#c-generic
[C-OBJECT]: flexibility.html#c-object

[C-NEWTYPE]: type-safety.html#c-newtype
[C-CUSTOM-TYPE]: type-safety.html#c-custom-type
[C-BITFLAG]: type-safety.html#c-bitflag
[C-BUILDER]: type-safety.html#c-builder

[C-VALIDATE]: dependability.html#c-validate
[C-DTOR-FAIL]: dependability.html#c-dtor-fail
[C-DTOR-BLOCK]: dependability.html#c-dtor-block

[C-DEBUG]: debuggability.html#c-debug
[C-DEBUG-NONEMPTY]: debuggability.html#c-debug-nonempty

[C-SEALED]: future-proofing.html#c-sealed
[C-STRUCT-PRIVATE]: future-proofing.html#c-struct-private
[C-NEWTYPE-HIDE]: future-proofing.html#c-newtype-hide
[C-STRUCT-BOUNDS]: future-proofing.html#c-struct-bounds

[C-STABLE]: necessities.html#c-stable
[C-PERMISSIVE]: necessities.html#c-permissive