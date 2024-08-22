# 互操作性

<a id="c-common-traits"></a>
## 类型应尽早实现常见的 trait（C-COMMON-TRAITS）

Rust 的 trait 系统不允许存在“孤儿”：大致上，每个 `impl` 必须存在于定义该 trait 的 crate 或实现该类型的 crate 中。因此，定义新类型的 crate 应该尽早实现所有适用的常见 trait。

为了理解这一点，请考虑以下情况：

* Crate `std` 定义了 trait `Display`。
* Crate `url` 定义了类型 `Url`，但没有实现 `Display`。
* Crate `webapp` 从 `std` 和 `url` 中导入了内容。

由于 `webapp` 既不定义 `Display` 也不定义 `Url`，因此它无法为 `Url` 添加 `Display`。 （注意：newtype 模式可以提供一种有效但不便的解决方法。）

最重要的常见 trait 实现来自 `std`：

- [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html)
- [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html)
- [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)
- [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html)
- [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)
- [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html)
- [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html)
- [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html)
- [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html)

请注意，通常类型同时实现 `Default` 和空的 `new` 构造函数是常见且预期的。`new` 是 Rust 中的构造函数惯例，用户希望它存在，所以如果基本构造函数不需要参数，那么即使它与 `default` 功能相同，也应该存在。

<a id="c-conv-traits"></a>
## 转换使用标准的 `From`、`AsRef`、`AsMut` trait（C-CONV-TRAITS）

在合适的情况下，应该实现以下转换 trait：

- [`From`](https://doc.rust-lang.org/std/convert/trait.From.html)
- [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html)
- [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html)
- [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)

以下转换 trait 不应被实现：

- [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html)
- [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html)

这些 trait 基于 `From` 和 `TryFrom` 提供了通用实现。应优先实现这些。

### 标准库中的示例

- `From<u16>` 实现了 `u32`，因为较小的整数总是可以转换为较大的整数。
- `From<u32>` 并未为 `u16` 实现，因为如果整数太大，转换可能无法完成。
- `TryFrom<u32>` 为 `u16` 实现，当整数太大而无法容纳在 `u16` 中时返回错误。
- [`From<Ipv6Addr>`] 实现了 [`IpAddr`]，它是一种可以表示 v4 和 v6 IP 地址的类型。

[`From<Ipv6Addr>`]: https://doc.rust-lang.org/std/net/struct.Ipv6Addr.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html

<a id="c-collect"></a>
## 集合实现 `FromIterator` 和 `Extend`（C-COLLECT）

[`FromIterator`] 和 [`Extend`] 使集合可以方便地与以下迭代器方法一起使用：

[`FromIterator`]: https://doc.rust-lang.org/std/iter/trait.FromIterator.html
[`Extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html

- [`Iterator::collect`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)
- [`Iterator::partition`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.partition)
- [`Iterator::unzip`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.unzip)

`FromIterator` 用于创建一个包含迭代器中项的新集合，而 `Extend` 则用于将迭代器中的项添加到现有集合中。

### 标准库中的示例

- [`Vec<T>`] 实现了 `FromIterator<T>` 和 `Extend<T>`。

[`Vec<T>`]: https://doc.rust-lang.org/std/vec/struct.Vec.html

<a id="c-serde"></a>
## 数据结构实现 Serde 的 `Serialize` 和 `Deserialize`（C-SERDE）

扮演数据结构角色的类型应实现 [`Serialize`] 和 [`Deserialize`]。

[`Serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`Deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html

从数据结构到非数据结构存在一个连续体，其中有一部分是灰色区域。[`LinkedHashMap`] 和 [`IpAddr`] 是数据结构。有人希望从 JSON 文件中读取 `LinkedHashMap` 或 `IpAddr`，或者通过 IPC 将其发送到另一个进程，这是完全合理的。而 [`LittleEndian`] 不是数据结构。它是 `byteorder` crate 使用的一种标记，用于在编译时针对特定字节顺序进行优化，实际上 `LittleEndian` 的实例在运行时永远不会存在。这些是明确的例子；#rust 或 #serde IRC 频道可以帮助评估更多模棱两可的情况。

[`LinkedHashMap`]: https://docs.rs/linked-hash-map/0.4.2/linked_hash_map/struct.LinkedHashMap.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html
[`LittleEndian`]: https://docs.rs/byteorder/1.0.0/byteorder/enum.LittleEndian.html

如果一个 crate 没有因为其他原因依赖于 Serde，它可以选择通过 Cargo 配置选项来控制 Serde 的实现。这样，下游库只需要在需要这些实现时支付编译 Serde 的代价。

为了与其他基于 Serde 的库保持一致，Cargo 配置的名称应为 `"serde"`。不要使用 `serde_impls` 或 `serde_serialization` 等不同的名称。

在不使用派生的情况下，规范的实现看起来像这样：

```toml
[dependencies]
serde = { version = "1.0", optional = true }
```

```rust
pub struct T { /* ... */ }

#[cfg(feature = "serde")]
impl Serialize for T { /* ... */ }

#[cfg(feature = "serde")]
impl<'de> Deserialize<'de> for T { /* ... */ }
```

使用派生时：

```toml
[dependencies]
serde = { version = "1.0", optional = true, features = ["derive"] }
```

```rust
#[cfg_attr(feature = "serde", derive(Serialize, Deserialize))]
pub struct T { /* ... */ }
```

<a id="c-send-sync"></a>
## 类型在可能的情况下实现 `Send` 和 `Sync`（C-SEND-SYNC）

当编译器确定合适时，`Send` 和 `Sync` 会自动实现。

[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html

在操作原始指针的类型中，要谨慎确保你的类型的 `Send` 和 `Sync` 状态准确反映其线程安全特性。类似于以下的测试可以帮助捕捉类型是否实现 `Send` 或 `Sync` 的非预期回归。

```rust
#[test]
fn test_send() {
    fn assert_send<T: Send>() {}
    assert_send::<MyStrangeType>();
}

#[test]
fn test_sync() {
    fn assert_sync<T: Sync>() {}
    assert_sync::<MyStrangeType>();
}
```

<a id="c-good-err"></a>
## 错误类型应具有意义且表现良好（C-GOOD-ERR）

错误类型是任何 `Result<T, E>` 中的 `E` 类型，它由你的 crate 的任何公共函数返回。错误类型应始终实现 [`std::error::Error`] trait，这是错误处理库如 [`error-chain`] 抽象不同错误类型的机制，并且允许错误作为另一个错误的 [`source()`]。

[`std::error::Error`]: https://doc.rust-lang.org/std/error/trait.Error.html
[`error-chain`]: https://docs.rs/error-chain
[`source()`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.source

此外，错误类型应实现 [`Send`] 和 [`Sync`] trait