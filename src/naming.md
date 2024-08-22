# 命名

<a id="c-case"></a>
## 大小写遵循 RFC 430 (C-CASE)

基本的 Rust 命名约定在 [RFC 430] 中有描述。

一般来说，Rust 倾向于使用 `UpperCamelCase`（大驼峰命名法）用于“类型级”构造（类型和特征），使用 `snake_case`（蛇形命名法）用于“值级”构造。更准确地说：

| 项目 | 约定 |
| ---- | ---- |
| Crates | [不明确](https://github.com/rust-lang/api-guidelines/issues/29) |
| Modules | `snake_case` |
| Types | `UpperCamelCase` |
| Traits | `UpperCamelCase` |
| Enum variants | `UpperCamelCase` |
| Functions | `snake_case` |
| Methods | `snake_case` |
| General constructors | `new` or `with_more_details` |
| Conversion constructors | `from_some_other_type` |
| Macros | `snake_case!` |
| Local variables | `snake_case` |
| Statics | `SCREAMING_SNAKE_CASE` |
| Constants | `SCREAMING_SNAKE_CASE` |
| Type parameters | 简洁的 `UpperCamelCase`，通常是单个大写字母：`T` |
| Lifetimes | 简短的 `lowercase`，通常是单个字母：`'a`，`'de`，`'src` |
| Features | [不明确](https://github.com/rust-lang/api-guidelines/issues/101) 但是见 [C-FEATURE] |

在 `UpperCamelCase` 中，缩略词和复合词的缩写算作一个词：使用 `Uuid` 而不是 `UUID`，`Usize` 而不是 `USize`，或 `Stdin` 而不是 `StdIn`。在 `snake_case` 中，缩略词和缩写则要小写：`is_xid_start`。

在 `snake_case` 或 `SCREAMING_SNAKE_CASE` 中，一个“单词”不应该只包含一个字母，除非它是最后一个“单词”。所以，我们有 `btree_map` 而不是 `b_tree_map`，但是有 `PI_2` 而不是 `PI2`。

Crate 名称不应使用 `-rs` 或 `-rust` 作为后缀或前缀。每个 crate 都是 Rust！没必要一直提醒用户这一点。

[RFC 430]: https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md
[C-FEATURE]: #c-feature

### 标准库中的示例

整个标准库。这条准则应该很容易！

<a id="c-conv"></a>
## 临时转换遵循 `as_`，`to_`，`into_` 约定 (C-CONV)

转换应该以方法的形式提供，其名称根据以下前缀命名：

| 前缀 | 成本 | 所有权 |
| ---- | ---- | ------ |
| `as_` | 免费 | 借用 -> 借用 |
| `to_` | 昂贵 | 借用 -> 借用<br>借用 -> 拥有 (非 Copy 类型)<br>拥有 -> 拥有 (Copy 类型) |
| `into_`| 可变 | 拥有 -> 拥有 (非 Copy 类型) |

例如：

- [`str::as_bytes()`] 提供字符串作为一片 UTF-8 字节视图，这个操作是免费的。输入是借用的 `&str`，输出是借用的 `&[u8]`。
- [`Path::to_str`] 在操作系统路径的字节上执行昂贵的 UTF-8 检查。输入和输出都是借用的。称之为 `as_str` 在运行时有非琐碎的成本是不正确的。
- [`str::to_lowercase()`] 生成 `str` 的 Unicode 正确的小写版本，这涉及字符串字符的迭代并可能需要内存分配。输入是借用的 `&str`，输出是拥有的 `String`。
- [`f64::to_radians()`] 将浮点数从度转换为弧度。输入是 `f64`。传递引用 `&f64` 是不必要的，因为 `f64` 的复制成本很低。称此函数为 `into_radians` 会产生误导，因为输入不会被消耗。
- [`String::into_bytes()`] 提取 `String` 的底层 `Vec<u8>`，这是免费的。它获取 `String` 的所有权并返回拥有的 `Vec<u8>`。
- [`BufReader::into_inner()`] 获取一个缓冲读取器的所有权，并提取出底层读取器，这是免费的。缓冲的数据会被丢弃。
- [`BufWriter::into_inner()`] 获取一个缓冲写入器的所有权，并提取出底层写入器，这可能需要昂贵的刷新操作来处理任何缓冲数据。

[`str::as_bytes()`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`Path::to_str`]: https://doc.rust-lang.org/std/path/struct.Path.html#method.to_str
[`str::to_lowercase()`]: https://doc.rust-lang.org/std/primitive.str.html#method.to_lowercase
[`f64::to_radians()`]: https://doc.rust-lang.org/std/primitive.f64.html#method.to_radians
[`String::into_bytes()`]: https://doc.rust-lang.org/std/string/struct.String.html#method.into_bytes
[`BufReader::into_inner()`]: https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner
[`BufWriter::into_inner()`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html#method.into_inner

以 `as_` 和 `into_` 为前缀的转换通常 _降低抽象_，要么暴露对底层表示的视图（`as`），要么将数据分解为其底层表示（`into`）。而以 `to_` 为前缀的转换通常保持在同一抽象级别，但会进行一些工作以在表示之间进行转换。

当一个类型包装单个值以将其与更高级别的语义关联时，访问被包装的值应通过 `into_inner()` 方法提供。这适用于提供缓冲的包装器，如 [`BufReader`]，编码或解码如 [`GzDecoder`]，原子访问如 [`AtomicBool`]，或任何类似语义的情况。

[`BufReader`]: https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner
[`GzDecoder`]: https://docs.rs/flate2/0.2.19/flate2/read/struct.GzDecoder.html#method.into_inner
[`AtomicBool`]: https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.into_inner

如果转换方法名称中的 `mut` 修饰符是返回类型的一部分，则其应如同其在类型中出现的方式出现。例如 [`Vec::as_mut_slice`] 返回 mut 切片；它名副其实。该名称优于 `as_slice_mut`。

[`Vec::as_mut_slice`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.as_mut_slice

```rust
// 返回类型是一个可变切片。
fn as_mut_slice(&mut self) -> &mut [T];
```

##### 更多标准库的示例

- [`Result::as_ref`](https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref)
- [`RefCell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr)
- [`slice::to_vec`](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec)
- [`Option::into_iter`](https://doc.rust-lang.org/std/option/enum.Option.html#method.into_iter)

<a id="c-getter"></a>
## Getter 名称遵循 Rust 约定 (C-GETTER)

除少数例外，Rust 代码中的 getter 不使用 `get_` 前缀。

```rust
pub struct S {
    first: First,
    second: Second,
}

impl S {
    // 不是 get_first。
    pub fn first(&self) -> &First {
        &self.first
    }

    // 不是 get_first_mut、get_mut_first 或 mut_first。
    pub fn first_mut(&mut self) -> &mut First {
        &mut self.first
    }
}
```

`get` 命名仅在有单一且明显的东西可以通过 getter 获得时使用。例如 [`Cell::get`] 用于访问 `Cell` 的内容。

[`Cell::get`]: https://doc.rust-lang.org/std/cell/struct.Cell.html#method.get

对于进行运行时验证（如边界检查）的 getter，考虑添加不安全的 `_unchecked` 变体。通常这些方法具有以下签名：

```rust
fn get(&self, index: K) -> Option<&V>;
fn get_mut(&mut self, index: K) -> Option<&mut V>;
unsafe fn get_unchecked(&self, index: K) -> &V;
unsafe fn get_unchecked_mut(&mut self, index: K) -> &mut V;
```

getters 和转换 ([C-CONV](#c-conv)) 之间的区别可能很微妙，并不总是很明确。例如 [`TempDir::path`] 可以被理解为临时目录的文件系统路径的 getter，而 [`TempDir::into_path`] 是一种转换，将删除临时目录的责任转移给调用者。由于 `path` 是一个 getter，将其称为 `get_path` 或 `as_path` 是不正确的。

[`TempDir::path`]: https://docs.rs/tempdir/0.3.5/tempdir/struct.TempDir.html#method.path
[`TempDir::into_path`]: https://docs.rs/tempdir/0.3.5/tempdir/struct.TempDir.html#method.into_path

### 标准库的示例

- [`std::io::Cursor::get_mut`](https://doc.rust-lang.org/std/io/struct.Cursor.html#method.get_mut)
- [`std::ptr::Unique::get_mut`](https://doc.rust-lang.org/std/ptr/struct.Unique.html#method.get_mut)
- [`std::sync::PoisonError::get_mut`](https://doc.rust-lang.org/std/sync/struct.PoisonError.html#method.get_mut)
- [`std::sync::atomic::AtomicBool::get_mut`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.get_mut)
- [`std::collections::hash_map::OccupiedEntry::get_mut`](https://doc.rust-lang.org/std/collections/hash_map/struct.OccupiedEntry.html#method.get_mut)
- [`<[T]>::get_unchecked`](https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked)

<a id="c-iter"></a>
## 生成迭代器的方法遵循 `iter`，`iter_mut`，`into_iter` 约定 (C-ITER)

根据 [RFC 199]。

对于包含元素类型 `U` 的容器，迭代器方法应被命名为：

```rust
fn iter(&self) -> Iter             // Iter 实现了 Iterator<Item = &U>
fn iter_mut(&mut self) -> IterMut  // IterMut 实现了 Iterator<Item = &mut U>
fn into_iter(self) -> IntoIter     // IntoIter 实现了 Iterator<Item = U>
```

此准则适用于概念上同质集合的数据结构。作为反例，`str` 类型是一片保证为有效 UTF-8 的字节。这在概念上比同质集合更为复杂，因此它没有提供 `iter`/`iter_mut`/`into_iter` 一组迭代器方法，而是提供了 [`str::bytes`] 以字节迭代和 [`str::chars`] 以字符迭代。

[`str::bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.bytes
[`str::chars`]: https://doc.rust-lang.org/std/primitive.str.html#method.chars

这一准则只适用于方法，而不适用于函数。例如，`url` crate 中的 [`percent_encode`] 返回一个迭代器，具有对百分编码字符串片段的迭代功能。使用 `iter`/`iter_mut`/`into_iter` 约定并不会带来清晰性。

[`percent_encode`]: https://docs.rs/url/1.4.0/url/percent_encoding/fn.percent_encode.html
[RFC 199]: https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md

### 标准库的示例

- [`Vec::iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter)
- [`Vec::iter_mut`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut)
- [`Vec::into_iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter)
- [`BTreeMap::iter`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter)
- [`BTreeMap::iter_mut`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter_mut)

<a id="c-iter-ty"></a>
## 迭代器类型名称与生成它们的方法匹配 (C-ITER-TY)

名为 `into_iter()` 的方法应返回一个名为 `IntoIter` 的类型，而其他所有返回迭代器的方法也是如此。

这一准则主要适用于方法，但通常也对函数适用。例如，`url` crate 中的 [`percent_encode`] 函数返回一个名为 [`PercentEncode`][PercentEncode-type] 的迭代器类型。

[PercentEncode-type]: https://docs.rs/url/1.4.0/url/percent_encoding/struct.PercentEncode.html

这些类型名称在其所属模块的前缀下最有意义，例如 [`vec::IntoIter`]。

[`vec::IntoIter`]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html

### 标准库的示例

* [`Vec::iter`] 返回 [`Iter`][slice::Iter]
* [`Vec::iter_mut`] 返回 [`IterMut`][slice::IterMut]
* [`Vec::into_iter`] 返回 [`IntoIter`][vec::IntoIter]
* [`BTreeMap::keys`] 返回 [`Keys`][btree_map::Keys]
* [`BTreeMap::values`] 返回 [`Values`][btree_map::Values]

[`Vec::iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter
[slice::Iter]: https://doc.rust-lang.org/std/slice/struct.Iter.html
[`Vec::iter_mut`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut
[slice::IterMut]: https://doc.rust-lang.org/std/slice/struct.IterMut.html
[`Vec::into_iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter
[vec::IntoIter]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html
[`BTreeMap::keys`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.keys
[btree_map::Keys]: https://doc.rust-lang.org/std/collections/btree_map/struct.Keys.html
[`BTreeMap::values`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.values
[btree_map::Values]: https://doc.rust-lang.org/std/collections/btree_map/struct.Values.html

<a id="c-feature"></a>
## 功能名称不应包含占位符词语 (C-FEATURE)

不要在 [Cargo 功能] 的名称中包含无意义的词，比如 `use-abc` 或 `with-abc`。直接将功能命名为 `abc`。

[Cargo 功能]: http://doc.crates.io/manifest.html#the-features-section

这在对 Rust 标准库有可选依赖的 crate 中尤为普遍。正确的做法是：

```toml
# 在 Cargo.toml 中

[features]
default = ["std"]
std = []
```

```rust
// 在 lib.rs 中
#![no_std]

#[cfg(feature = "std")]
extern crate std;
```

不要将功能命名为 `use-std` 或 `with-std`，或其他不具创意且不是 `std` 的名字。这种命名约定与 Cargo 为可选依赖推断的隐式功能命名一致。考虑 crate `x` 对 Serde 和 Rust 标准库的可选依赖：

```toml
[package]
name = "x"
version = "0.1.0"

[features]
std = ["serde/std"]

[dependencies]
serde = { version = "1.0", optional = true }
```

当我们依赖 `x` 时，可以通过 `features = ["serde"]` 启用可选 Serde 依赖。同样地，我们可以通过 `features = ["std"]` 启用可选标准库依赖。Cargo 为可选依赖推断的隐式功能名为 `serde`，而不是 `use-serde` 或 `with-serde`，因此我们希望显式功能以相同方式工作。

另外需要注意的是，Cargo 要求功能是增量性的，因此像 `no-abc` 这样的负面命名几乎从来都不正确。

<a id="c-word-order"></a>
## 名称使用一致的词序 (C-WORD-ORDER)

以下是标准库中的一些错误类型：

- [`JoinPathsError`](https://doc.rust-lang.org/std/env/struct.JoinPathsError.html)
- [`ParseBoolError`](https://doc.rust-lang.org/std/str/struct.ParseBoolError.html)
- [`ParseCharError`](https://doc.rust-lang.org/std/char/struct.ParseCharError.html)
- [`ParseFloatError`](https://doc.rust-lang.org/std/num/struct.ParseFloatError.html)
- [`ParseIntError`](https://doc.rust-lang.org/std/num/struct.ParseIntError.html)
- [`RecvTimeoutError`](https://doc.rust-lang.org/std/sync/mpsc/enum.RecvTimeoutError.html)
- [`StripPrefixError`](https://doc.rust-lang.org/std/path/struct.StripPrefixError.html)

以上所有都使用了动词-对象-错误词序。如果我们要添加一个表示地址解析失败的错误，为了一致性，我们应该按照动词-对象-错误词序命名它为 `ParseAddrError`，而不是 `AddrParseError`。

特定的词序选择并不重要，但要注意内部的统一性，以及与标准库中类似功能的一致性。