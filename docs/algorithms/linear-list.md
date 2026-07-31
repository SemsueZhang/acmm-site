# 线性表与 STL 容器

线性表中的元素有明确前后次序。数组和链表都能表示线性表，但它们对“随机访问”和“中间插删”作出不同取舍。

## 数组与 `vector`

连续内存使 `a[i]` 为 $O(1)$，缓存友好；中间插入/删除要移动后续元素，最坏 $O(n)$。`vector` 在容量不足时会申请更大空间并搬迁元素，因此：

- 尾部 `push_back` 是均摊 $O(1)$，不是每次都严格 $O(1)$；
- 扩容后旧的指针、引用、迭代器可能失效；
- 已知规模时 `reserve` 可减少扩容。

### 例题：原地删除指定值

用写指针 `write` 保存下一个保留位置，读指针扫描所有元素：

```cpp
int write = 0;
for (int x : a)
    if (x != target) a[write++] = x;
a.resize(write);
```

每个元素只处理一次，时间 $O(n)$，空间 $O(1)$。在 `vector` 中循环调用 `erase` 可能退化为 $O(n^2)$。

## 链表

链表节点分散存储，单链表保存后继，双链表还保存前驱。已知待操作节点时插入/删除 $O(1)$；但查找第 $k$ 个节点仍需 $O(n)$，所以“链表删除 $O(1)$”隐含前提是已拿到节点位置。

```cpp
struct Node {
    int value;
    Node* next;
};
// 在节点 p 后插入 x
Node* x = new Node{value, p->next};
p->next = x;
```

竞赛中手写链表常用数组模拟：`value[i]` 存值，`next[i]` 存下一节点编号，避免动态内存常数与指针错误。STL `list` 虽支持 $O(1)$ 插删，但通常缓存性能差，只有确实需要稳定迭代器时才使用。

## `pair`、`tuple`、迭代器与常用算法

- `pair`/`tuple` 把多个字段组合，可结构化绑定 `auto [x,y]=p`。
- 迭代器表示容器中的位置，`begin()` 指向首元素，`end()` 指向末尾之后。
- `sort`、`lower_bound` 需要随机访问迭代器，不能直接用于 `list`。
- `set/map` 自动有序，操作 $O(\log n)$；`multiset/multimap` 允许重复键。
- `bitset<N>` 用紧凑位集合支持整体 `& | ^ << >>` 和 `count()`，当状态是固定上限布尔集合时，常比 `vector<bool>` 更快。

### `bitset` 例题：可达和

每个正整数只能选一次，求哪些和可达。`reachable[s]=1` 表示和 $s$ 可达；处理值 $x$ 时左移 $x$ 位并合并：

```cpp
bitset<100001> reachable;
reachable[0] = 1;
for (int x : a) reachable |= reachable << x;
```

右侧读取的是移位前的整个位集，因此每个数使用一次。`bitset` 大小必须是编译期常量，且左移超出范围的位会丢弃。

## 易错点

- 边遍历 `vector` 边 `push_back`，扩容使引用/迭代器失效。
- 误以为链表能 $O(1)$ 按下标访问。
- 删除链表节点后继续访问其指针。
- 对 `set` 使用下标，或误以为 `unordered_map` 有序。
- `bitset` 上限小于实际答案范围，结果被静默截断。
