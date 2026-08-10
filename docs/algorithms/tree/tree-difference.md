# DFS 序与树上差分

树上差分把许多条路径的逐点修改压缩成端点标记，再通过一次后序遍历统一汇总。它依赖两件事：树上路径唯一，以及子节点贡献只会沿父边向上传递。

## 先理解 DFS 序

进入节点 $u$ 时记录 `tin[u]`，处理完其整棵子树时记录 `tout[u]`。深度优先遍历会连续处理完一棵子树，所以 $u$ 的子树恰好对应区间 `[tin[u], tout[u]]`。因此“单点修改、子树求和”可转成树状数组上的单点修改和区间查询。

## 路径点差分

对路径 $(u,v)$ 上每个点加 1，设 $w=\operatorname{lca}(u,v)$：

```text
diff[u] += 1
diff[v] += 1
diff[w] -= 1
diff[parent[w]] -= 1
```

全部标记后，按后序执行 `diff[u] += diff[child]`。

为什么公式正确？$u,v$ 的两个端点贡献分别向上走，到 $w$ 时合成两份；减去一份后让 $w$ 保留一次，再在 `parent[w]` 减一，阻止贡献继续越过路径最高点。对不在路径上的分支，没有端点贡献进入，因此汇总值为 0。多条路径由加法线性叠加，仍保持这个不变量。

若统计边，把边 `(parent[x],x)` 的答案记在节点 $x$，标记改为 `diff[u]++、diff[v]++、diff[w]-=2`。

## 真题：P3128 Max Flow

[洛谷 P3128 [USACO15DEC] Max Flow](https://www.luogu.com.cn/problem/P3128) 给出多条树上路径，求被经过次数最多的节点。逐条沿路径行走最坏为 $O(nm)$；倍增 LCA 加点差分把每条路径降为 $O(\log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int vertexCount, pathCount;
    cin >> vertexCount >> pathCount;
    vector<vector<int>> graph(vertexCount + 1);
    for (int i = 1; i < vertexCount; ++i) {
        int u, v;
        cin >> u >> v;
        graph[u].push_back(v);
        graph[v].push_back(u);
    }

    int log = 1;
    while ((1 << log) <= vertexCount) ++log;
    vector<vector<int>> parent(log, vector<int>(vertexCount + 1, 0));
    vector<int> depth(vertexCount + 1, 0), order;
    order.reserve(vertexCount);
    stack<int> pending;
    pending.push(1);
    parent[0][1] = 0;
    while (!pending.empty()) {
        int vertex = pending.top();
        pending.pop();
        order.push_back(vertex);
        for (int next : graph[vertex]) {
            if (next == parent[0][vertex]) continue;
            parent[0][next] = vertex;
            depth[next] = depth[vertex] + 1;
            pending.push(next);
        }
    }
    for (int bit = 1; bit < log; ++bit) {
        for (int vertex = 1; vertex <= vertexCount; ++vertex) {
            parent[bit][vertex] = parent[bit - 1][parent[bit - 1][vertex]];
        }
    }

    auto lca = [&](int u, int v) {
        if (depth[u] < depth[v]) swap(u, v);
        int difference = depth[u] - depth[v];
        for (int bit = 0; bit < log; ++bit) {
            if ((difference >> bit) & 1) u = parent[bit][u];
        }
        if (u == v) return u;
        for (int bit = log - 1; bit >= 0; --bit) {
            if (parent[bit][u] != parent[bit][v]) {
                u = parent[bit][u];
                v = parent[bit][v];
            }
        }
        return parent[0][u];
    };

    vector<long long> difference(vertexCount + 1, 0);
    while (pathCount--) {
        int u, v;
        cin >> u >> v;
        int ancestor = lca(u, v);
        ++difference[u];
        ++difference[v];
        --difference[ancestor];
        --difference[parent[0][ancestor]];
    }

    long long answer = 0;
    reverse(order.begin(), order.end());
    for (int vertex : order) {
        answer = max(answer, difference[vertex]);
        difference[parent[0][vertex]] += difference[vertex];
    }
    cout << answer << '\n';
    return 0;
}
```

预处理 $O(n\log n)$，每条路径求 LCA 为 $O(\log n)$，后序汇总 $O(n)$；总时间 $O((n+m)\log n)$、空间 $O(n\log n)$。

## 易错点

- 点差分和边差分公式混用。
- 忘记 `parent[root]=0` 对应的差分哨兵需要存在。
- 标记完没有按后序汇总，或按先序把尚未完整的子树值上传。
- DFS 序的退出时间额外自增，破坏子树对应连续区间的定义。
