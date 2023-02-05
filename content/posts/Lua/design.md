---
title: "lua 编程范式"
date: 2023-02-02T12:38:45+08:00
tags: ["lua"]
categories: ["编程"]
hiddenFromHomePage: true
draft: false
---

## 设计

对于编程语言来说除了掌握基本语法，这包含了语言的设计思想之外，如何运用,
组织该语言的程序结构。
这就需要掌握语言的范式，从而可以掌握该语言写项目的最佳实践。对于C语言可以
参考一本《C语言接口与实现》的书。对于lua（主要应用于nvim插件和配置）推荐参考
[mini.nvim](https://github.com/echasnovski/mini.nvim) 一系列的插件。

## lua中的范式

lua中存在多种范式编程，利用其元表的概念可以实现oop等范式。
这里推荐参考的是[mini.nvim](https://github.com/echasnovski/mini.nvim)
一系列的插件。
这系列插件的主要特点是，每个插件都是独立的单lua文件，且不依赖其他插件。
提供了插件标准的配置方式。通过`setup({})`。
