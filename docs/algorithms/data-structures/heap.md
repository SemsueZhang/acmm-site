# 堆与优先队列

二叉堆是一棵完全二叉树。小根堆满足父节点不大于子节点，因此堆顶是全局最小值；它不保证其余元素整体有序。

本文讨论数组实现的二叉堆和标准库优先队列。核心不变量只有两项：数组形状始终对应完全二叉树，父子之间始终满足堆序。理解插入、删除如何恢复这两项，比记忆 `priority_queue` 接口更重要。

## 数组表示与操作

使用 0 下标时，节点 $i$ 的父亲为 $(i-1)/2$，孩子为 $2i+1,2i+2$。

- 插入：放到数组末尾，不断与父亲交换（上浮），$O(\log n)$。
- 删除堆顶：末尾元素移到根，删除末尾，再与较小孩子交换（下沉），$O(\log n)$。
- 取堆顶：$O(1)$。
- 自底向上建堆：从最后一个非叶节点依次下沉，总复杂度 $O(n)$，不是 $O(n\log n)$。

插入只可能破坏新节点到根路径上的堆序，上浮到父亲不再更大时即可停止；删除堆顶后也只有替补根到叶的一条路径可能失序，每次与更合适的孩子交换即可恢复。因此两种操作的正确性来自“其他边从未改变”，步数至多为树高 $O(\log n)$。

线性建堆的证明不能简单把每个节点都按 $O(\log n)$ 相加。高度至少为 $h$ 的节点至多约有 $n/2^h$ 个，总代价满足

$$
\sum_{h\ge 1}\frac{n}{2^h}h=O(n).
$$

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

## 真题：P3378 堆

[洛谷 P3378【模板】堆](https://www.luogu.com.cn/problem/P3378) 支持插入、查询最小值和删除最小值，恰好对应小根堆的三个基本操作。

题目从不要求查询第二小或删除任意值，所以堆暴露的“只保证堆顶最优”正好足够；若需要按键查找或有序遍历，应改用有序集合等结构。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int operations;
    cin >> operations;
    priority_queue<int, vector<int>, greater<int>> heap;

    while (operations--) {
        int type;
        cin >> type;
        if (type == 1) {
            int value;
            cin >> value;
            heap.push(value);
        } else if (type == 2) {
            cout << heap.top() << '\n';
        } else {
            heap.pop();
        }
    }
    return 0;
}
```

查询堆顶 $O(1)$，插入和删除 $O(\log n)$，空间 $O(n)$。题目保证查询和删除时堆非空；通用程序仍应自行检查这一前提。

## 易错点

- 把堆当作有序数组，试图快速查找或删除任意值。
- `priority_queue` 默认是大根堆。
- 弹出前未判断为空。
- Dijkstra 中修改了距离却不重新入堆；STL 优先队列不支持直接修改键，通常插入新状态并忽略旧状态。
