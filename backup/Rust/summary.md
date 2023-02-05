# Rust 语法总结

工欲善其事，必先利其器

# 工具

rustfmt ：格式化工具
clippy: 用于捕捉常见错误和改进 Rust 代码
cargo doc: 生成本地文档
rust-analyzer: 参见我的 neovim 配置,提供 lsp 功能

通过类型系统保证内存安全

```rust
fn spawn<F, T>(f: F) -> JoinHandle<T>
where
    F: FnOnce() -> T,
    F: Send + 'static,
    T: Send + 'static,
{
    Builder::new().spawn(f).expect("failed to spawn theard")
}
```

这里的 `'static ` lifetime bound 是说：F 不能使用任何借用的系统

## 带生命周期限制的借用

- 可以借用任何值（栈内存， 堆内存）
- 编译器检查（不需要耗费运行期的 cpu）
- Rust 借用检查器基本上是个生命周期检查器

## Trait Object

- 无法直接把值赋给 trait(不像 Java ， Rust 没有隐式引用)
- trait object 是胖指针（自动生成）
  ptr: 指向数据的指针， vptr 指向 vtable 的指针
- 动态分发

```rust
use std::io::Write;
let mut buf: Vec<u8> = vec![];
let write: Write = buf; // `Write` does not have a constant size
let writer: &mut Write = &mut buf; // ok. Trait object
```

A type is Send if it's safe to move to another thread.
A type is Sync if it's safe to share between multiple threads.
That is, if `T` is Sync, `&T` is Send.
