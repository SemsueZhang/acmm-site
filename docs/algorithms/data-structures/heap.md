# 堆与优先队列

二叉堆是一棵完全二叉树。小根堆满足父节点不大于子节点，因此堆顶是全局最小值；它不保证其余元素整体有序。

## 数组表示与操作

使用 0 下标时，节点 $i$ 的父亲为 $(i-1)/2$，孩子为 $2i+1,2i+2$。

- 插入：放到数组末尾，不断与父亲交换（上浮），$O(\log n)$。
- 删除堆顶：末尾元素移到根，删除末尾，再与较小孩子交换（下沉），$O(\log n)$。
- 取堆顶：$O(1)$。
- 自底向上建堆：从最后一个非叶节点依次下沉，总复杂度 $O(n)$，不是 $O(n\log n)$。

## 例题：合并 $k$ 个有序序列

堆中保存每个序列当前未取出的第一个元素 `(value, sequenceId, position)`。每次弹出最小值，再把它所在序列的下一个元素压入。

```cpp
using State = tuple<int,int,int>;
priority_queue<State, vector<State>, greater<State>> pq;
for (int id = 0; id < k; ++id)
    if (!a[id].empty()) pq.push({a[id][0], id, 0});

while (!pq.empty()) {
    auto [value, id, pos] = pq.top(); pq.pop();
    answer.push_back(value);
    if (pos + 1 < (int)a[id].size())
        pq.push({a[id][pos + 1], id, pos + 1});
}
```

若总元素数为 $N$，堆中至多 $k$ 个状态，时间 $O(N\log k)$，空间 $O(k)$。

## Top K 模型

求最大的 $k$ 个数时，维护大小不超过 $k$ 的小根堆。堆顶是当前入选元素中最小的；新值更大时替换堆顶。时间 $O(n\log k)$。

## STL 写法

```cpp
priority_queue<int> maxHeap; // 大根堆
priority_queue<int, vector<int>, greater<int>> minHeap; // 小根堆
```

## 易错点

- 把堆当作有序数组，试图快速查找或删除任意值。
- `priority_queue` 默认是大根堆。
- 弹出前未判断为空。
- Dijkstra 中修改了距离却不重新入堆；STL 优先队列不支持直接修改键，通常插入新状态并忽略旧状态。
