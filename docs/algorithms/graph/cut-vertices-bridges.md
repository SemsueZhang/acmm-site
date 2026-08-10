# 无向图的割点与桥

在无向图中，删除一条边后连通分量数量增加，这条边称为桥；删除一个顶点及其 incident edges 后连通分量增加，这个点称为割点。两者共享无向 DFS 树上的 `dfn` 与 `low`，但判定条件和根节点处理不同。

## `low` 表示什么

`dfn[u]` 是首次访问时间；`low[u]` 是从 $u$ 的 DFS 子树出发，经过若干树边和至多一条非父边，能够到达的最小 `dfn`。无向边存两次，所以必须按边编号排除“刚走来的那一条父边”，不能按父顶点排除全部平行边。

## 桥的判定

DFS 树边 $(u,v)$ 是桥，当且仅当

$$
low[v]>dfn[u].
$$

若 `low[v] <= dfn[u]`，子树能通过另一条边回到 $u$ 或更早祖先，删除树边后仍有替代路线；若严格大于，子树与外部的唯一连接就是该树边。

## 割点的判定

非根顶点 $u$ 是割点，当存在孩子 $v$ 满足

$$
low[v]\ge dfn[u].
$$

等号也要算：子树即使能回到 $u$，删除 $u$ 后仍无法到达 $u$ 的祖先。DFS 根没有祖先，只有当它拥有至少两个 DFS 树孩子时才是割点。

## 手算例子

三角形 `1-2-3-1` 再接边 `3-4`。三角形边都有替代路线，不是桥；`3-4` 没有替代路线，是桥。删除点 3 后，点 4 与 `{1,2}` 分离，所以 3 是割点。

## 真实题目：P3388 割点

[洛谷 P3388【模板】割点](https://www.luogu.com.cn/problem/P3388) 要输出无向图全部割点。图可能不连通，同一割点可能被多个孩子重复发现，因此使用布尔数组标记。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Edge {
    int to;
    int id;
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    cin >> n >> m;
    vector<vector<Edge>> graph(n + 1);
    for (int id = 0; id < m; ++id) {
        int u, v;
        cin >> u >> v;
        graph[u].push_back({v, id});
        graph[v].push_back({u, id});
    }

    vector<int> dfn(n + 1), low(n + 1);
    vector<bool> isCutVertex(n + 1, false);
    int timer = 0;
    function<void(int, int, int)> dfs = [&](int u, int parentEdge,
                                             int root) {
        dfn[u] = low[u] = ++timer;
        int treeChildren = 0;
        for (Edge edge : graph[u]) {
            if (edge.id == parentEdge) continue;
            int v = edge.to;
            if (dfn[v] == 0) {
                ++treeChildren;
                dfs(v, edge.id, root);
                low[u] = min(low[u], low[v]);
                if (u != root && low[v] >= dfn[u]) isCutVertex[u] = true;
            } else {
                low[u] = min(low[u], dfn[v]);
            }
        }
        if (u == root && treeChildren >= 2) isCutVertex[u] = true;
    };

    for (int vertex = 1; vertex <= n; ++vertex) {
        if (dfn[vertex] == 0) dfs(vertex, -1, vertex);
    }

    int count = 0;
    for (int vertex = 1; vertex <= n; ++vertex) count += isCutVertex[vertex];
    cout << count << '\n';
    for (int vertex = 1; vertex <= n; ++vertex) {
        if (isCutVertex[vertex]) cout << vertex << ' ';
    }
    cout << '\n';
    return 0;
}
```

每个顶点访问一次，每条无向边检查两次，时间 $O(n+m)$，空间 $O(n+m)$。若要同时求桥，在树边递归返回后额外判断 `low[v] > dfn[u]`，并保存该边编号。

## 易错点

- 割点使用 `>` 而不是 `>=`。
- DFS 根套用普通顶点条件。
- 按父顶点跳边，平行边时把合法返祖边一并忽略。
- 图不连通却只启动一次 DFS。
