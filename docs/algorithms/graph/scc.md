# 强连通分量与缩点

在有向图中，若两个顶点互相可达，则它们属于同一个强连通分量（Strongly Connected Component，SCC）。把每个 SCC 缩成一个点后一定得到有向无环图（Directed Acyclic Graph，DAG），否则环上的分量仍互相可达，本应合并。

## `dfn` 与 `low` 的含义

Tarjan 算法进行深度优先搜索：

- `dfn[u]` 是 $u$ 首次访问的时间；
- `low[u]` 是从 $u$ 的 DFS 子树出发，沿树边并至多通过一条指向“仍在栈中顶点”的边，能够到达的最小 `dfn`；
- 栈保存已经访问、但尚未确定所属 SCC 的顶点。

树边递归返回时用 `low[v]` 更新；指向栈内顶点的边用 `dfn[v]` 更新。指向已经出栈分量的边不能更新，因为那个分量已封闭，不可能与当前子树互相可达。

当 `low[u]==dfn[u]` 时，$u$ 无法通过当前未定分量到达更早顶点；从栈顶弹到 $u$ 的顶点互相可达，且不能再与栈中更早部分合并，因此恰好组成一个 SCC。

## 真实题目：P3387 缩点

[洛谷 P3387【模板】缩点](https://www.luogu.com.cn/problem/P3387) 给每个点一个权值，求有向路径能够获得的最大点权和。先求 SCC 并累加分量权值，再在缩点 DAG 上按拓扑序做最长路 DP。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    cin >> n >> m;
    vector<long long> weight(n + 1);
    for (int vertex = 1; vertex <= n; ++vertex) cin >> weight[vertex];
    vector<vector<int>> graph(n + 1);
    vector<pair<int, int>> edges;
    for (int edge = 0; edge < m; ++edge) {
        int from, to;
        cin >> from >> to;
        graph[from].push_back(to);
        edges.push_back({from, to});
    }

    vector<int> dfn(n + 1), low(n + 1), component(n + 1), stackVertex;
    vector<bool> inStack(n + 1, false);
    int timer = 0, componentCount = 0;
    function<void(int)> tarjan = [&](int u) {
        dfn[u] = low[u] = ++timer;
        stackVertex.push_back(u);
        inStack[u] = true;
        for (int v : graph[u]) {
            if (dfn[v] == 0) {
                tarjan(v);
                low[u] = min(low[u], low[v]);
            } else if (inStack[v]) {
                low[u] = min(low[u], dfn[v]);
            }
        }
        if (low[u] != dfn[u]) return;
        ++componentCount;
        while (true) {
            int v = stackVertex.back();
            stackVertex.pop_back();
            inStack[v] = false;
            component[v] = componentCount;
            if (v == u) break;
        }
    };
    for (int vertex = 1; vertex <= n; ++vertex) {
        if (dfn[vertex] == 0) tarjan(vertex);
    }

    vector<long long> componentWeight(componentCount + 1);
    for (int vertex = 1; vertex <= n; ++vertex) {
        componentWeight[component[vertex]] += weight[vertex];
    }
    vector<vector<int>> dag(componentCount + 1);
    vector<int> indegree(componentCount + 1);
    for (auto [from, to] : edges) {
        int x = component[from], y = component[to];
        if (x == y) continue;
        dag[x].push_back(y);
        ++indegree[y];
    }

    queue<int> ready;
    vector<long long> best = componentWeight;
    for (int node = 1; node <= componentCount; ++node) {
        if (indegree[node] == 0) ready.push(node);
    }
    long long answer = 0;
    while (!ready.empty()) {
        int u = ready.front();
        ready.pop();
        answer = max(answer, best[u]);
        for (int v : dag[u]) {
            best[v] = max(best[v], best[u] + componentWeight[v]);
            if (--indegree[v] == 0) ready.push(v);
        }
    }
    cout << answer << '\n';
    return 0;
}
```

任意原图路径进入同一 SCC 后可以访问其中所有需要的点，再从某条出边离开；缩点把内部互达选择压成一个 DAG 节点。Tarjan 与建图均为 $O(n+m)$，DAG DP 也是 $O(n+m)$，总空间 $O(n+m)$。缩点重边不会破坏最大值转移，但在其他计数问题中可能需要去重。

## 易错点

- 用已经出栈顶点的 `dfn` 更新 `low`。
- 把无向图桥算法的父边规则套进 SCC。
- 只从 1 号点启动 Tarjan，漏掉不连通部分。
- 缩点后仍按原图顺序 DP，没有使用拓扑序。
