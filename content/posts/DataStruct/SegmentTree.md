---
title: "线段树"
date: 2022-02-04T21:03:47+08:00
tags: ["Java"]
categories: ["数据结构"]
draft: false
---

# 线段树

线段树是算法竞赛中常用的用来维护 `区间信息` 的数据结构。

线段树可以在`log(n)`的时间复杂度内实现单点修改、

区间修改、区间查询（区间求和，求区间最大值，求区间最小值）等操作。

## 常见问题

1. 区间染色问题 有一面墙，长度为 n，每次选择一段墙进行染色，m 次操作后，[i,j]
   区间内有多少种颜色

- 染色操作（更新区间）
- 查询操作（查询区间）

2. 区间查询 查询一个区间[i, j]的最大值，最小值，或者区间数字和

- 线段树不是完全二叉树
- 线段树是平衡二叉树 (堆也是平衡二叉树 深度不超过一)
- 可以用数组表示
- 线段树不考虑添加元素，即区间固定

## 线段树需要的空间

对满二叉树： h 层，一共有 `2 ^ h - 1` 个节点（大约是 2^h) 最后一层（h-1），有
`2^(h-1)` 个节点:最后一层的节点数大致等于前面所有层节点之和

如果区间有 n 个元素 数组需要多少个节点 如果 `n = 2 ^ k` 只需要 2n 的空间
最坏情况 ，如果 `n = 2 ^ k + 1` 需要 4n 的空间

## 线段树的数据结构

```java
public class SegmentTree<E> {
  private E[] data;
  private E[] tree;  // 4 * data.length
  private Merge<E> merger;
}
```

## 定义自己的操作方式接口

java 中使用 BiFunction 接口

```java
public interface Merge<E> {
  E merge(E a, E b);
}
```

## 辅助方法 leftChild() rightChild()

```java
public int leftChild(int index) {
  return 2 * index + 1;
}


public int leftChild(int index) {
  return 2 * index + 2;
}
```

## 构造方法

```java
public SegmentTree(E[] arr, Merge<E> merger) {
  this.merge = merger;
  data = (E[]) new Object[arr.length];
  for (int i = 0; i < arr.length; i++)
    data[i] = arr[i];

  tree = (E[]) new Object[data.length];
  buildSegmentTree(0, 0, data.length - 1);
}

// 在 treeIndex 位置创建表示区间 [l...r] 的线段树
public void buildSegmentTree(int treeIndex, int l, int r) {
  if (l == r) {
    tree[treeIndex]  = data[l];
    return;
  }

  int mid = l + (r - l) / 2;
  int leftTreeIndex = leftChild(treeIndex);
  int rightTreeIndex = rightChild(treeIndex);

  buildSegmentTree(leftTreeIndex, l, mid);
  buildSegmentTree(rightTreeIndex, mid + 1, r);

  tree[treeIndex] = merger.merge(tree[leftTreeIndex], tree[rightTreeIndex])
}
```

## 查询 query()

```java
// 返回区间 [queryL, queryR] 的值
public E query(int queryL, int queryR) {
    if (queryL < 0 || queryL >= data.length || queryR < 0 || queryR >= data.length || queryL > queryR)
        throw new IllegalArgumentException("Index is illegal");

    return query(0, 0, data.length - 1, queryL, queryR);
}

// 在以 treeId 为根的线段树中 [l...r] 的范围里， 搜索区间 [queryL...queryR] 的值
private E query(int treeIndex, int l, int r, int queryL, int queryR) {
    if (queryL == l && queryR == r)
        return tree[treeIndex];

    int mid = l + (r - l) / 2;
    // treeIndex的节点分为[l...mid]和[mid+1...r]两部分
    int leftTreeIndex = leftChild(treeIndex);
    int rightTreeIndex = rightChild(treeIndex);

    if (queryR <= mid) {
        return query(leftTreeIndex, l, mid, queryL, queryR);
    } else if (queryL > mid) {
        return query(rightTreeIndex, mid + 1, r, queryL, queryR);
    }

    E leftResult = query(leftTreeIndex, l, mid, queryL, mid);
    E rightResult = query(rightTreeIndex, mid + 1, r, mid + 1, queryR);

    return merger.merge(leftResult, rightResult);
}
```

## 更新 set()

```java
public void set(int index, E e) {
    if (index < 0 || index > data.length - 1)
        throw new IllegalArgumentException("Index is illegal.");
    data[index] = e;
    set(0, 0, data.length - 1, index, e);
}

private void set(int treeIndex, int l, int r, int index, E e) {
    if (l == r) {
        tree[treeIndex] = e;
        return;
    }

    int leftTreeIndex = leftChild(treeIndex);
    int rightTreeIndex = rightChild(treeIndex);

    int mid = l + (r - l) / 2;

    if (index <= mid) {
        set(leftTreeIndex, l, mid, index, e);
    } else { //index > mid
        set(rightTreeIndex, mid + 1, r, index, e);
    }

    tree[treeIndex] = merger.merge(tree[leftTreeIndex], tree[rightTreeIndex]);
}
```
