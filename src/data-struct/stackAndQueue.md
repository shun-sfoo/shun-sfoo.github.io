# Stack

- 栈也是一种线性结构
- 相比数组，栈对应的操作是数组的子集
- 只能从一段添加元素，也只能从一段取出元素
- 这一端称为栈顶
- 栈是一种 _后进先出_ 的数据结构
- Last In First Out(LIFO)

## Java 中的实现-使用 Deque

`Deque<Integer> stack = new ArrayQueue<>();`

[Java 有哪些不好的设计？ - 刘宇波的回答 - 知乎](https://www.zhihu.com/question/25372706/answer/1252100096)

## 栈的应用

- 无处不在的 Undo 操作（撤销）
- 程序调用的系统栈

## 栈的接口

```java
public interface Stack<E> {
    void push(E e);

    E pop();

    E peek();

    int getSize();

    boolean isEmpty();
}
```

# Queue

- 队列也是一种线性结构
- 相比数组，栈对应的操作是数组的子集
- 只能从一端（队尾）添加元素，只能从 **另一端**（队首）取出元素
- 队列是一种先进先出的数据结构（先到先得）
- First In First Out (FIFO)

## 队列的接口

```java
public interface Queue<E> {
    void enqueue(E e);

    E dequeue();

    E getFront();

    int getSize();

    boolean isEmpty();
}
```

## 循环队列

- 将数组队列(ArrayQueue)的`dequeue` 时间复杂度 `O(n)` 优化成 `O(1)`
- 维护 `front` 和 `tail` 两个变量
- `front == tail` 时队列为空
- `(tail + 1) % capacity == front` 时队列满
- capacity 中，浪费了一个空间

### 循环队列数据结构

```java
public class LoopQueue<E> implements Queue<E> {
    private E[] data;
    private int front, tail;
    private int size; // 可实现无size版本
}
```

### 循环队列主要方法

#### 构造器

```java
public LoopQueue(int capacity) {
    data = (E[]) new Object[capacity + 1]; // loopQueue中会有一个空间浪费
    front = 0;
    tail = 0;
    size = 0;
}
```

#### isEmpty(), getCapacity(), getFront()

```java
@Override
public boolean isEmpty() {
    return tail == front;
}

public int getCapacity() {
    return data.length - 1;
}

@Override
public E getFront() {
    if (isEmpty())
        throw new IllegalArgumentException("Queue is empty");
    return data[front];
}
```

#### enqueue()

```java
@Override
public void enqueue(E e) {
    if ((tail + 1) % data.length == front)  // 数组满了的情况下
        resize(getCapacity() * 2);

    data[tail] = e;
    tail = (tail + 1) % data.length;
    size++;
}
```

#### dequeue()

```java
@Override
public E dequeue() {
    if (isEmpty())
        throw new IllegalArgumentException("Cannot dequeue from an empty queue");
    E ret = data[front];
    data[front] = null;
    front = (front + 1) % data.length;
    size--;
    if (size == getCapacity() / 4 && getCapacity() / 2 != 0)
        resize(getCapacity() / 2);
    return ret;
}
```

#### resize()

```java
private void resize(int newCapacity) {
    E[] newData = (E[]) new Object[newCapacity + 1];
    for (int i = 0; i < size; i++)
        newData[i] = data[(i + front) % data.length];
    data = newData;
    front = 0;
    tail = size;
}
```

#### toString()

```java
@Override
public String toString() {
    StringBuilder res = new StringBuilder();
    res.append(String.format("LoopQueue: size = %d, capacity = %d , front = %d, tail = %d\n", size, getCapacity(), front, tail));
    res.append("LoopQueue: ");
    res.append("front [");
    for (int i = front; i != tail; i = (i + 1) % data.length) { // 另一种遍历
        res.append(data[i]);
        if ((i + 1) % data.length != tail)
            res.append(", ");
    }
    res.append("] tail");
    return res.toString();
}

```

## 栈，队列时间复杂度分析

| 结构          | 方法               | 时间复杂度 |
| ------------- | ------------------ | ---------- |
| ArrayStack<E> | void push(E)       | O(1) 均摊  |
|               | E pop(E)           | O(1) 均摊  |
|               | E peek(E)          | O(1)       |
|               | int getSize(E)     | O(1)       |
|               | boolean isEmpty(E) | O(1)       |
| ArrayQueue<E> | void enquque(E)    | O(1) 均摊  |
|               | E dequque()        | **O(n)**   |
|               | E getFront()       | O(1)       |
|               | int getSize()      | O(1)       |
|               | boolean isEmpty()  | O(1)       |
| LoopQueue<E>  | void enquque(E)    | O(1) 均摊  |
|               | E dequque()        | O(1) 均摊  |
|               | E getFront()       | O(1)       |
|               | int getSize()      | O(1)       |
|               | boolean isEmpty()  | O(1)       |
