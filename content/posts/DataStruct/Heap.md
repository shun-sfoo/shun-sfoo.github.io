---
title: "堆"
date: 2022-02-04T17:46:15+08:00
tags: ["Java"]
categories: ["数据结构"]
draft: false
---

## 堆的基本结构

二叉堆 (Binary Heap)

- 二叉堆是一颗完全二叉树（把元素顺序排列成树的形状）
- 堆中每个节点的值总是不大于其父节点的值 （最大堆）
- 用数组储存二叉堆
  `parent(i) = (i-1)/2; left child (i) = 2*i+1; right child (i) = 2*i+2`

### 数据结构

```java
public class MaxHeap<E extends Comparable<E>> {
    private Array<E> data;

    public MaxHeap(int capacity) {
        data = new Array<>(capacity);
    }

    public MaxHeap() {
        data = new Array<>();
    }

    // heapify
    // 所有叶子节点都可以视为完全二叉树
    // 最后一个不为叶子节点的索引为 (arr.length - 1) / 2
    // 由此节点往前遍历每个索引进行shiftDown操作可将数组直接转换成最大二叉堆
    public MaxHeap(E[] arr) {
        data = new Array<>(arr);
        if (arr.length != 1) {
            for (int i = parent(arr.length - 1); i >= 0; i--) {
                shiftDown(i);
            }
        }
    }

}
```

### 辅助函数 parent(), leftChild(), rightChild()

```java
//返回完全二叉树的数组表示中， 一个索引所表示的元素的父亲节点的索引
private int parent(int index) {
    if (index == 0)
        throw new IllegalArgumentException("index-0 doesn't have parent");

    return (index - 1) / 2;
}

//返回完全二叉树的数组表示中， 一个索引所表示的元素的左孩子节点的索引
private int leftChild(int index) {
    return index * 2 + 1;
}

//返回完全二叉树的数组表示中， 一个索引所表示的元素的右孩子节点的索引
private int rightChild(int index) {
    return index * 2 + 2;
}
```

### 上浮和下沉 shiftUp() shitfDown()

上浮应用于新添加元素时 下沉应用于元素发生变动时

```java
// 上浮
private void shiftUp(int k) {
    while (k > 0 && data.get(k).compareTo(data.get(parent(k))) > 0) {
        data.swap(k, parent(k));
        k = parent(k);
    }
}

// 下沉
private void shiftDown(int k) {
    // 循环终止条件 以索引 k 为节点的没有左右子节点
    // 即 leftChild 的值越界  leftChild >= data.getSize()
    while (leftChild(k) < data.getSize()) {
        int j = leftChild(k);
        if (j + 1 < data.getSize() && data.get(j + 1).compareTo(data.get(j)) > 0)
            j = rightChild(k);

        // data[j] 是 leftChild 和 rightChild 中的最大值
        if (data.get(k).compareTo(data.get(j)) >= 0)
            break;

        data.swap(k, j);
        k = j;
    }
}
```

### 原地堆排序

直接在数组的基础上排序，不新开辟空间保存数据。

将队首最大的元素与队尾互换，然后针对除队尾外其他元素形成最大堆 依次进行替换。

```java
// 原地堆排序
public void sort() {
    for (int i = data.getSize() - 1; i > 0; i--) {
        data.swap(0, i);
        shiftDown(0, i);
    }
}

private void shiftDown(int k, int n) {
    while (leftChild(k) < n) {
        int j = leftChild(k);
        if (j + 1 < n && data.get(j).compareTo(data.get(j + 1)) < 0)
            j = rightChild(k);
        if (data.get(k).compareTo(data.get(j)) >= 0)
            break;
        data.swap(k, j);
        k = j;
    }
}
```
