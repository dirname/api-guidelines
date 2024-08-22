# 调试能力

<a id="c-debug"></a>
## 所有公共类型都实现了 `Debug` （C-DEBUG）

如果有例外情况，也应该极为罕见。

<a id="c-debug-nonempty"></a>
## `Debug` 的表现形式永远不为空（C-DEBUG-NONEMPTY）

即使对于概念上为空的值，`Debug` 的表现形式也不应该是空的。

```rust
let empty_str = "";
assert_eq!(format!("{:?}", empty_str), "\"\"");

let empty_vec = Vec::<bool>::new();
assert_eq!(format!("{:?}", empty_vec), "[]");
```