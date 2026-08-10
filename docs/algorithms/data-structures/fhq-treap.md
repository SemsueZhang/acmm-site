# FHQ Treap

普通二叉搜索树在有序插入时会退化成链。FHQ Treap 给每个节点附加随机优先级：键值满足 BST 性质，优先级满足堆性质。随机优先级让树高在期望意义下保持 $O(\log n)$。

本文维护允许重复键的动态有序多重集合。读者需要先掌握二叉搜索树（Binary Search Tree，BST）的中序有序性质和子树大小；本页重点不是记住六个操作，而是证明“分裂与合并始终保持两项不变量”。

## 节点维护什么

每个节点保存：

- `value`：集合中的键；
- `priority`：随机优先级；
- `left/right`：左右孩子；
- `size`：子树中的元素个数。

本页把重复值作为多个独立节点保存，因此 `size` 会自然计入重复元素。

## 两个基本操作

### 按值分裂

`split(root, value, x, y)` 把一棵树分为：

- `x`：所有键值 $\le value$；
- `y`：所有键值 $>value$。

若根值不大于界限，根和左子树必在 `x` 中，只需继续分裂右子树；反之同理。

正确性可由递归不变量说明：返回时 `x` 中所有键都不大于界限，`y` 中所有键都大于界限，且两棵树内部原有的中序相对顺序和优先级堆序不变。递归只改动搜索路径上的一个孩子，其他子树整体保留。

### 合并

`merge(x,y)` 要求 `x` 中所有值都不大于 `y` 中所有值。比较两棵树根的随机优先级：优先级更小者成为新根，再递归合并尚未确定的一个孩子。

为什么只能选优先级更小的根？新树必须满足小根堆序，根只能是两棵树根中优先级更小者。又因为 `x` 的所有键都不大于 `y`，若 `x` 根成为新根，它的左子树无需变化，只需把 `x` 的右子树与 `y` 合并；另一种情况对称。这样同时保持 BST 与堆两项性质。

```cpp
struct Node {
    int left = 0, right = 0;
    int value = 0, priority = 0, size = 1;
};

vector<Node> tree(1);

int sizeOf(int root) { return root == 0 ? 0 : tree[root].size; }

void pull(int root) {
    tree[root].size = sizeOf(tree[root].left) + sizeOf(tree[root].right) + 1;
}

void split(int root, int value, int& x, int& y) {
    if (root == 0) {
        x = y = 0;
        return;
    }
    if (tree[root].value <= value) {
        x = root;
        split(tree[root].right, value, tree[root].right, y);
        pull(x);
    } else {
        y = root;
        split(tree[root].left, value, x, tree[root].left);
        pull(y);
    }
}

int merge(int x, int y) {
    if (x == 0 || y == 0) return x | y;
    if (tree[x].priority < tree[y].priority) {
        tree[x].right = merge(tree[x].right, y);
        pull(x);
        return x;
    }
    tree[y].left = merge(x, tree[y].left);
    pull(y);
    return y;
}
```

每次递归只沿树高下降，单次分裂或合并期望 $O(\log n)$。

## 如何组合操作

- 插入 `value`：按 `value` 分裂，把新节点合并到两部分之间。
- 删除一个 `value`：先分出 $\le value$，再从中分出 $<value$；把等值树的根删掉一个后合并回来。
- 查询排名：分出 $<value$ 的部分，其大小加 1。
- 第 $k$ 小：比较 $k$ 与左子树大小，决定进入哪棵子树。
- 前驱：分出 $<value$ 的部分，取其中最大值。
- 后继：分出 $\le value$ 的部分，取另一部分最小值。

所有查询后都要把树原样合并回来。

## 真题：P3369 普通平衡树

[洛谷 P3369【模板】普通平衡树](https://www.luogu.com.cn/problem/P3369) 要动态维护可重集合的插入、删除、排名、第 $k$ 小、前驱和后继。下面实现按题目值域使用 `value-1`；若通用数据可能出现 `INT_MIN`，应改写成“严格小于界限”的分裂函数。

题目的六类操作都能归约为“按一个阈值切开有序集合—读取或修改一部分—按原顺序合并”。例如排名只需要严格小于 `value` 的元素数；前驱则是这部分中的最大键。这个映射比背诵每个函数更重要。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int left = 0, right = 0;
    int value = 0, priority = 0, size = 1;
};

class FhqTreap {
    vector<Node> tree;
    int root = 0;
    mt19937 randomEngine{712367821};

    int sizeOf(int node) const { return node == 0 ? 0 : tree[node].size; }
    void pull(int node) {
        tree[node].size = sizeOf(tree[node].left) + sizeOf(tree[node].right) + 1;
    }
    int newNode(int value) {
        tree.push_back({0, 0, value, (int)randomEngine(), 1});
        return (int)tree.size() - 1;
    }
    void split(int node, int value, int& x, int& y) {
        if (node == 0) { x = y = 0; return; }
        if (tree[node].value <= value) {
            x = node;
            split(tree[node].right, value, tree[node].right, y);
            pull(x);
        } else {
            y = node;
            split(tree[node].left, value, x, tree[node].left);
            pull(y);
        }
    }
    int merge(int x, int y) {
        if (x == 0 || y == 0) return x | y;
        if (tree[x].priority < tree[y].priority) {
            tree[x].right = merge(tree[x].right, y);
            pull(x);
            return x;
        }
        tree[y].left = merge(x, tree[y].left);
        pull(y);
        return y;
    }
    int kthNode(int node, int k) const {
        while (node != 0) {
            int leftSize = sizeOf(tree[node].left);
            if (k == leftSize + 1) return node;
            if (k <= leftSize) node = tree[node].left;
            else k -= leftSize + 1, node = tree[node].right;
        }
        return 0;
    }

public:
    explicit FhqTreap(int operations) {
        tree.reserve(operations + 1);
        tree.push_back(Node{});
    }
    void insert(int value) {
        int x, y;
        split(root, value, x, y);
        root = merge(merge(x, newNode(value)), y);
    }
    void eraseOne(int value) {
        int x, equal, y;
        split(root, value, x, y);
        split(x, value - 1, x, equal);
        if (equal != 0) equal = merge(tree[equal].left, tree[equal].right);
        root = merge(merge(x, equal), y);
    }
    int rankOf(int value) {
        int x, y;
        split(root, value - 1, x, y);
        int answer = sizeOf(x) + 1;
        root = merge(x, y);
        return answer;
    }
    int kth(int k) const { return tree[kthNode(root, k)].value; }
    int predecessor(int value) {
        int x, y;
        split(root, value - 1, x, y);
        int answer = tree[kthNode(x, sizeOf(x))].value;
        root = merge(x, y);
        return answer;
    }
    int successor(int value) {
        int x, y;
        split(root, value, x, y);
        int answer = tree[kthNode(y, 1)].value;
        root = merge(x, y);
        return answer;
    }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int operations;
    cin >> operations;
    FhqTreap treap(operations);
    for (int i = 0; i < operations; ++i) {
        int type, value;
        cin >> type >> value;
        if (type == 1) treap.insert(value);
        else if (type == 2) treap.eraseOne(value);
        else if (type == 3) cout << treap.rankOf(value) << '\n';
        else if (type == 4) cout << treap.kth(value) << '\n';
        else if (type == 5) cout << treap.predecessor(value) << '\n';
        else cout << treap.successor(value) << '\n';
    }
    return 0;
}
```

## 为什么复杂度是期望 $O(\log n)$

忽略键值后，随机优先级决定的 Treap 与随机插入形成的 BST 具有相同分布，期望树高为 $O(\log n)$。所有操作只进行常数次分裂、合并或沿树高下降，所以单次期望 $O(\log n)$，空间 $O(n)$。

“期望”不是“最坏保证”。固定、低质量或可预测的伪随机序列可能被针对；需要严格最坏界时应选择其他平衡树。

完整程序中每个操作调用常数次 `split`、`merge` 或沿树高查询，因此单次期望 $O(\log n)$，总空间 $O(q)$，其中 $q$ 是插入节点总数。本实现删除节点后不回收数组槽位，故空间按历史插入次数而不是当前集合大小计算。

## 易错点

- 修改孩子后忘记 `pull`，导致排名和第 $k$ 小错误。
- 调用 `merge(x,y)` 时不满足 `x` 中所有键不大于 `y`。
- 删除等值树时直接清空整棵树，误删所有重复值。
- 查询前驱、后继后忘记合并，破坏根。
- 对可能为最小整数的值计算 `value-1` 发生溢出。
