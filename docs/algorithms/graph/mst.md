# 最小生成树

给定带权无向连通图，生成树用 $n-1$ 条边连接所有点且无环；边权和最小的生成树叫最小生成树（MST）。它最小化的是总边权，不保证任意两点路径都最短。

本文只讨论无向图。图不连通时不存在覆盖全部顶点的生成树，但同样的算法可以得到最小生成森林。学习重点是割性质如何证明每次贪心选择安全，而不是只记“边排序后用并查集”。

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

严格地说，设当前已选边集 $F$ 能扩充成一棵最小生成树 $T$，Kruskal 下一条边 $e$ 连接两个不同连通块。若 $e\in T$，结论成立；否则把 $e$ 加入 $T$ 会形成一个环，环上必有另一条跨过同一割的边 $f$。由于 $e$ 是尚未选择的最轻跨割边，$w(e)\le w(f)$。用 $e$ 替换 $f$ 后仍是生成树，权值不增且包含 $F\cup\{e\}$。归纳可知所有选择都能扩充成某棵 MST。

## Prim：从点集向外扩张

维护已在生成树中的点集，每次选择一条连接点集内外的最轻边。邻接表加堆为 $O(m\log n)$；邻接矩阵朴素实现为 $O(n^2)$，适合稠密小图。

## 手算例题

边为 `1-2(1), 2-3(2), 1-3(4), 3-4(1), 2-4(5)`。Kruskal 依次选 `1-2(1)`、`3-4(1)`、`2-3(2)`，此时四点连通，总权值 4；`1-3(4)` 会成环而跳过。

## 最小生成森林

图不连通时不存在生成树，但 Kruskal 仍会在每个连通分量内得到一棵 MST，合起来称最小生成森林。题目若要求连接全部点，必须检查最终选边数是否为 $n-1$。

## 真题：P3366 最小生成树

[洛谷 P3366【模板】最小生成树](https://www.luogu.com.cn/problem/P3366) 要求输出最小生成树权值；若图不连通则输出 `orz`。Kruskal 的完整实现如下。

边对应候选连接，DSU 的连通块对应当前森林分量；`unite` 成功恰好表示加入该边不会成环。最终使用边数等于 $n-1$ 时，森林连通且无环，因此是生成树；不足则原图不连通。

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
