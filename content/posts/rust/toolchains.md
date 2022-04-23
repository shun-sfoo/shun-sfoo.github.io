+++
title = "Rust Tool Chains"
date = "2022-04-23T12:01:09+08:00"
author = "shun-sfoo"
authorTwitter = "" #do not include @
cover = ""
tags = ["rust", "tutorial"]
keywords = ["rust", "tutorial"]
description = "Makefile write and rust tool chain"
showFullContent = false
readingTime = false
+++

## ToolChains

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
