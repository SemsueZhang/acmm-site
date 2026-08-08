# 最小生成树

给定带权无向连通图，生成树用 $n-1$ 条边连接所有点且无环；边权和最小的生成树叫最小生成树（MST）。它最小化的是总边权，不保证任意两点路径都最短。

## Kruskal：从边出发

按边权升序扫描，只选择连接两个不同连通块的边。并查集负责判断是否成环。

```cpp
struct Edge { int u, v; long long w; };
sort(edges.begin(), edges.end(), [](const Edge& a, const Edge& b) {
    return a.w < b.w;
});
DSU dsu(n);
long long answer = 0;
int used = 0;
for (auto [u, v, w] : edges) {
    if (dsu.unite(u, v)) {
        answer += w;
        ++used;
    }
}
if (used != n - 1) cout << "No spanning tree\n";
else cout << answer << '\n';
```

复杂度由排序主导，为 $O(m\log m)$。

### 为什么贪心正确：割性质

把顶点分成两个非空集合称为一个割。跨过某个割的最轻边一定存在于某棵 MST 中。Kruskal 当前的连通块形成割，它选择的最轻可用边可安全加入；这就是交换论证在图上的形式。

## Prim：从点集向外扩张

维护已在生成树中的点集，每次选择一条连接点集内外的最轻边。邻接表加堆为 $O(m\log n)$；邻接矩阵朴素实现为 $O(n^2)$，适合稠密小图。

## 手算例题

边为 `1-2(1), 2-3(2), 1-3(4), 3-4(1), 2-4(5)`。Kruskal 依次选 `1-2(1)`、`3-4(1)`、`2-3(2)`，此时四点连通，总权值 4；`1-3(4)` 会成环而跳过。

## 最小生成森林

图不连通时不存在生成树，但 Kruskal 仍会在每个连通分量内得到一棵 MST，合起来称最小生成森林。题目若要求连接全部点，必须检查最终选边数是否为 $n-1$。

## 真题：P3366 最小生成树

[洛谷 P3366【模板】最小生成树](https://www.luogu.com.cn/problem/P3366) 要求输出最小生成树权值；若图不连通则输出 `orz`。Kruskal 的完整实现如下。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Edge { int from, to, weight; };

struct DSU {
    vector<int> parent, size;
    explicit DSU(int n) : parent(n + 1), size(n + 1, 1) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        return parent[x] == x ? x : parent[x] = find(parent[x]);
    }
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;
        if (size[a] < size[b]) swap(a, b);
        parent[b] = a;
        size[a] += size[b];
        return true;
    }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    cin >> n >> m;
    vector<Edge> edges(m);
    for (Edge& edge : edges) cin >> edge.from >> edge.to >> edge.weight;
    sort(edges.begin(), edges.end(), [](const Edge& a, const Edge& b) {
        return a.weight < b.weight;
    });

    DSU dsu(n);
    long long answer = 0;
    int usedEdges = 0;
    for (Edge edge : edges) {
        if (!dsu.unite(edge.from, edge.to)) continue;
        answer += edge.weight;
        if (++usedEdges == n - 1) break;
    }

    if (usedEdges == n - 1) cout << answer << '\n';
    else cout << "orz\n";
    return 0;
}
```

排序为 $O(m\log m)$，并查集操作总计近似 $O(m)$，空间 $O(n+m)$。

## 易错点

- 在有向图上套 MST。
- 忘记连通性检查。
- 权值和用 `int` 溢出。
- 把 Prim 的“最小入边”与 Dijkstra 的“源点最短距离”混淆。
