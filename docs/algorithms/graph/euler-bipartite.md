# 欧拉路与二分图

## 欧拉道路与欧拉回路

欧拉道路经过每条边恰好一次；起点终点相同则是欧拉回路。注意它关心的是**边**，哈密顿路关心每个顶点一次，两者完全不同。

### 存在条件

忽略孤立点后还需保证相关顶点连通。

- 无向图欧拉回路：所有点度数为偶数。
- 无向图欧拉道路：恰有 0 或 2 个奇度点；有 2 个时它们是起终点。
- 有向图欧拉回路：每点入度等于出度。
- 有向图欧拉道路：除起点出度比入度大 1、终点入度比出度大 1 外，其余相等。

### Hierholzer 算法

从起点不断走未使用边；走不动时把点加入答案并回退，最后反转。每条边只处理一次，$O(n+m)$。

```cpp
vector<int> path;
vector<int> iter(n + 1), used(m);
void dfsEuler(int u) {
    while (iter[u] < (int)g[u].size()) {
        auto [v, edgeId] = g[u][iter[u]++];
        if (used[edgeId]) continue;
        used[edgeId] = true;
        dfsEuler(v);
    }
    path.push_back(u);
}
// 调用后 reverse(path.begin(), path.end())
```

无向边的两个方向必须共享同一个 `edgeId`，否则会被走两次。最终还应检查答案包含 $m+1$ 个顶点。

## 二分图

若顶点可分成两组，使每条边的两个端点分属不同组，则是二分图。一个无向图是二分图，当且仅当它不存在奇环。

### 例题：判定二分图

用 0/1 染色。每条边要求两端颜色相反；若遇到同色相邻点，则从两条 BFS 树路径加该边可构成奇环。

```cpp
vector<int> color(n + 1, -1);
for (int s = 1; s <= n; ++s) if (color[s] == -1) {
    queue<int> q;
    color[s] = 0;
    q.push(s);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : g[u]) {
            if (color[v] == -1) {
                color[v] = color[u] ^ 1;
                q.push(v);
            } else if (color[v] == color[u]) {
                cout << "Not bipartite\n";
                return 0;
            }
        }
    }
}
```

图可能不连通，必须从每个未染色点启动。时间 $O(n+m)$。

## 易错点

- 只检查度数条件，不检查非零度顶点是否连通。
- Hierholzer 在进入边时就输出，正确做法是无边可走时后序加入。
- 二分图只从 1 号点染色，漏掉其他连通分量。
- 把“二分图判定”与“二分图最大匹配”混为一谈；后者在当前大纲中属于 NOI 级算法。
