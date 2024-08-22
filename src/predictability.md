# 可预测性

<a id="c-smart-ptr"></a>
## 智能指针不添加固有方法 (C-SMART-PTR)

例如，这就是 [`Box::into_raw`] 函数定义方式的原因。

[`Box::into_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_raw

```rust
impl<T> Box<T> where T: ?Sized {
    fn into_raw(b: Box<T>) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;
let ptr = Box::into_raw(boxed_str);
```

如果将其定义为一个固有方法，那么在调用点就会产生混淆，不知道被调用的方法是 `Box<T>` 上的方法还是 `T` 上的方法。

```rust
impl<T> Box<T> where T: ?Sized {
    // 不要这样做。
    fn into_raw(self) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;

// 这是通过智能指针 Deref 实现访问的 str 上的方法。
boxed_str.chars()

// 这是 Box<str> 上的方法...?
boxed_str.into_raw()
```

<a id="c-conv-specific"></a>
## 转换发生在涉及的最具体的类型上 (C-CONV-SPECIFIC)

在有疑问时，优先使用 `to_`/`as_`/`into_` 而不是 `from_`，因为它们更符合使用习惯（并且可以与其他方法链式调用）。

对于两种类型之间的许多转换，其中一种类型显然更加“具体”：它提供了一些在其他类型中不存在的附加不变性或解释。例如，[`str`] 比 `&[u8]` 更具体，因为它是一个 UTF-8 编码的字节序列。

[`str`]: https://doc.rust-lang.org/std/primitive.str.html

转换应当存在于涉及的类型中更具体的那个上。因此，`str` 提供了 [`as_bytes`] 方法和 [`from_utf8`] 构造器用于与 `&[u8]` 值之间的转换。除了直观之外，这种约定还避免了使用具体类型如 `&[u8]` 被无穷尽的转换方法污染。

[`as_bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`from_utf8`]: https://doc.rust-lang.org/std/str/fn.from_utf8.html

<a id="c-method"></a>
## 具有明确接收对象的函数是方法 (C-METHOD)

倾向于

```rust
impl Foo {
    pub fn frob(&self, w: widget) { /* ... */ }
}
```

而非

```rust
pub fn frob(foo: &Foo, w: widget) { /* ... */ }
```

用于任何明显与特定类型关联的操作。

方法相较于函数有很多优势：

* 它们不需要导入或限定即可使用：你只需要一个合适类型的值。
* 它们的调用执行自动借用（包括可变借用）。
* 它们使回答“我可以用类型 `T` 的值做什么”这一问题变得容易（特别是在使用 rustdoc 时）。
* 它们提供 `self` 表示法，更简洁且通常更清晰地传达所有权区别。

<a id="c-no-out"></a>
## 函数不接受输出参数 (C-NO-OUT)

倾向于

```rust
fn foo() -> (Bar, Bar)
```

而非

```rust
fn foo(output: &mut Bar) -> Bar
```

用于返回多个 `Bar` 值。

诸如元组和结构体的复合返回类型编译得非常高效，不需要堆分配。如果一个函数需要返回多个值，应通过这些类型之一来返回。

主要的例外情况：有时候函数旨在修改调用者已经拥有的数据，例如重用一个缓冲区：

```rust
fn read(&mut self, buf: &mut [u8]) -> io::Result<usize>
```

<a id="c-overload"></a>
## 操作符重载应不令人惊讶 (C-OVERLOAD)

带有内建语法（`*`, `|`, 等）的操作符可以通过实现 [`std::ops`] 中的特征为类型提供。这些操作符伴随着强烈的期望：仅为与乘法有某种相似性的运算（并共享期望的属性, 如结合律）实现 `Mul`，其他特征同理。

[`std::ops`]: https://doc.rust-lang.org/std/ops/index.html#traits

<a id="c-deref"></a>
## 只有智能指针实现 `Deref` 和 `DerefMut` (C-DEREF)

`Deref` 特征在很多情况下被编译器隐式使用，并与方法解析交互。相关规则专门设计用于适应智能指针，因此这些特征应仅用于此目的。

### 标准库中的示例

- [`Box<T>`](https://doc.rust-lang.org/std/boxed/struct.Box.html)
- [`String`](https://doc.rust-lang.org/std/string/struct.String.html) 是指向 [`str`](https://doc.rust-lang.org/std/primitive.str.html) 的智能指针
- [`Rc<T>`](https://doc.rust-lang.org/std/rc/struct.Rc.html)
- [`Arc<T>`](https://doc.rust-lang.org/std/sync/struct.Arc.html)
- [`Cow<'a, T>`](https://doc.rust-lang.org/std/borrow/enum.Cow.html)

<a id="c-ctor"></a>
## 构造函数是静态的、固有的方法 (C-CTOR)

在 Rust 中，“构造函数”只是一个约定。关于构造函数命名有各种约定，区别往往微妙。

最基本形式的构造函数是一个无参数的 `new` 方法。

```rust
impl<T> Example<T> {
    pub fn new() -> Example<T> { /* ... */ }
}
```

构造函数是为它们所构造的类型定义的静态（无 `self`）固有方法。结合完全导入类型名称的做法，这种约定可导致信息丰富但简洁的构造：

```rust
use example::Example;

// 构造一个新的 Example。
let ex = Example::new();
```

名称 `new` 通常用于实例化类型的主要方法。有时它不带参数，如上例。有时它确实带有参数，如将值放入 `Box` 的 [`Box::new`]。

一些类型的构造函数，尤其是 I/O 资源类型，使用与其构造函数相对应的命名约定，如 [`File::open`]、[`Mmap::open`]、[`TcpStream::connect`] 和 [`UdpSocket::bind`]。在这些情况下，名称是根据具体领域选择的。

通常，有多种方式构造一个类型。在这些情况下，辅助构造函数常常带有 `_with_foo` 后缀，如 [`Mmap::open_with_offset`]。如果你的类型有多种构造选项，请考虑使用构建器模式 ([C-BUILDER])。

一些构造函数是“转换构造函数”，即从其他类型的现有值创建新类型的方法。它们通常具有以 `from_` 开头的名称，如 [`std::io::Error::from_raw_os_error`]。但也请注意 `From` 特征 ([C-CONV-TRAITS])，它与此非常相似。`from_` 前缀的转换构造函数和 `From<T>` 实现之间有三个区别：

- `from_` 构造函数可以是 unsafe 的；而 `From` 实现不能。例如 [`Box::from_raw`]。
- `from_` 构造函数可以接受附加参数来消除源数据的意义，以 [`u64::from_str_radix`] 为例。
- `From` 实现仅适用于当源数据类型足以确定输出数据类型的编码时。当输入只是数据包，如在 [`u64::from_be`] 或 [`String::from_utf8`] 中，转换构造函数名称能够标识其意义。

[`Box::from_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.from_raw
[`u64::from_str_radix`]: https://doc.rust-lang.org/std/primitive.u64.html#method.from_str_radix
[`u64::from_be`]: https://doc.rust-lang.org/std/primitive.u64.html#method.from_be
[`String::from_utf8`]: https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8

注意，通常且预期的做法是类型同时实现 `Default` 和一个 `new` 构造函数。对于同时拥有这两者的类型，它们应具有相同的行为。任何一个都可以通过另一个来实现。

[C-BUILDER]: type-safety.html#c-builder
[C-CONV-TRAITS]: interoperability.html#c-conv-traits

### 标准库中的示例

- [`std::io::Error::new`] 是用于 IO 错误的常用构造函数。
- [`std::io::Error::from_raw_os_error`] 是基于从操作系统接收到的错误代码的转换构造函数。
- [`Box::new`] 创建一个新容器类型，接受一个参数。
- [`File::open`] 打开一个文件资源。
- [`Mmap::open_with_offset`] 打开一个内存映射文件，并带有附加选项。

[`File::open`]: https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.open
[`Mmap::open`]: https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open
[`Mmap::open_with_offset`]: https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open_with_offset
[`TcpStream::connect`]: https://doc.rust-lang.org/stable/std/net/struct.TcpStream.html#method.connect
[`UdpSocket::bind`]: https://doc.rust-lang.org/stable/std/net/struct.UdpSocket.html#method.bind
[`std::io::Error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new
[`std::io::Error::from_raw_os_error`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.from_raw_os_error
[`Box::new`]: https://doc.rust-lang.org/stable/std/boxed/struct.Box.html#method.new