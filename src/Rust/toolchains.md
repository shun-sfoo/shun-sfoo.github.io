## ToolChains

## static compile

`rustup target add x86_64-unknown-linux-musl`

if use openssl crate must add `vendored` features

`cargo add openssl --features "vendored"`

[vendored](https://docs.rs/openssl/latest/openssl/#vendored)

### cargo-edit

`cargo install cargo-edit`

### usage

you can simple use it by `cargo add tokio` and it will show the features.
Then you can add the features by `cargo add tokio --features "fs full"`

## cargo-watch

Cargo Watch watches over your project's source for changes, and runs Cargo commands when they occur.

`-x, --exec <cmd>...`

Cargo command(s) to execute on changes [default: check]

## Makefile

`.PHONY`

在 Makefile 中，.PHONY 后面的 target 表示的也是一个伪造的 target, 而不是真实存在的文件 target，注意 Makefile 的 target 默认是文件

if you have a Makefile

```Makefile
.PHONY clean
clean:
 do something
```

and you have a file, it's name is clean and make clean will do something install of make the clean file.

### Makefile in rust

```Makefile
.PHONY watch
watch:
      cargo watch -x 'run -- -r /tmp'
```
