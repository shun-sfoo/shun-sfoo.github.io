---
title: "二叉树"
date: 2022-02-04T17:53:57+08:00
tags: ["Java"]
categories: ["数据结构"]
draft: false
---

## 二叉树

- 二叉树具有唯一根节点
- 二叉树每个节点最多有两个孩子
- 二叉树每个节点最多有一个父亲
- 二叉树具有天然的递归结构 `每个节点的左右子树都是二叉树`
- 二叉树不一定是满的 `一个节点也是二叉树 空也是二叉树`

## 二分搜索树 (Binary Search Tree BST)

- 二分搜索树是二叉树
- 二分搜索树的每个节点的值：`大于其左子树的所有节点的值 小于其右子树的所有节点的值`

### BST 数据结构

```java
public class BST<E extends Comparable<E>> {
    // 要求包含类具有可比性
    private class Node {
        public E e;
        public Node left, right;

        public Node(E e) {
            this.e = e;
            this.left = null;
            this.right = null;
        }
    }

    private Node root;
    private int size;

    public BST() {
        root = null;
        size = 0;
    }
}
```

### add(), contains()

```java
public void add(E e) {
    root = add(root, e);
}

// 向以node为根节点的二分搜索树中插入元素e， 递归算法
// 返回插入新节点后二分搜索树的根
private Node add(Node node, E e) {
    if (node == null) {
        size++;
        return new Node(e);
    }

    if (e.compareTo(node.e) > 0) {
        node.left = add(node.left, e);
    } else if (e.compareTo(node.e) < 0) {
        node.right = add(node.right, e);
    }

    return node;
}

public boolean contains(E e) {
    return contains(root, e);
}

// 查看二分搜索树中是否包含元素e
private boolean contains(Node node, E e) {
    if (node == null)
        return false;

    if (e.compareTo(node.e) < 0) {
        return contains(node.left, e);
    } else if (e.compareTo(node.e) > 0) {
        return contains(node.right, e);
    }
    return true;
}
```

### 前中后序遍历

```java
// 二分搜索树的前序遍历
public void preOrder() {
    preOrder(root);
}

private void preOrder(Node node) {
    if (node == null)
        return;
    System.out.println(node.e);
    preOrder(node.left);
    preOrder(node.right);
}
// inOrder postOrder 中后序遍历略
```

- 中序遍历：按照从小到大顺序排列
- 后序遍历： 应用(C++中)：为二分搜索树释放内存

### 前序遍历的非递归写法

借助栈将根节点压入栈顶，取出栈顶分别压入该节点的右节点和左节点（先入后出）

重复操作直到栈为空

- 二分搜索树遍历的非递归实现，比递归实现复杂很多
- TODO 中序和后序遍历的非递归实现

```java
// 二分搜索树前序遍历非递归实现 借助栈结构， 先入栈右节点然后左节点
public void preOrderNR() {
    Deque<Node> stack = new ArrayDeque<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        Node cur = stack.pop();
        System.out.println(cur.e);
        if (cur.right != null)
            stack.push(cur.right);

        if (cur.left != null)
            stack.push(cur.left);
    }
}
```

### 层序遍历

属于广度优先遍历

- 更快的找到问题的解
- 常用于算法设计中-最短路径

```java
// 二分搜索树层序遍历 借助队列
public void levelOrder() {
    LoopQueue<Node> queue = new LoopQueue<>();
    queue.enqueue(root);
    while (!queue.isEmpty()) {
        Node cur = queue.dequeue();
        System.out.println(cur.e);
        if (cur.left != null)
            queue.enqueue(cur.left);

        if (cur.left != null)
            queue.enqueue(cur.right);
    }

}
```

### 二分搜索树的最大值和最小值

```java
//寻找最大值
public E maximum() {
    if (size == 0)
        throw new IllegalArgumentException("BST is empty");
    return maximum(root).e;
}

private Node maximum(Node node) {
    if (node.right == null)
        return node;
    return maximum(node.right);
}
// 寻找最小值略
```

## BST 删除操作

### 删除最大值/最小值

```java
//删除最大值
public E removeMax() {
    E ret = maximum();
    root = removeMax(root);
    return ret;
}

private Node removeMax(Node node) {
    if (node.right == null) {
        Node leftNode = node.left;
        node.left = null;
        size--;
        return leftNode;
    }

    node.right = removeMax(node.right);
    return node;
}
// 删除最小值略
```

### 删除任意节点

只有左/右子节点：该子节点取代当前节点

#### Hibbard Deletion

删除左右都有孩子的节点 d 找到其后继 s

- `s = min(d->right)`
- `s->right = removeMin(d->right)`
- `s->left = d->left`

删除 d ， s 是新的子树的根

同理删除 d 的前驱（predeccesso）也可以完成删除操作。

```java
// 删除任意节点
public void remove(E e) {
    root = remove(root, e);
}

private Node remove(Node node, E e) {
    if (node == null)
        return null;

    if (e.compareTo(node.e) < 0) {
        node.left = remove(node.left, e);
        return node;
    } else if (e.compareTo(node.e) > 0) {
        node.right = remove(node.right, e);
        return node;
    } else { // e.compareTo(node.e) == 0

        // 待删除左子树为空的情况
        if (node.left == null) {
            Node rightNode = node.right;
            node.right = null;
            size--;
            return rightNode;
        }

        // 待删除右子树为空的情况
        if (node.right == null) {
            Node leftNode = node.left;
            node.left = null;
            size--;
            return leftNode;
        }

        // 待删除左右子树均不为空的情况
        // 找到比待删除节点大的最小节点，即待删除节点右子树的最小节点
        // 用这个节点顶替待删除节点的位置
        Node successor = minimum(node.right);
        successor.right = removeMin(node.right);
        successor.left = node.left;
        node.left = node.right = null;
        return successor;
    }
}
```

## TODO

- 实现 floor ceil rank select
- rank 和 select 的 node 添加一个 size 属性，包含节点个数(包含自身)
