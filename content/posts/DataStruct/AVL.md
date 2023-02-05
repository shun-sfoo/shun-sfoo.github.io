---
title: "AVL树"
date: 2022-02-04T18:04:41+08:00
tags: ["Java"]
categories: ["数据结构"]
draft: false
---

## 平衡二叉树和 AVL 树

对于任意一个节点，左子树和右子树的高度差不能超过 1

平衡二叉树的高度和节点数量之间的关系也是 $ \Omicron(\log n) $的

## 平衡因子

判断一棵树是否为 BST

利用 BST 中序遍历过程中访问元素是顺序的特点来判断一棵树是否为 BST

```java
// 判断该二叉树是否是一棵二分搜索树
public boolean isBST() {

    ArrayList<K> keys = new ArrayList<>();
    inOrder(root, keys);
    for (int i = 1; i < keys.size(); i++)
        if (keys.get(i - 1).compareTo(keys.get(i)) > 0)
            return false;
    return true;
}

private void inOrder(Node node, ArrayList<K> keys) {

    if (node == null)
        return;

    inOrder(node.left, keys);
    keys.add(node.key);
    inOrder(node.right, keys);
}
```

### 辅助函数 getHeight()， getBalanceFactor()

```java
// 获得节点node的高度
private int getHeight(Node node) {
    if (node == null)
        return 0;
    return node.height;
}

// 获得节点node的平衡因子
private int getBalanceFactor(Node node) {
    if (node == null)
        return 0;
    return getHeight(node.left) - getHeight(node.right);
}
```

## AVL 的左旋转和右旋转

### 右旋转 LL

当不平衡发生在左孩子节点的左孩子新增节点时

`balanceFactor > 1 && getBalanceFactor(node.left) >= 0`

```txt
// 对节点y进行向右旋转操作，返回旋转后新的根节点x
//        y                              x
//       / \                           /   \
//      x   T4     向右旋转 (y)        z     y
//     / \       - - - - - - - ->    / \   / \
//    z   T3                       T1  T2 T3 T4
//   / \
// T1   T2
private Node rightRotate(Node y) {
    Node x = y.left;
    Node T3 = x.right;

    // 向右旋转过程
    x.right = y;
    y.left = T3;

    // 更新height
    y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;
    x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

    return x;
}
```

### 左旋转 RR

当不平衡发生在右孩子节点的右孩子新增节点时
`balanceFactor < -1 && getBalanceFactor(node.right) <= 0`

```txt
// 对节点y进行向左旋转操作，返回旋转后新的根节点x
    //    y                             x
    //  /  \                          /   \
    // T1   x      向左旋转 (y)       y     z
    //     / \   - - - - - - - ->   / \   / \
    //   T2  z                     T1 T2 T3 T4
    //      / \
    //     T3 T4
    private Node leftRotate(Node y) {
        Node x = y.right;
        Node T2 = x.left;

        // 向左旋转过程
        x.left = y;
        y.right = T2;

        // 更新height
        y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;
        x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

        return x;
    }
```

### LR 与 RL

LR 含义为当前节点左子树比右子树高(平衡因子大于 1),且右子树的左子树比高
将左子树左旋，此时变成了 LL 的情况然后当前节点右旋。 RL 为 LR 的镜像操作

## add()

```java
// 向二分搜索树中添加新的元素(key, value)
public void add(K key, V value) {
    root = add(root, key, value);
}

// 向以node为根的二分搜索树中插入元素(key, value)，递归算法
// 返回插入新节点后二分搜索树的根
private Node add(Node node, K key, V value) {

    if (node == null) {
        size++;
        return new Node(key, value);
    }

    if (key.compareTo(node.key) < 0)
        node.left = add(node.left, key, value);
    else if (key.compareTo(node.key) > 0)
        node.right = add(node.right, key, value);
    else // key.compareTo(node.key) == 0
        node.value = value;

    // 更新height
    node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));

    // 计算平衡因子
    int balanceFactor = getBalanceFactor(node);

    // 平衡维护
    // LL
    if (balanceFactor > 1 && getBalanceFactor(node.left) >= 0)
        return rightRotate(node);

    // RR
    if (balanceFactor < -1 && getBalanceFactor(node.right) <= 0)
        return leftRotate(node);

    // LR
    if (balanceFactor > 1 && getBalanceFactor(node.left) < 0) {
        node.left = leftRotate(node.left);
        return rightRotate(node);
    }

    // RL
    if (balanceFactor < -1 && getBalanceFactor(node.right) > 0) {
        node.right = rightRotate(node.right);
        return leftRotate(node);
    }

    return node;
}
```

## remove()

```java
// 从二分搜索树中删除键为key的节点
public V remove(K key) {

    Node node = getNode(root, key);
    if (node != null) {
        root = remove(root, key);
        return node.value;
    }
    return null;
}

private Node remove(Node node, K key) {

    if (node == null)
        return null;

    Node retNode;
    if (key.compareTo(node.key) < 0) {
        node.left = remove(node.left, key);
        // return node;
        retNode = node;
    } else if (key.compareTo(node.key) > 0) {
        node.right = remove(node.right, key);
        // return node;
        retNode = node;
    } else {   // key.compareTo(node.key) == 0

        // 待删除节点左子树为空的情况
        if (node.left == null) {
            Node rightNode = node.right;
            node.right = null;
            size--;
            // return rightNode;
            retNode = rightNode;
        }

        // 待删除节点右子树为空的情况
        else if (node.right == null) {
            Node leftNode = node.left;
            node.left = null;
            size--;
            // return leftNode;
            retNode = leftNode;
        }

        // 待删除节点左右子树均不为空的情况
        else {
            // 找到比待删除节点大的最小节点, 即待删除节点右子树的最小节点
            // 用这个节点顶替待删除节点的位置
            Node successor = minimum(node.right);
            //successor.right = removeMin(node.right);
            successor.right = remove(node.right, successor.key);
            successor.left = node.left;

            node.left = node.right = null;

            // return successor;
            retNode = successor;
        }
    }

    if (retNode == null)
        return null;

    // 更新height
    retNode.height = 1 + Math.max(getHeight(retNode.left), getHeight(retNode.right));

    // 计算平衡因子
    int balanceFactor = getBalanceFactor(retNode);

    // 平衡维护
    // LL
    if (balanceFactor > 1 && getBalanceFactor(retNode.left) >= 0)
        return rightRotate(retNode);

    // RR
    if (balanceFactor < -1 && getBalanceFactor(retNode.right) <= 0)
        return leftRotate(retNode);

    // LR
    if (balanceFactor > 1 && getBalanceFactor(retNode.left) < 0) {
        retNode.left = leftRotate(retNode.left);
        return rightRotate(retNode);
    }

    // RL
    if (balanceFactor < -1 && getBalanceFactor(retNode.right) > 0) {
        retNode.right = rightRotate(retNode.right);
        return leftRotate(retNode);
    }

    return retNode;
}
```
