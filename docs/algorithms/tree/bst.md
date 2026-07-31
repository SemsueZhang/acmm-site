# 二叉搜索树

二叉搜索树（BST）满足：左子树所有键小于当前键，右子树所有键大于当前键；重复键需自行规定放哪侧或在节点维护计数。中序遍历会得到有序序列。

## 查找与插入

比较目标 `x` 与当前键：相等即找到；更小进左子树，更大进右子树。插入沿同样路径找到空位置。

```cpp
struct Node {
    int key, count = 1;
    Node *left = nullptr, *right = nullptr;
};

Node* insert(Node* root, int x) {
    if (!root) return new Node{x};
    if (x < root->key) root->left = insert(root->left, x);
    else if (x > root->key) root->right = insert(root->right, x);
    else ++root->count;
    return root;
}
```

## 删除的三种情况

1. 叶节点：直接删除。
2. 只有一个孩子：用孩子替代它。
3. 两个孩子：用右子树最小键（后继）或左子树最大键（前驱）替换，再从对应子树删除该键。

### 例题

依次插入 `5,3,7,6,8`。删除 7 时它有两个孩子，可用后继 8 替换 7，再删除原叶子 8；中序仍为 `3,5,6,8`。

## 复杂度与退化

操作时间是 $O(h)$，$h$ 为树高。随机形态平均约 $O(\log n)$；若依次插入 `1,2,3,4,5`，树退化成右链，操作变为 $O(n)$。AVL、Treap、Splay 等平衡树通过维持高度解决退化，见[笛卡尔树与平衡树](../data-structures/advanced-trees.md)。

若节点维护 `subtreeSize = leftSize + rightSize + count`，就能按左子树大小求第 $k$ 小与元素排名；每次结构改变后必须更新这一信息。

## 实战建议与易错点

- 只需有序集合时优先用 `set/multiset`，避免手写内存管理。
- `multiset.erase(value)` 会删掉所有等于 value 的元素；只删一个应用 `find` 得到迭代器后 `erase(it)`。
- BST 的“平均 $O(\log n)$”不是最坏保证。
- 两孩子删除时只复制键却忘记计数或其他附加信息。
- 重复键策略不一致，破坏 BST 不变量。
