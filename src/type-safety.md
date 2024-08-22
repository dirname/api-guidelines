# 类型安全

<a id="c-newtype"></a>
## 新类型提供静态区分 (C-NEWTYPE)

新类型可以静态地区分底层类型的不同解释。

例如，一个 `f64` 值可能用来表示以英里或公里为单位的数量。通过使用新类型，我们可以跟踪预期的解释：

```rust
struct Miles(pub f64);
struct Kilometers(pub f64);

impl Miles {
    fn to_kilometers(self) -> Kilometers { /* ... */ }
}
impl Kilometers {
    fn to_miles(self) -> Miles { /* ... */ }
}
```

一旦我们将这两种类型分开，我们就可以静态地确保不会混淆它们。例如，以下函数

```rust
fn are_we_there_yet(distance_travelled: Miles) -> bool { /* ... */ }
```

不能被意外地传入一个 `Kilometers` 值。编译器会提醒我们进行转换，从而避免某些[灾难性的错误]。

[灾难性的错误]: http://en.wikipedia.org/wiki/Mars_Climate_Orbiter

<a id="c-custom-type"></a>
## 参数通过类型传达意义，而不是 `bool` 或 `Option` (C-CUSTOM-TYPE)

推荐使用

```rust
let w = Widget::new(Small, Round)
```

而不是

```rust
let w = Widget::new(true, false)
```

`bool`、`u8` 和 `Option` 等核心类型有许多可能的解释。

使用明确的类型（无论是枚举、结构体还是元组）来传达解释和不变量。在上面的例子中，如果不查找参数名称，很难立即理解 `true` 和 `false` 所表达的含义，而 `Small` 和 `Round` 则更具暗示性。

使用自定义类型使得以后扩展选项变得更加容易，例如通过添加 `ExtraLarge` 变体。

有关包装现有类型并赋予其区别名称的无成本方法，请参见新类型模式 ([C-NEWTYPE])。

[C-NEWTYPE]: #c-newtype

<a id="c-bitflag"></a>
## 标志集的类型应使用 `bitflags`，而不是枚举 (C-BITFLAG)

Rust 支持具有显式指定判别值的 `enum` 类型：

```rust
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}
```

当 `enum` 类型需要与其他系统/语言兼容地序列化为整数值时，自定义判别值非常有用。它们支持“类型安全”的 API：通过接受 `Color` 类型而不是整数，函数可以确保获得格式良好的输入，即使稍后将这些输入视为整数。

一个 `enum` 允许 API 从多个选项中准确请求一个选择。而有时，API 的输入是多个标志的存在或不存在。在 C 代码中，这通常通过让每个标志对应一个特定位来实现，从而允许一个整数表示 32 或 64 个标志。Rust 的 [`bitflags`] crate 提供了这种模式的类型安全表示。

[`bitflags`]: https://github.com/bitflags/bitflags

```rust
use bitflags::bitflags;

bitflags! {
    struct Flags: u32 {
        const FLAG_A = 0b00000001;
        const FLAG_B = 0b00000010;
        const FLAG_C = 0b00000100;
    }
}

fn f(settings: Flags) {
    if settings.contains(Flags::FLAG_A) {
        println!("执行操作 A");
    }
    if settings.contains(Flags::FLAG_B) {
        println!("执行操作 B");
    }
    if settings.contains(Flags::FLAG_C) {
        println!("执行操作 C");
    }
}

fn main() {
    f(Flags::FLAG_A | Flags::FLAG_C);
}
```

<a id="c-builder"></a>
## 构建器支持复杂值的构建 (C-BUILDER)

有些数据结构由于其构建需要：

* 大量输入
* 复合数据（例如切片）
* 可选的配置数据
* 在多种模式之间进行选择

这很容易导致大量的独立构造函数，每个构造函数都有许多参数。

如果 `T` 是这样一个数据结构，可以考虑引入一个 `T` _构建器_：

1. 引入一个单独的数据类型 `TBuilder`，用于逐步配置一个 `T` 值。如果可能的话，选择一个更好的名称：例如 [`Command`] 是一个 [子进程] 的构建器，[`Url`] 可以从 [`ParseOptions`] 创建。
2. 构建器的构造函数应该只接受构造 `T` 所需的数据作为参数。
3. 构建器应提供一套方便的方法进行配置，包括逐步设置复合输入（如切片）。这些方法应返回 `self` 以允许链式调用。
4. 构建器应提供一个或多个“_终端_”方法来实际构建一个 `T`。

[`Command`]: https://doc.rust-lang.org/std/process/struct.Command.html
[子进程]: https://doc.rust-lang.org/std/process/struct.Child.html
[`Url`]: https://docs.rs/url/1.4.0/url/struct.Url.html
[`ParseOptions`]: https://docs.rs/url/1.4.0/url/struct.ParseOptions.html

当构建一个 `T` 涉及副作用（如启动任务或进程）时，构建器模式尤其合适。

在 Rust 中，有两种不同的构建器模式，区别在于所有权的处理，如下所述。

### 非消费型构建器（首选）

在某些情况下，构建最终的 `T` 并不需要消费构建器本身。以下是 [`std::process::Command`] 的一个示例：

[`std::process::Command`]: https://doc.rust-lang.org/std/process/struct.Command.html

```rust
// 注意：实际的 Command API 并不使用拥有的字符串；
// 这是一个简化版本。

pub struct Command {
    program: String,
    args: Vec<String>,
    cwd: Option<String>,
    // 等等
}

impl Command {
    pub fn new(program: String) -> Command {
        Command {
            program: program,
            args: Vec::new(),
            cwd: None,
        }
    }

    /// 添加一个传递给程序的参数。
    pub fn arg(&mut self, arg: String) -> &mut Command {
        self.args.push(arg);
        self
    }

    /// 添加多个传递给程序的参数。
    pub fn args(&mut self, args: &[String]) -> &mut Command {
        self.args.extend_from_slice(args);
        self
    }

    /// 设置子进程的工作目录。
    pub fn current_dir(&mut self, dir: String) -> &mut Command {
        self.cwd = Some(dir);
        self
    }

    /// 以子进程的形式执行命令，并返回子进程。
    pub fn spawn(&self) -> io::Result<Child> {
        /* ... */
    }
}
```

注意，`spawn` 方法实际上使用构建器配置来启动进程，它通过共享引用来获取构建器。这是可能的，因为启动进程并不需要配置数据的所有权。

由于终端 `spawn` 方法只需要引用，配置方法接受并返回 `self` 的可变借用。

#### 优点

通过在整个过程中使用借用，`Command` 可以方便地用于一行代码和更复杂的构建：

```rust
// 一行代码
Command::new("/bin/cat").arg("file.txt").spawn();

// 复杂配置
let mut cmd = Command::new("/bin/ls");
if size_sorted {
    cmd.arg("-S");
}
cmd.arg(".");
cmd.spawn();
```

### 消费型构建器

有时构建器在构建最终类型 `T` 时必须转移所有权，这意味着终端方法必须接受 `self` 而不是 `&self`。

```rust
impl TaskBuilder {
    /// 为即将创建的任务命名。
    pub fn named(mut self, name: String) -> TaskBuilder {
        self.name = Some(name);
        self
    }

    /// 重定向任务本地的标准输出。
    pub fn stdout(mut self, stdout: Box<io::Write + Send>) -> TaskBuilder {
        self.stdout = Some(stdout);
        self
    }

    /// 创建并执行一个新的子任务。
    pub fn spawn<F>(self, f: F) where F: FnOnce() + Send {
        /* ... */
    }
}
```

这里，`stdout` 配置涉及传递 `io::Write` 的所有权，该所有权必须在构建时转移到任务中（在 `spawn` 中）。

当构建器的终端方法需要所有权时，存在一个基本的权衡：

* 如果其他构建器方法接受/返回可变借用，复杂配置情况会很好处理，但一行代码的配置变得不可能。

* 如果其他构建器方法接受/返回拥有的 `self`，一行代码的配置仍然可以正常工作，但复杂配置变得不太方便。

在使简单的事情