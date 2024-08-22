# 宏


<a id="c-evocative"></a>
## 输入语法能引发对输出的联想 (C-EVOCATIVE)

Rust 宏允许你设计出几乎任何形式的输入语法。尽量保持输入语法与用户代码中其他部分一致，通过尽可能地模仿现有的 Rust 语法，使其更具亲和力和连贯性。注意关键词和标点符号的选择与放置。

一个好的指引是使用类似于宏输出结果的语法，特别是关键词和标点符号。

例如，如果你的宏在输入中声明了一个具有特定名称的结构体，请在该名称前加上关键词 `struct`，以向读者标明正在定义一个具有给定名称的结构体。

```rust
// 优先这样写...
bitflags! {
    struct S: u32 { /* ... */ }
}

// ...而不是省略关键词...
bitflags! {
    S: u32 { /* ... */ }
}

// ...或使用随意的词。
bitflags! {
    flags S: u32 { /* ... */ }
}
```

另一个例子是分号 vs 逗号。在 Rust 中，常量后用分号，所以如果你的宏声明了一连串常量，即使语法略有不同，也应当使用分号。

```rust
// 普通常量使用分号。
const A: u32 = 0b000001;
const B: u32 = 0b000010;

// 所以更建议这样写...
bitflags! {
    struct S: u32 {
        const C = 0b000100;
        const D = 0b001000;
    }
}

// ...而不是这样。
bitflags! {
    struct S: u32 {
        const E = 0b010000,
        const F = 0b100000,
    }
}
```

宏的多样性使得这些具体例子可能不适用，但请考虑如何在你的情况下运用相同的原则。


<a id="c-macro-attr"></a>
## 项目宏与属性很好地组合 (C-MACRO-ATTR)

生成多个输出项目的宏应支持为任何一个输出项目添加属性。一个常见的用法是通过 cfg 置于单个项目后。

```rust
bitflags! {
    struct Flags: u8 {
        #[cfg(windows)]
        const ControlCenter = 0b001;
        #[cfg(unix)]
        const Terminal = 0b010;
    }
}
```

生成结构体或枚举作为输出的宏应该支持属性，以便输出可以与派生属性一起使用。

```rust
bitflags! {
    #[derive(Default, Serialize)]
    struct Flags: u8 {
        const ControlCenter = 0b001;
        const Terminal = 0b010;
    }
}
```

<a id="c-anywhere"></a>
## 项目宏可在任何允许放置项目的地方运行 (C-ANYWHERE)

Rust 允许项目放置在模块级别或更紧凑的范围内，如函数内。项目宏应该在这些地方与普通项目一样正常工作。测试套件应包括在至少模块范围和函数范围内调用宏的情况。

```rust
#[cfg(test)]
mod tests {
    test_your_macro_in_a!(module);

    #[test]
    fn anywhere() {
        test_your_macro_in_a!(function);
    }
}
```

一个简单的错误示例是，这个宏在模块范围内工作良好，但在函数范围内失败。

```rust
macro_rules! broken {
    ($m:ident :: $t:ident) => {
        pub struct $t;
        pub mod $m {
            pub use super::$t;
        }
    }
}

broken!(m::T); // 好的，展开为 T 和 m::T

fn g() {
    broken!(m::U); // 编译失败，super::U 指向的是包含的模块而不是 g
}
```

<a id="c-macro-vis"></a>
## 项目宏支持可见性说明符 (C-MACRO-VIS)

遵循 Rust 语法，宏生成的项目默认是私有的，如果指定 `pub` 则是公有的。

```rust
bitflags! {
    struct PrivateFlags: u8 {
        const A = 0b0001;
        const B = 0b0010;
    }
}

bitflags! {
    pub struct PublicFlags: u8 {
        const C = 0b0100;
        const D = 0b1000;
    }
}
```

<a id="c-macro-ty"></a>
## 类型片段是灵活的 (C-MACRO-TY)

如果你的宏在输入中接受类型片段如 `$t:ty`，它应能与以下所有内容一起使用：

- 基本类型：`u8`，`&str`
- 相对路径：`m::Data`
- 绝对路径：`::base::Data`
- 向上相对路径：`super::Data`
- 泛型：`Vec<String>`

一个简单的错误示例是，这个宏在使用基本类型和绝对路径时工作良好，但在使用相对路径时失败。

```rust
macro_rules! broken {
    ($m:ident => $t:ty) => {
        pub mod $m {
            pub struct Wrapper($t);
        }
    }
}

broken!(a => u8); // 好的

broken!(b => ::std::marker::PhantomData<()>); // 好的

struct S;
broken!(c => S); // 编译失败
```