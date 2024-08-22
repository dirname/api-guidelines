# 灵活性

<a id="c-intermediate"></a>
## 函数暴露中间结果以避免重复工作 (C-INTERMEDIATE)

许多函数在回答问题的同时也计算了相关的有趣数据。如果这些数据对客户端可能有用，应该考虑在 API 中暴露这些数据。

### 标准库中的示例

- [`Vec::binary_search`] 不返回一个表示是否找到值的 `bool`，也不返回一个表示可能找到值的索引的 `Option<usize>`。它返回的是有关索引的信息，如果找到则返回索引，如果没找到则返回插入值所需的索引。

- [`String::from_utf8`] 如果输入字节不是 UTF-8，则可能会失败。在错误情况下，它返回一个中间结果，该结果显示输入中有效 UTF-8 的字节偏移量，并返还输入字节的所有权。

- [`HashMap::insert`] 返回一个 `Option<T>`，它返回给定键的预存在值，如果有的话。对于用户想要恢复这个值的情况，由插入操作返回它可以避免用户进行第二次哈希表查找。

[`Vec::binary_search`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.binary_search
[`String::from_utf8`]: https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8
[`HashMap::insert`]: https://doc.rust-lang.org/stable/std/collections/struct.HashMap.html#method.insert

<a id="c-caller-control"></a>
## 调用者决定复制和放置数据的位置 (C-CALLER-CONTROL)

如果函数需要参数的所有权，它应该接收参数的所有权，而不是借用和克隆参数。

```rust
// 优先这样：
fn foo(b: Bar) {
    /* 直接使用 b 的所有权 */
}

// 而不是这样：
fn foo(b: &Bar) {
    let b = b.clone();
    /* 克隆后使用 b 的所有权 */
}
```

如果函数*不需要*参数的所有权，它应该借用参数，而不是获取所有权并丢弃参数。

```rust
// 优先这样：
fn foo(b: &Bar) {
    /* 借用 b 使用 */
}

// 而不是这样：
fn foo(b: Bar) {
    /* 借用 b 使用，函数返回前会隐式丢弃 */
}
```

只有在绝对需要时才应将 `Copy` trait 用作约束，而不是作为标记副本应该容易制作的方式。

<a id="c-generic"></a>
## 函数通过使用泛型来最小化对参数的假设 (C-GENERIC)

函数对其输入做出的假设越少，其可用性就越广。

优先

```rust
fn foo<I: IntoIterator<Item = i64>>(iter: I) { /* ... */ }
```

而不是任何以下实现

```rust
fn foo(c: &[i64]) { /* ... */ }
fn foo(c: &Vec<i64>) { /* ... */ }
fn foo(c: &SomeOtherCollection<i64>) { /* ... */ }
```

如果函数只需要迭代数据。

更一般地，考虑使用泛型来准确地点出函数需要对其参数进行的假设。

### 泛型的优势

* _重用性_。泛型函数可以应用于一个开放式的类型集合，同时为这些类型需要提供的功能提供一个清晰的契约。

* _静态调度和优化_。每次使用泛型函数时，都会专门为实现 trait 约束的特定类型进行“单态化”，这意味着 (1) trait 方法的调用为静态的、直接的实现调用，（2）编译器可以内联和进一步优化这些调用。

* _内联布局_。如果一个 `struct` 和 `enum` 类型是某个类型参数 `T` 的泛型，`T` 类型的值将在 `struct`/`enum` 中内联布局，而没有任何间接性。

* _推断_。由于通常可以推断给泛型函数的类型参数，泛型函数可以帮助减少代码中的冗长，通常无需显式转换或其他方法调用。

* _精确类型_。因为泛型为实现一个 trait 的特定类型提供了一个_名称_，所以可以准确地表示需要或产生该确切类型的地方。例如，一个函数

  ```rust
  fn binary<T: Trait>(x: T, y: T) -> T
  ```

  可以保证使用和返回完全相同类型 `T` 的元素；不能用不同类型的参数调用此函数，即使这些类型都实现了 `Trait`。

### 泛型的劣势

* _代码大小_。专用的泛型函数意味着函数主体被复制。必须权衡代码大小的增加与静态调度的性能收益之间的关系。

* _同质类型_。这是“精确类型”问题的另一面：如果 `T` 是一个类型参数，它代表一个_单一_实际类型。因此，例如，`Vec<T>` 包含单一具体类型的元素（实际上，矢量表示被专门用于内联这些元素）。有时，异类集合是有用的；见 [trait 对象][C-OBJECT]。

* _签名冗长_。大量使用泛型可能使函数的签名更难以阅读和理解。

[C-OBJECT]: #c-object

### 标准库中的示例

- [`std::fs::File::open`] 接受一个通用类型 `AsRef<Path>` 的参数。这允许通过字符串字面量 `"f.txt"`、[`Path`]、[`OsString`] 和一些其他类型方便地打开文件。

[`std::fs::File::open`]: https://doc.rust-lang.org/std/fs/struct.File.html#method.open
[`Path`]: https://doc.rust-lang.org/std/path/struct.Path.html
[`OsString`]: https

<a id="c-object"></a>
## 具有潜在用途的 trait 对象是对象安全的 (C-OBJECT)

trait 对象有一些显著的限制：通过 trait 对象调用的方法不能使用泛型，并且不能在接收者位置之外使用 `Self`。

设计 trait 时，及早决定是将它用作对象还是用作泛型的约束。

如果一个 trait 是被用作对象，其方法应该接收和返回 trait 对象而不是使用泛型。

`where` 子句 `Self: Sized` 可用于将特定方法从 trait 的对象中排除出去。以下 trait 由于泛型方法而不是对象安全的。

```rust
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T);
}
```

为泛型方法添加 `Self: Sized` 要求，将其从 trait 对象中排除，使 trait 对象安全。

```rust
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T) where Self: Sized;
}
```

### trait 对象的优点

* _异构性_。当你需要它时，你真的需要它。
* _代码大小_。与泛型不同，trait 对象不生成专用（单态化）的代码版本，这可以大大减少代码大小。

### trait 对象的缺点

* _无泛型方法_。trait 对象目前不能提供泛型方法。
* _动态调度和胖指针_。trait 对象本身涉及间接性和 vtable 调度，这可能带来性能上的代价。
* _无 Self_。除接收者参数以外，trait 对象上的方法不能使用 `Self` 类型。

### 标准库中的示例

- [`io::Read`] 和 [`io::Write`] trait 经常用作对象。
- [`Iterator`] trait 有几个用 `where Self: Sized` 标记的泛型方法，以保留将 `Iterator` 用作对象的能力。

[`io::Read`]: https://doc.rust-lang.org/std/io/trait.Read.html
[`io::Write`]: https://doc.rust-lang.org/std/io/trait.Write.html
[`Iterator`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html