---
title: 'Chromium'
date: 2022-05-26T19:58:00+08:00
draft: false
---

## develop chromium

notice

[build](https://chromium.googlesource.com/chromium/src/+/main/docs/linux/build_instructions.md)

`gn gen out/Default --export-compile-commands`

```make
.PHONY: run
run:
	LANGUAGE=zh ./out/Default/chrome --ozone-platform-hint=auto
```
