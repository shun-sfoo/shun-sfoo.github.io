---
title: "Rust工具链"
date: 2023-02-05T10:28:30+08:00
tags: ["Rust"]
categories: ["编程"]
draft: false
---

## cross compile 交叉编译

### static compile

`rustup target add x86_64-unknown-linux-musl`

### openssl (deprecated)

if use openssl crate must add `vendored` features

`cargo add openssl --features "vendored"`

[vendored](https://docs.rs/openssl/latest/openssl/#vendored)

### Use Rustls

first in archlinux should `sudo pacman -S --needed musl`

and then when add a crate choose the `rustls` features instead of `nativtls` or
`openssl`

### linux to windows 10

1. rustup toolchain install stable-x86_64-pc-windows-gnu (need to check is
   necessary)

2. rustup target add x86_64-pc-windows-gnu

3. sudo pacman -S mingw-w64-gcc

## cargo-edit

`cargo install cargo-edit`

### usage

you can simple use it by `cargo add tokio` and it will show the features. Then
you can add the features by `cargo add tokio --features "fs full"`

## cargo-watch

Cargo Watch watches over your project's source for changes, and runs Cargo
commands when they occur.

`-x, --exec <cmd>...`

Cargo command(s) to execute on changes [default: check]

## Makefile

`.PHONY`

在 Makefile 中，.PHONY 后面的 target 表示的也是一个伪造的 target,
而不是真实存在的文件 target，注意 Makefile 的 target 默认是文件

if you have a Makefile

```Makefile
.PHONY clean
clean:
 do something
```

and you have a file, it's name is clean and make clean will do something install
of make the clean file.

### Makefile in rust

```Makefile
.PHONY watch
watch:
      cargo watch -x 'run -- -r /tmp'
```

### 工具

rustfmt ：格式化工具 clippy: 用于捕捉常见错误和改进 Rust 代码 cargo doc:
生成本地文档 rust-analyzer: 参见我的 neovim 配置,提供 lsp 功能

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

这里的 `'static` lifetime bound 是说：F 不能使用任何借用的系统

## 带生命周期限制的借用

- 可以借用任何值（栈内存， 堆内存）
- 编译器检查（不需要耗费运行期的 cpu）
- Rust 借用检查器基本上是个生命周期检查器

## Trait Object

- 无法直接把值赋给 trait(不像 Java ， Rust 没有隐式引用)
- trait object 是胖指针（自动生成） ptr: 指向数据的指针， vptr 指向 vtable
  的指针
- 动态分发

```rust
use std::io::Write;
let mut buf: Vec<u8> = vec![];
let write: Write = buf; // `Write` does not have a constant size
let writer: &mut Write = &mut buf; // ok. Trait object
```

A type is Send if it's safe to move to another thread. A type is Sync if it's
safe to share between multiple threads. That is, if `T` is Sync, `&T` is Send.

### bacon

[介绍](https://zjp-cn.github.io/neovim0.6-blogs/nvim/rust/bacon.html)

[nvim插件](https://github.com/Canop/nvim-bacon)
