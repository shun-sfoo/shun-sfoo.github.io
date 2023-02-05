---
title: "数组"
date: 2022-02-02T18:05:51+08:00
tags: ["Java"]
categories: ["数据结构"]
math: true
draft: false
---

## 定义

数组定义为承载某种类型元素可重复集合。

## 数据结构

```java
public class Array<E> {
    /**
     * 数组内容
     */
    private E[] data;

    /**
     * 数组中元素的个数
     */
    private int size;
}
```

capacity 为数据的容量。

## 主要方法

### 添加

```java
/**
 * 在指定索引位置添加元素
 */
public void add(int index, E e) {
    if (index < 0 || index > size)
        throw new 
          IllegalArgumentException("AddLast Failed! Required index >= 0 || index <= size");

    if (size == data.length)
        resize(2 * data.length);

    for (int i = size - 1; i >= index; i--)
        data[i + 1] = data[i];

    data[index] = e;
    size++;
}
```

### 删除

```java
/**
 * 删除index索引处的值，返回该值
 */
public E remove(int index) {
    if (index < 0 || index >= size)
        throw new 
          IllegalArgumentException("AddLast Failed! Required index >= 0 || index < size");

    E ret = data[index];
    for (int i = index + 1; i < size; i++)
        data[i - 1] = data[i];

    size--;
    data[size] = null;

    if (size == data.length / 4 && data.length / 2 != 0) //lazy
        resize(data.length / 2);
    return ret;
}
```

### 更改容量

```java
private void resize(int newCapacity) {
    E[] newData = (E[]) new Object[newCapacity];
    for (int i = 0; i < size; i++)
        newData[i] = data[i];

    data = newData;
}
```

### 查找 e 对应的索引

```java
/**
 * 寻找元素e的索引，不存在返回-1
 */
public int find(E e) {
    for (int i = 0; i < size; i++) {
        if (data[i].equals(e))
            return i;
    }
    return -1;
}
```

### 删除数组中所有为 e 的元素

```Java
/**
 * 删除所有为e的元素
 *
 * @param e
 */
public void removeAllElement(E e) {
    int index = find(e);
    while (index != -1) {
        remove(index);
        index = find(e);
    }
}
```

## 时间复杂度分析

- 增：$ \Omicron(n) $
- 删：$ \Omicron(n) $
- 改：已知索引 $ \Omicron(1) $; 位置索引 $ \Omicron(n) $
- 查：已知索引 $ \Omicron(1) $; 位置索引 $ \Omicron(n) $

### 均摊时间负载度

假设 capacity = n, n+1 次 addLast(), 触发 resize(), 总共进行 2n+1 次基本操作，
平均每次 addLast() 操作，进行 2 次基本操作。 时间复杂度为 $ \Omicron(1) $;同理，
removeLast() 操作时间复杂度也是 $ \Omicron(n) $。

### 复杂度震荡

在数组零界点反复进行 addLast() 后 接着 removeLast()， 每一次都会触发 resize()
这时时间复杂度为 $\Omicron\left(n^{\smash{2}}\right) $

出现问题的原因： removeLast() 时 resize() 过于着急 (Eager) 解决方案： Lazy。 当
`size == capacity / 4` 时， 才将 capacity 减半。
