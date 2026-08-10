# 并查集

并查集维护若干互不相交的集合，支持“合并两个集合”和“判断两个元素是否同属一集合”。它不擅长删除边，也不能直接给出两点间路径。

本文只讨论普通、不可撤销的并查集（Disjoint Set Union，DSU）。核心表示不变量是：每个元素沿 `parent` 指针最终到达唯一根，两个元素属于同一集合当且仅当根相同；`size` 只在根节点处表示整个集合大小。

## 森林表示

每个集合是一棵树，根代表集合。`find(x)` 沿父指针找到根；合并时把一个根接到另一个根。

两项优化缺一不可：

- **路径压缩**：查找后把路径上的点直接连向根。
- **按大小/秩合并**：把小树接到大树，避免树变成长链。

```cpp
struct DSU {
    vector<int> parent, size;
    DSU(int n) : parent(n + 1), size(n + 1, 1) {
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
    bool same(int a, int b) { return find(a) == find(b); }
};
```

单次操作均摊复杂度 $O(\alpha(n))$，反阿克曼函数增长极慢，竞赛规模下可视作常数。

路径压缩只把节点改连到同一棵树的根，不会改变集合划分；按大小合并也只连接两个根，因此不会把一个集合拆开。两项优化共同得到 $O(\alpha(n))$ 的均摊界，这一结论是多次操作总量的保证，不表示每一次递归都严格为常数时间。

## 例题：动态连通块

初始有 $n$ 个孤立点，两类操作：连接 $a,b$；询问 $a,b$ 是否连通。连接时 `unite(a,b)`，询问时比较根。若还要维护连通块数量，初始 `components=n`，仅当 `unite` 返回 `true` 时减一。

## 扩展：带关系并查集

若题目维护“朋友/敌人”这类相对关系，可令 `relation[x]` 表示 $x$ 到父亲的关系，并在路径压缩时累加（或异或）关系。核心仍是：集合根确定归属，根路径上的附加量确定相对关系。

## 真题：P3367 并查集

[洛谷 P3367【模板】并查集](https://www.luogu.com.cn/problem/P3367) 包含合并集合与查询连通性两类操作。它没有删边，也不要求输出路径，因此并查集比每次 DFS 更合适。

把已经加入的无向边看成等价关系：每次连接合并两个等价类，询问只比较代表元。正向看，同根说明存在由已合并关系连接的链；反向看，每次合并只发生在题目明确连接的两个集合之间，所以不会把本不连通的集合无依据地合并。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, size;
    explicit DSU(int n) : parent(n + 1), size(n + 1, 1) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        return parent[x] == x ? x : parent[x] = find(parent[x]);
    }
    void unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return;
        if (size[a] < size[b]) swap(a, b);
        parent[b] = a;
        size[a] += size[b];
    }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    cin >> n >> m;
    DSU dsu(n);
    while (m--) {
        int type, x, y;
        cin >> type >> x >> y;
        if (type == 1) dsu.unite(x, y);
        else cout << (dsu.find(x) == dsu.find(y) ? 'Y' : 'N') << '\n';
    }
    return 0;
}
```

$m$ 次操作总复杂度 $O(m\alpha(n))$，空间 $O(n)$。

## 易错点

- 直接写 `parent[a]=b`，没有先找根。
- 重复合并同一集合时仍修改大小或连通块数量。
- 路径压缩后误把 `size[x]` 当作每个节点都有效；通常只有根的 `size` 有意义。
- 在需要撤销或在线删边的题中直接使用普通并查集。
