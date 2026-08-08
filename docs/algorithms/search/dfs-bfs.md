# DFS、BFS 与泛洪

## 状态空间

设计搜索前写清：

1. 一个状态包含哪些信息？
2. 从状态能产生哪些下一状态？
3. 怎样判重？
4. 什么时候得到答案？

状态缺少信息会把本应不同的情况错误合并；状态包含无关信息又会导致空间爆炸。

## DFS：选择、递归、撤销

### 例题：从 $n$ 个数中选 $k$ 个

`dfs(pos, chosen)` 决定位置 `pos` 是否选择。更紧凑的写法直接枚举下一次选择的位置，并保证下标递增，从结构上避免重复排列。

```cpp
vector<int> path;
void dfs(int start) {
    if ((int)path.size() == k) {
        // 使用一个组合
        return;
    }
    int need = k - path.size();
    for (int i = start; i <= n - need + 1; ++i) {
        path.push_back(i);       // 选择
        dfs(i + 1);              // 递归
        path.pop_back();         // 撤销
    }
}
```

上界 `n-need+1` 是可行性剪枝：剩余元素不够时无需继续。

## BFS：按距离分层

边权相同的状态图中，BFS 第一次到达状态时使用的边数最少。必须在**入队时**标记访问，防止多个父状态重复加入同一节点。

### 例题：网格最短路

```cpp
queue<pair<int,int>> q;
vector<vector<int>> dist(n, vector<int>(m, -1));
dist[sx][sy] = 0;
q.push({sx, sy});
while (!q.empty()) {
    auto [x, y] = q.front(); q.pop();
    for (int d = 0; d < 4; ++d) {
        int nx = x + dx[d], ny = y + dy[d];
        if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
        if (blocked[nx][ny] || dist[nx][ny] != -1) continue;
        dist[nx][ny] = dist[x][y] + 1;
        q.push({nx, ny});
    }
}
```

若要恢复路径，首次访问 `(nx,ny)` 时记录其前驱 `(x,y)`，从终点反向追溯再翻转。

## 泛洪（Flood Fill）

泛洪从一个格子出发，把上下左右相连且满足条件的格子全部访问，本质上是网格上的 DFS/BFS。统计岛屿数量时，每遇到一个未访问陆地就启动泛洪，启动次数就是连通块数。

## 复杂度

显式图遍历为 $O(n+m)$。隐式状态搜索应写成“状态数 × 每状态转移数”，不能只看代码循环。若状态是一个长度 $n$ 的排列，状态数可能达到 $n!$。

## 真题：P1443 马的遍历

[洛谷 P1443 马的遍历](https://www.luogu.com.cn/problem/P1443) 要求棋盘上一个马到每个格子的最少步数。每次移动代价都是 1，因此第一次 BFS 到达某格时，距离已经最短。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int rows, columns, startX, startY;
    cin >> rows >> columns >> startX >> startY;
    --startX; --startY;

    vector<vector<int>> distance(rows, vector<int>(columns, -1));
    queue<pair<int,int>> states;
    distance[startX][startY] = 0;
    states.push({startX, startY});

    int dx[8] = {-2,-2,-1,-1,1,1,2,2};
    int dy[8] = {-1,1,-2,2,-2,2,-1,1};
    while (!states.empty()) {
        auto [x, y] = states.front();
        states.pop();
        for (int direction = 0; direction < 8; ++direction) {
            int nextX = x + dx[direction];
            int nextY = y + dy[direction];
            if (nextX < 0 || nextX >= rows ||
                nextY < 0 || nextY >= columns) continue;
            if (distance[nextX][nextY] != -1) continue;
            distance[nextX][nextY] = distance[x][y] + 1;
            states.push({nextX, nextY});
        }
    }

    for (const auto& row : distance) {
        for (int value : row) cout << left << setw(5) << value;
        cout << '\n';
    }
    return 0;
}
```

棋盘共有 $nm$ 个状态，每个状态尝试 8 次转移，时间和空间都是 $O(nm)$。

## 易错点

- DFS 修改全局状态后忘记撤销。
- BFS 出队时才判重，队列膨胀。
- 用普通 BFS 处理不同权值的边。
- 网格坐标行列混用或越界检查在访问数组之后。
- 递归深度过大导致栈溢出。
