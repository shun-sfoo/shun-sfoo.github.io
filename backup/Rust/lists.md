# Too Many Linked Lists

## Basic Data Layout

链表的定义类似于
`List a = Empty | Elem a (List a)`
这是一个复合类型 (sum type) 的递归定义。
在 Rust 中 复合类型是 `enum`

```rust
pub enum List {
    Empty,
    Elem(i32, Box<List>),
}
```

但这不是一个好的定义，下面解释原因

```
[] = stack 在栈中
() = heap 在堆中

// 第一种形式
[Elem A, ptr] -> (Elem B, ptr) -> (Empty, *junk*)

// 第二种形式
[ptr] -> (Elem A, ptr) -> (Elem B, *null*)

```

第一种形式中这里有两点问题

1. 我们分配了一个节点表示 "我其实不是一个节点"
2. 其中一个节点(Elem A)根本没有分配在堆里

为了避免多余垃圾， 统一分配内存，并且从空指针优化中获益。
采取如下定义:

```rust
struct Node {
    elem: i32,
    next: List,
}

pub enum List {
    Empty,
    More(Box<Node>),
}
```

`enum List` 是完全公开的，但 Node 却是内部类型，需要改进成

```rust
pub struct List {
    head: Link,
}

enum Link {
    Empty,
    More(Box<Node>),
}
// => 等效于 type Link = Option<Box<Node>>;

struct Node {
    elem: i32,
    next: Link,
}
```
