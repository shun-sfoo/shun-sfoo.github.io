# 并查集

# 解决连接问题

## 数据结构

```java
public class UnionFind implements UF {

    // rank[i]表示以i为根的集合所表示的树的层数
    // 在后续的代码中, 并不会维护rank的语意, 也就是rank的值在路径压缩的过程中, 有可能不在是树的层数值
    // 这也是rank不叫height或者depth的原因, 它只是作为比较的一个标准
    private int[] rank;
    private int[] parent; // parent[i]表示第i个元素所指向的父节点

    // 构造函数
    public UnionFind(int size) {

        rank = new int[size];
        parent = new int[size];

        // 初始化, 每一个parent[i]指向自己, 表示每一个元素自己自成一个集合
        for (int i = 0; i < size; i++) {
            parent[i] = i;
            rank[i] = 1;
        }
    }

    @Override
    public int getSize() {
        return parent.length;
    }

}
```

将每一个元素，看作是一个节点
由孩子指向父节点

## 基于 rank 的优化

`rank[i]` 表示树的高度

## 路径压缩

发生在 `find()` 操作时
`parent[p] = parent[parent[p]]`

```java
// 查找过程, 查找元素p所对应的集合编号
// O(h)复杂度, h为树的高度
private int find(int p) {
    if (p < 0 || p >= parent.length)
        throw new IllegalArgumentException("p is out of bound.");

    while (p != parent[p]) {
        parent[p] = parent[parent[p]];
        p = parent[p];
    }
    return p;
}
```

## 链接操作

```java
@Override
public void unionElements(int p, int q) {

    int pRoot = find(p);
    int qRoot = find(q);

    if (pRoot == qRoot)
        return;

    // 根据两个元素所在树的rank不同判断合并方向
    // 将rank低的集合合并到rank高的集合上
    if (rank[pRoot] < rank[qRoot])
        parent[pRoot] = qRoot;
    else if (rank[qRoot] < rank[pRoot])
        parent[qRoot] = pRoot;
    else { // rank[pRoot] == rank[qRoot]
        parent[pRoot] = qRoot;
        rank[qRoot] += 1;   // 此时, 我维护rank的值
    }
}
```

## 时间复杂度

`o(log*n)` -> iterated logarithm

$ log\*n = \begin{cases} 0 &\text{if } (n <= 1) \\\ 1+log\*(logn) &\text{if } (n > 1) \end{cases} $

近乎是 O(1) 级别的
