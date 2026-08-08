# 图的概念、存储与遍历

图 $G=(V,E)$ 由顶点集合与边集合组成。无向边连接两个点；有向边只允许沿指定方向通过；带权边还携带距离、费用等信息。

## 必须分清的概念

- **度**：无向图中与点相连的边数；有向图分入度和出度。
- **路径与环**：路径是相邻边序列；起终点相同且至少含一条边时形成环。
- **连通分量**：无向图中互相可达的最大点集。
- **强连通分量**：有向图中任意两点都互相可达的最大点集。
- **树**：连通且无环的无向图，$n$ 个点恰有 $n-1$ 条边。
- **DAG**：无环有向图。

## 存储

邻接矩阵 `g[u][v]` 查询某条边 $O(1)$，但空间 $O(n^2)$，适合点少或稠密图。邻接表只存存在的边，空间 $O(n+m)$，遍历所有边也为 $O(n+m)$，竞赛中更常用。

```cpp
struct Edge { int to; long long weight; };
vector<vector<Edge>> g(n + 1);
g[u].push_back({v, w});
g[v].push_back({u, w}); // 仅无向图需要反向边
```

若有重边，不能默认后读入的边覆盖前面的边；自环也可能影响度数和环的判断。

## DFS 与 BFS

DFS 沿一条路走到底再回溯，适合连通块、环、树形递归；BFS 按层扩展，适合无权最短路。邻接表中遍历一次整张图，二者都是 $O(n+m)$。

### 例题：统计连通分量

对每个未访问点启动一次 DFS，每次启动恰对应一个新连通分量。

```cpp
void dfs(int u) {
    visited[u] = true;
    for (auto e : g[u])
        if (!visited[e.to]) dfs(e.to);
}

int components = 0;
for (int u = 1; u <= n; ++u) {
    if (!visited[u]) {
        ++components;
        dfs(u);
    }
}
```

图可能不连通，因此只从 1 号点遍历通常不够。无向图判环时还要记录父边编号；只记录父节点会在重边图上出错。

## 稀疏图

当 $m$ 远小于 $n^2$ 时称为稀疏图。提高组大图通常稀疏，应使用邻接表和 $O((n+m)\log n)$ 一类算法，而不是邻接矩阵与 $O(n^2)$ 扫描。

## 真题：P1596 Lake Counting

[洛谷 P1596 Lake Counting](https://www.luogu.com.cn/problem/P1596) 把每个有水格子看成顶点，八方向相邻看成边，问题就变成统计连通分量。每发现一个尚未访问的 `W`，启动一次 BFS 并把整个水域标记掉。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int rows, columns;
    cin >> rows >> columns;
    vector<string> grid(rows);
    for (string& row : grid) cin >> row;

    int dx[8] = {-1,-1,-1,0,0,1,1,1};
    int dy[8] = {-1,0,1,-1,1,-1,0,1};
    int components = 0;

    for (int x = 0; x < rows; ++x) {
        for (int y = 0; y < columns; ++y) {
            if (grid[x][y] != 'W') continue;
            ++components;
            queue<pair<int,int>> cells;
            cells.push({x, y});
            grid[x][y] = '.';
            while (!cells.empty()) {
                auto [currentX, currentY] = cells.front();
                cells.pop();
                for (int direction = 0; direction < 8; ++direction) {
                    int nextX = currentX + dx[direction];
                    int nextY = currentY + dy[direction];
                    if (nextX < 0 || nextX >= rows ||
                        nextY < 0 || nextY >= columns) continue;
                    if (grid[nextX][nextY] != 'W') continue;
                    grid[nextX][nextY] = '.';
                    cells.push({nextX, nextY});
                }
            }
        }
    }
    cout << components << '\n';
    return 0;
}
```

每个格子最多入队一次，时间和空间都是 $O(rows\times columns)$。若题目只允许四方向相邻，方向数组必须相应修改。

## 易错点

- 无向边忘记加入两个方向，或误加两次导致四条边。
- 多组数据没有清空邻接表与访问数组。
- 深链图递归 DFS 栈溢出，必要时改显式栈。
- 边数是 $m$，无向邻接表中存储项却有 $2m$ 个。
