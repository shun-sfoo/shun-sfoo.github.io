# 红黑树性质

结合 2-3 树理解

1. 每个节点或者是红色的，或者是黑色的
2. 根节点是黑色的
3. 每一个叶子节点（最后的空节点） 是黑色的
4. 如果一个节点是红色的，那么他的孩子节点都是黑色的
5. 从任意一个节点到叶子节点，经过的黑色节点是一样的

## 2-3 树

有助于理解红黑树和 B 类树

## 性质

满足二分搜索树的基本性质
节点可以存放一个元素或者两个元素
每个节点有 2 个孩子(2 节点)或者 3 个孩子(3 节点)
2-3 树是一颗绝对平衡的树

## 2-3 树增加节点

新添加节点不会添加到空节点上

```
[37] -> [42] = [37][42]

-----------------------------------------------------
[12] -> [37][42] = [12][37][42] = [37]
                                  /    \
                                [12]   [42]

-----------------------------------------------------

[18] -> [37]   =        [37]
        /  \         /       \
     [12]  [42]   [12][18]    [42]

-----------------------------------------------------

[6]  ->    [37]          =        [37]     =    [37]   =     [12][37]
         /       \              /      \        /   \          / |   \
      [12][18]    [42]    [6][12][18]   [42]   [12] [42]    [6] [18] [42]
                                               /  \
                                             [6]  [18]

-----------------------------------------------------

```

## 红黑树和 2-3 树

相等性，红黑树可以看作 2-3 树
当有三节点时左侧节点作为孩子节点标红表示 2-3 树中两个节点并列
所有的红色节点都是左倾斜的(或者实现右倾斜的版本)
![BRTree](../static/Java/BR-23.png)
![BRTree1](../static/Java/BR-23-1.png)

添加节点默认为红色: 2-3 树中新添加节点总是先融合

红黑树是保持“黑平衡”的二叉树
严格意义上，不是平衡二叉树 最大高度： `2logn` o(logn)

## 红黑树添加新元素

- 2-3 树中添加一个新元素
- 添加进 2 节点，形成一个 3 节点
- 添加进 3 节点，暂时形成一个 4 节点
- 永远添加红色节点
- 根节点为黑色节点

![ADD](../static/Java/BRTree_ADD.png)

图中流程可以总结为： **流程想象对应 2-3 树的变换**

1. 如果节点右孩子为红色而左孩子不为红色（都为红只需要颜色翻转）进行左旋转操作，
   换上去节点的颜色为旋转节点的颜色，旋转节点颜色变成红色, 返回换上去的节点。
   (对应 2 节点插入到右边和 3 节点插入到中间的情况)

```java
//   node                     x
//  /   \     左旋转         /  \
// T1   x   --------->   node   T3
//     / \              /   \
//    T2 T3            T1   T2
private Node leftRotate(Node node) {

    Node x = node.right;

    // 左旋转
    node.right = x.left;
    x.left = node;

    x.color = node.color;
    node.color = RED;

    return x;
}
```

2. 如果左孩子为红色并且左孩子的左孩子也为红色进行右旋转操作。
   换上去节点的颜色为旋转节点的颜色，旋转节点颜色变成红色。

```java
//     node                   x
//    /   \     右旋转       /  \
//   x    T2   ------->   y   node
//  / \                       /  \
// y  T1                     T1  T2
private Node rightRotate(Node node) {

    Node x = node.left;

    // 右旋转
    node.left = x.right;
    x.right = node;

    x.color = node.color;
    node.color = RED;

    return x;
}
```

3. 如果节点的左右孩子都为红色节点进行颜色翻转。

```java
// 颜色翻转
private void flipColors(Node node) {
    node.color = RED;
    node.left.color = BLACK;
    node.right.color = BLACK;
}
```

三个过程并不是互斥的。

### RBTree 数据结构

```java
public class RBTree<K extends Comparable<K>, V> {

    private static final boolean RED = true;
    private static final boolean BLACK = false;

    private class Node {
        public K key;
        public V value;
        public Node left, right;
        public boolean color;

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
            left = null;
            right = null;
            color = RED;
        }
    }

    private Node root;
    private int size;

    public RBTree() {
        root = null;
        size = 0;
    }

}
```

### 辅助方法 isRed() getNode()

```java
// 判断节点node的颜色
private boolean isRed(Node node) {
    // 所有叶子节点为黑色
    if (node == null)
        return BLACK;
    return node.color;
}

// getNode 方法作为实现 contains set get 方法的辅助函数
private Node getNode(Node node, K key) {
  if (node == null)
    return null;
  if (key.compareTo(node.key) > 0) {
    return getNode(node.right, key);
  }else if (key.compareTo(node.key) < 0 ) {
    return getNode(node.left, key);
  }else {
    return node;
  }
}
```

### add()

```java
// 向红黑树中添加新的元素(key, value)
public void add(K key, V value) {
    root = add(root, key, value);
    root.color = BLACK; // 最终根节点为黑色节点
}

// 向以node为根的红黑树中插入元素(key, value)，递归算法
// 返回插入新节点后红黑树的根
private Node add(Node node, K key, V value) {

    if (node == null) {
        size++;
        return new Node(key, value); // 默认插入红色节点
    }

    if (key.compareTo(node.key) < 0)
        node.left = add(node.left, key, value);
    else if (key.compareTo(node.key) > 0)
        node.right = add(node.right, key, value);
    else // key.compareTo(node.key) == 0
        node.value = value;

    if (isRed(node.right) && !isRed(node.left))
        node = leftRotate(node);

    if (isRed(node.left) && isRed(node.left.left))
        node = rightRotate(node);

    if (isRed(node.left) && isRed(node.right))
        flipColors(node);

    return node;
}
```

## TODO 右倾红黑树 Splay Tree （伸展树）
