# 未来适应性

<a id="c-sealed"></a>
## 封闭的特征防止下游实现 (C-SEALED)

某些特征（traits）仅应在定义它们的 crate 内部实现。在这种情况下，我们可以通过使用封闭特征模式来保留改变特征的能力，而不引入不兼容更改。

```rust
/// 此特征是封闭的，不能在本 crate 之外为类型实现。
pub trait TheTrait: private::Sealed {
    // 一个或多个用户可调用的方法。
    fn ...();

    // 一个或多个私有方法，不允许用户调用。
    #[doc(hidden)]
    fn ...();
}

// 为某些类型实现。
impl TheTrait for usize {
    /* ... */
}

mod private {
    pub trait Sealed {}

    // 为相同的类型实现，但不为其他类型实现。
    impl Sealed for usize {}
}
```

空的私有 `Sealed` 超特征无法被下游 crate 命名，因此我们可以确保 `Sealed`（因此也包括 `TheTrait`）的实现仅存在于当前 crate 中。我们可以在非破坏性的版本中添加方法到 `TheTrait`，即使这通常是对于未封闭特征的破坏性更改。此外，我们可以更改未公开文档的方法的签名。

请注意，移除公共方法或更改封闭特征中公共方法的签名仍然是破坏性的更改。

为了避免用户尝试实现该特征的挫折，应该在 rustdoc 中记录该特征是封闭的且不应在当前 crate 之外实现。

### 示例

- [`serde_json::value::Index`](https://docs.serde.rs/serde_json/value/trait.Index.html)
- [`byteorder::ByteOrder`](https://docs.rs/byteorder/1.1.0/byteorder/trait.ByteOrder.html)

<a id="c-struct-private"></a>
## 结构体具有私有字段 (C-STRUCT-PRIVATE)

将字段设为公共是一项强大的承诺：它固定了一种表示选择，并且禁止该类型提供任何验证或维持字段内容的不变量，因为客户端可以随意修改它。

公共字段最适用于 C 精神中的 `struct` 类型：组合的、被动的数据结构。否则，请考虑提供 getter/setter 方法并隐藏字段。

<a id="c-newtype-hide"></a>
## 新类型封装实现细节 (C-NEWTYPE-HIDE)

新类型可以用于隐藏表示细节，同时向客户端做出精确的承诺。

例如，考虑一个返回复合迭代器类型的函数 `my_transform`。

```rust
use std::iter::{Enumerate, Skip};

pub fn my_transform<I: Iterator>(input: I) -> Enumerate<Skip<I>> {
    input.skip(3).enumerate()
}
```

我们希望向客户端隐藏该类型，以使客户端对返回类型的视图大致为 `Iterator<Item = (usize, T)>`。我们可以使用新类型模式来实现：

```rust
use std::iter::{Enumerate, Skip};

pub struct MyTransformResult<I>(Enumerate<Skip<I>>);

impl<I: Iterator> Iterator for MyTransformResult<I> {
    type Item = (usize, I::Item);

    fn next(&mut self) -> Option<Self::Item> {
        self.0.next()
    }
}

pub fn my_transform<I: Iterator>(input: I) -> MyTransformResult<I> {
    MyTransformResult(input.skip(3).enumerate())
}
```

除了简化签名之外，这种新类型的使用允许我们向客户端承诺更少。客户端不知道结果迭代器是如何构造或表示的，这意味着表示可以在不破坏客户端代码的情况下更改。

Rust 1.26 还引入了 [`impl Trait`][] 特性，比新类型模式更为简洁，但有一些额外的权衡，特别是在表达上有限制。例如，返回实现 `Debug` 或 `Clone` 或其他迭代器扩展特征组合的迭代器可能会有问题。总之，对于内部 APIs，`impl Trait` 作为返回类型可能很好，甚至可能适用于公共 APIs，但可能并非所有情况下都合适。详情请参见新版指南中的 ["`impl Trait` for returning complex types with ease"][impl-trait-2] 部分。

[`impl Trait`]: https://github.com/rust-lang/rfcs/blob/master/text/1522-conservative-impl-trait.md
[impl-trait-2]: https://rust-lang.github.io/edition-guide/rust-2018/trait-system/impl-trait-for-returning-complex-types-with-ease.html

```rust
pub fn my_transform<I: Iterator>(input: I) -> impl Iterator<Item = (usize, I::Item)> {
    input.skip(3).enumerate()
}
```

<a id="c-struct-bounds"></a>
## 数据结构不重复派生特征边界 (C-STRUCT-BOUNDS)

泛型数据结构不应使用可以派生的或其他不增加语义价值的特征边界。`derive` 属性中的每个特征都会被扩展为一个单独的 `impl` 块，只有泛型参数实现该特征时才适用。

```rust
// 推荐这样写：
#[derive(Clone, Debug, PartialEq)]
struct Good<T> { /* ... */ }

// 而不是这样：
#[derive(Clone, Debug, PartialEq)]
struct Bad<T: Clone + Debug + PartialEq> { /* ... */ }
```

在 `Bad` 上重复派生特征作为边界是不必要的，而且是向后兼容性的隐患。要说明这一点，考虑在上一个示例的结构上派生 `PartialOrd`：

```rust
// 非破坏性更改：
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Good<T> { /* ... */ }

// 破坏性更改：
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Bad<T: Clone + Debug + PartialEq + PartialOrd> { /* ... */ }
```

一般来说，向数据结构添加特征边界是破坏性更改，因为该结构的每个消费者都需要开始满足额外的边界。使用 `derive` 属性从标准库派生更多特征不是破坏性更改。

以下特征不应在数据结构的边界中使用：

- `Clone`
- `PartialEq`
- `PartialOrd`
- `Debug`
- `Display`
- `Default`
- `Error`
- `Serialize`
- `Deserialize`
- `DeserializeOwned`

对于其他不可派生的特征边界，它们严格来说不是结构定义所必需的，例如 `Read` 或 `Write`，存在一个灰色地带。它们可能更好地在定义中表达了类型的预期行为，但也限制了未来的扩展性。在数据结构上包含语义上有用的特征边界比包含可派生的特征边界的问题要小。

### 例外情况

在以下三种情况下，结构体需要特征边界：

1. 数据结构指代特征上的相关类型。
2. 边界是 `?Sized`。
3. 数据结构有一个需要特征边界的 `Drop` 实现。 Rust 当前要求 `Drop` 实现上的所有特征边界也应出现在数据结构上。

### 标准库中的示例

- [`std::borrow::Cow`] 指代 `Borrow` 特征上的相关类型。
- [`std::boxed::Box`] 放弃了隐式的 `Sized` 边界。
- [`std::io::BufWriter`] 在其 `Drop` 实现中需要一个特征边界。

[`std::borrow::Cow`]: https://doc.rust-lang.org/std/borrow/enum.Cow.html
[`std::boxed::Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`std::io::BufWriter`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html