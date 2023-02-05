# linkedList

真正的动态数据结构

- 最简单的动态数据结构
- 更深入的理解引用（或者指针）
- 更深入的理解递归
- 辅助组成其他数据结构
- 优点：真正的动态，不需要处理固定容量的问题
- 缺点：丧失了随机访问的能力
- 链表的性质符合栈的特点，用于实现栈, 链表头为栈顶
- 通过定义尾指针可以实现队列

## 数据结构

使用私有的内部 node 类

```java
public class LinkedList<E> {

    private class Node {
        public E e;
        public Node next;

        public Node(E e, Node next) {
            this.e = e;
            this.next = next;
        }

        public Node(E e) {
            this(e, null);
        }

        public Node() {
            this(null, null);
        }

        @Override
        public String toString() {
            return e.toString();
        }
    }

    private Node dummyhead; // 虚拟头节点
    private int size;
}

```

## 虚拟头节点

为了统一头节点与其他节点的操作，其值为`null`

### linkedList add()

```java
/**
 * 在链表的index（0-base）位置添加新的元素e
 * 在链表中不是一个常用的操作，练习用
 *
 * @param index
 * @param e
 */
public void add(int index, E e) {
    if (index < 0 || index > size)
        throw new IllegalArgumentException("Add failed. Illegal index");

    Node prev = dummyHead;
    for (int i = 0; i < index; i++)
        prev = prev.next;

    // Node node = new Node(e);
    // node.next = prev.next;
    // prev.next = node;

    prev.next = new Node(e, prev.next);
    size++;
}
```

### LinkedList get()

```java
/**
 * 获得链表的index（0-base）位置的元素
 * 在链表中不是一个常用的操作，练习用
 *
 * @param index
 * @return
 */
public E get(int index) {
    if (index < 0 || index >= size)
        throw new IllegalArgumentException("Get failed, Illegal index");

    Node cur = dummyHead.next;
    for (int i = 0; i < index; i++)
        cur = cur.next;

    return cur.e;

}
```

### LinkedList remove()

```java
/**
 *
 * 在链表的index（0-base）位置删除
 * 在链表中不是一个常用的操作，练习用
 * @param index
 * @return
 */
public E remove(int index) {
    if (index < 0 || index >= size)
        throw new IllegalArgumentException("Remove failed, Illegal index");

    Node prev = dummyHead;
    for (int i = 0; i < index; i++) {
        prev = prev.next;
    }

    Node retNode = prev.next;
    prev.next = retNode.next;
    retNode.next = null;
    size--;

    return retNode.e;
}
```

### LinkedList toString() 包含遍历方式

```java
@Override
public String toString() {
    StringBuilder res = new StringBuilder();
    //  Node cur = dummyHead.next;
    //  while (cur != null) {
    //      res.append(cur).append("->");
    //      cur = cur.next;
    //  }

    for (Node cur = dummyHead.next; cur != null; cur = cur.next)
        res.append(cur).append("->");

    res.append("NULL");
    return res.toString();
}
```

## 使用链表实现队列

### 数据结构

带有尾指针实现队列，队尾增加，队首取出

```java
public class LinkedListQueue<E> implements Queue<E> {

    private class Node {
        public E e;
        public Node next;

        public Node(E e, Node next) {
            this.e = e;
            this.next = next;
        }

        public Node(E e) {
            this(e, null);
        }

        public Node() {
            this(null, null);
        }

        @Override
        public String toString() {
            return e.toString();
        }
    }

    private Node head, tail;
    private int size;

    public LinkedListQueue() {
        head = tail = null;
        size = 0;

    }

}
```

### enqueue()

```java
@Override
public void enqueue(E e) {
    if (tail == null) { // 意味着队列为空，head tail 要同时赋值
        tail = new Node(e);
        head = tail; // 同时赋值
    } else {
        tail.next = new Node(e);
        tail = tail.next;
    }
    size++;
}
```

### dequeue()

```java
@Override
public E dequeue() {
    if (isEmpty())
        throw new IllegalArgumentException("Cannot dequeue from an empty queue");

    Node retNode = head;
    head = head.next;
    retNode.next = null;
    if (head == null) // 队首取出后，如果队列为空了,要将队尾也设置为空
        tail = null;

    size--;
    return retNode.e;
}
```

### isEmpty() toString

```java
@Override
public boolean isEmpty() {
    return size == 0;
}

@Override
public String toString() {
    StringBuilder res = new StringBuilder();
    res.append("Queue: front ");
    //  while (cur != null) {
    //      res.append(cur.e).append("->");
    //      cur = cur.next;
    //  }

    for (Node cur = head; cur != null; cur = cur.next) {
        res.append(cur.e).append("->");
    }

    res.append("NULL tail");
    return res.toString();
}
```

## 时间复杂度

链表由于其特性实现的栈和队列与动态数组的栈和循环队列在时间复杂度同一级别,
但由于 linkedList 包含更多的 new 操作，有时候会落后于数组实现的操作，
在我的电脑上百万级别比较就可以体现了。
