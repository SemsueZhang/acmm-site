# 笛卡尔树与平衡树

这两类树都把序列或有序集合编码成树。它们属于提高级较难内容，先熟练堆、BST 与单调栈再学习。

## 笛卡尔树

给定互异序列，笛卡尔树同时满足：

- 中序遍历下标顺序与原序列一致；
- 节点值满足堆性质（这里以小根堆为例）。

根一定是全局最小值，左右子树分别由根左侧和右侧序列构成。直接递归找最小值会退化为 $O(n^2)$；用单调栈可 $O(n)$ 构造。

```cpp
vector<int> left(n, -1), right(n, -1), st;
for (int i = 0; i < n; ++i) {
    int last = -1;
    while (!st.empty() && a[st.back()] > a[i]) {
        last = st.back();
        st.pop_back();
    }
    if (!st.empty()) right[st.back()] = i;
    left[i] = last;
    st.push_back(i);
}
int root = st.front();
```

### 例题

序列 `3 1 4 2` 的最小值 1 是根；左子树只有 3，右侧 `4 2` 的根为 2，其左孩子为 4。中序遍历仍得到原下标顺序。笛卡尔树常把区间最值结构转成树上的祖先结构。

重复值时必须统一规定相等元素是否弹栈，否则树不唯一。

## 平衡树为什么必要

普通 BST 在有序插入时会退化成链，操作 $O(n)$。平衡树通过旋转或随机优先级保持期望/最坏 $O(\log n)$ 高度。大纲列出 AVL、Treap、Splay 等；竞赛中常见 FHQ Treap，因为分裂、合并较统一。

## FHQ Treap 的核心

每个节点同时保存：键值 `key`、随机优先级 `priority`、子树大小 `size`。它对 `key` 满足 BST 性质，对随机优先级满足堆性质。

- `split(root, key)`：分成键值 `<=key` 和 `>key` 的两棵树。
- `merge(a,b)`：要求 `a` 中所有键都不大于 `b`，按优先级选根并递归合并。
- 插入：先 split，再把新节点夹在中间 merge。
- 求排名/第 $k$ 小：利用左子树大小。

下面给出核心分裂与合并。约定 `split(root,key,x,y)` 把树分为 `x`（键 $\le key$）和 `y`（键 $>key$）；`pull` 重新计算子树大小。

```cpp
struct Node {
    int left = 0, right = 0;
    int key = 0, priority = 0, size = 1;
};
vector<Node> tr(1); // 0 表示空节点，size 为 0

int sizeOf(int u) { return u == 0 ? 0 : tr[u].size; }
void pull(int u) {
    tr[u].size = sizeOf(tr[u].left) + sizeOf(tr[u].right) + 1;
}

void split(int root, int key, int& x, int& y) {
    if (root == 0) { x = y = 0; return; }
    if (tr[root].key <= key) {
        x = root;
        split(tr[root].right, key, tr[root].right, y);
        pull(x);
    } else {
        y = root;
        split(tr[root].left, key, x, tr[root].left);
        pull(y);
    }
}

int merge(int x, int y) {
    if (x == 0 || y == 0) return x | y;
    if (tr[x].priority < tr[y].priority) { // 小优先级为堆顶
        tr[x].right = merge(tr[x].right, y);
        pull(x);
        return x;
    }
    tr[y].left = merge(x, tr[y].left);
    pull(y);
    return y;
}
```

插入键 `value` 时，把根按 `value` 分裂为 `x,y`，创建随机优先级新节点 `z`，再 `root=merge(merge(x,z),y)`。若需要删除一个 `value`，可先按 `value` 和 `value-1` 两次分裂出等值部分，删掉其中一个根后再合并。键可能为 `INT_MIN` 时不能直接计算 `value-1`，可改用“按 `< value` 分裂”的版本。

### 例题：动态第 $k$ 小

集合依次插入 `5,2,8,4`，中序序列是 `2,4,5,8`。求第 3 小时，从根开始：若左子树大小为 2，根恰是第 3 小；若 $k$ 更小进左树，否则令 `k -= leftSize+1` 进右树。

```cpp
int kth(int root, int k) {
    int leftSize = sizeOf(node[root].left);
    if (k == leftSize + 1) return node[root].key;
    if (k <= leftSize) return kth(node[root].left, k);
    return kth(node[root].right, k - leftSize - 1);
}
```

## 实战选择

若只需要有序集合、前驱后继，优先使用 `set/multiset`。需要排名、第 $k$ 小或区间序列操作时才手写平衡树。删除重复值时要明确是删除一个还是全部，可给节点维护出现次数。

## 易错点

- 修改孩子后没有重新计算子树大小。
- `merge` 时不满足左树所有键小于右树的前提。
- Treap 优先级使用固定或质量差的随机数，被数据卡退化。
- 多重集合没有维护重复次数，排名定义混乱。
