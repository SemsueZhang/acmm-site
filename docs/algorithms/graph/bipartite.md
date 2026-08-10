# 二分图判定与二染色

若无向图的顶点可以分成两组，使每条边的两个端点分属不同组，则称它为二分图。二染色既是定义的直接执行方式，也能给出“无奇环”等价条件的构造性证明。

## 定理：二分图当且仅当不存在奇环

必要性：沿任意环依次交替颜色。若环长为奇数，走一圈后起点会被要求同时取两种颜色，矛盾。

充分性：对每个连通分量任选根，按到根距离的奇偶染色。若存在一条边连接同色点，把两端到最近公共祖先的树路径与该边合并，就会形成奇环；题设排除了这种情况，所以每条边两端颜色都不同。

这个证明直接导出算法：广度优先搜索（Breadth-First Search，BFS）或深度优先搜索维护 0/1 颜色，遇到同色边即判定失败。

```cpp
bool colorComponent(int start, const vector<vector<int>>& graph,
                    vector<int>& color, array<int, 2>& count) {
    queue<int> waiting;
    color[start] = 0;
    ++count[0];
    waiting.push(start);
    while (!waiting.empty()) {
        int u = waiting.front();
        waiting.pop();
        for (int v : graph[u]) {
            if (color[v] == -1) {
                color[v] = color[u] ^ 1;
                ++count[color[v]];
                waiting.push(v);
            } else if (color[v] == color[u]) {
                return false;
            }
        }
    }
    return true;
}
```

## 为什么每个连通分量有两种染色

连通分量中任选一个点颜色后，沿路径传播会唯一确定所有点颜色；交换 0 与 1 得到另一种方案。因此恰有两种整体互换的染色。不同连通分量之间没有边，可以分别选择方向。

搜索访问每个点一次、检查每条边常数次，时间 $O(n+m)$，邻接表、颜色和队列占 $O(n+m)$ 空间。

## 手算例子

四边形 `1-2-3-4-1` 可染成 `{1,3}` 与 `{2,4}`。若再加入边 `1-3`，1 和 3 被迫同色却相邻，边 `1-2-3-1` 构成长度 3 的奇环，因此不再是二分图。

## 真实题目：P1330 封锁阳光大学

[洛谷 P1330 封锁阳光大学](https://www.luogu.com.cn/problem/P1330) 要使每条道路两端分属不同阵营，并最小化选择人数。对每个含边连通分量二染色后，两种合法选择正是选颜色 0 或颜色 1，取人数较少者；孤立点没有道路需要封锁，贡献 0。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    cin >> n >> m;
    vector<vector<int>> graph(n + 1);
    for (int edge = 0; edge < m; ++edge) {
        int u, v;
        cin >> u >> v;
        graph[u].push_back(v);
        graph[v].push_back(u);
    }

    vector<int> color(n + 1, -1);
    int answer = 0;
    for (int start = 1; start <= n; ++start) {
        if (color[start] != -1) continue;
        array<int, 2> count{0, 0};
        queue<int> waiting;
        color[start] = 0;
        ++count[0];
        waiting.push(start);
        bool valid = true;
        while (!waiting.empty() && valid) {
            int u = waiting.front();
            waiting.pop();
            for (int v : graph[u]) {
                if (color[v] == -1) {
                    color[v] = color[u] ^ 1;
                    ++count[color[v]];
                    waiting.push(v);
                } else if (color[v] == color[u]) {
                    valid = false;
                    break;
                }
            }
        }
        if (!valid) {
            cout << "Impossible\n";
            return 0;
        }
        answer += min(count[0], count[1]);
    }
    cout << answer << '\n';
    return 0;
}
```

每个连通分量独立取较小颜色类，所以局部最优之和就是全局最优。时间 $O(n+m)$、空间 $O(n+m)$。

## 易错点

- 只从 1 号点染色，漏掉其他连通分量。
- 没有区分“严格不同颜色”与其他关系约束。
- 把二分图判定和二分图最大匹配混为一谈；后者不属于本页。
- 对孤立点强制选择一人，导致答案偏大。
