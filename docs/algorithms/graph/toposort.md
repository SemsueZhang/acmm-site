# 拓扑排序

拓扑序是 DAG 顶点的一种线性排列，使每条有向边 $u\to v$ 中，$u$ 都出现在 $v$ 前。只有 DAG 才存在拓扑序。

## Kahn 算法

入度为 0 的点没有未完成前置条件，可以立即取出；删除它的所有出边，可能产生新的入度 0 点。

```cpp
vector<int> indegree(n + 1);
queue<int> q;
for (int u = 1; u <= n; ++u)
    if (indegree[u] == 0) q.push(u);

vector<int> order;
while (!q.empty()) {
    int u = q.front(); q.pop();
    order.push_back(u);
    for (int v : g[u])
        if (--indegree[v] == 0) q.push(v);
}
if ((int)order.size() < n) cout << "Cycle\n";
```

时间 $O(n+m)$。若最后不足 $n$ 个点，剩余部分每个点都有未删除入边，说明存在有向环。

### 例题：课程安排

依赖 `A→C, B→C, C→D`。初始 A、B 入度为 0，可先任选其一；二者都处理后 C 入度才变为 0，最后 D。因此 `A,B,C,D` 和 `B,A,C,D` 都合法。拓扑序通常不唯一。

若要求字典序最小拓扑序，把普通队列换成小根堆。

## DAG 上动态规划

拓扑序保证处理 $v$ 前，它的所有前驱已处理。最长路（无正环问题，因为 DAG 无环）可写为：

```cpp
for (int u : order)
    for (auto [v, w] : g[u])
        dp[v] = max(dp[v], dp[u] + w);
```

需要区分“不可达”的点，不能统一初始化为 0 后错误地从不可达点转移。

## DFS 做法

DFS 退出一个点时把它加入序列，最后反转得到拓扑序。用三色标记：0 未访问、1 在递归栈、2 已完成；遇到指向颜色 1 的边即发现环。

## 真题：P4017 最大食物链计数

[洛谷 P4017 最大食物链计数](https://www.luogu.com.cn/problem/P4017) 把“被吃者指向捕食者”看成有向边。生产者入度为 0，食物链在消费者出度为 0 处结束。定义 `ways[u]` 为到达 $u$ 的食物链条数，按拓扑序转移：

$$
ways[v]\mathrel{+}=ways[u].
$$

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    const int MOD = 80112002;
    int n, m;
    cin >> n >> m;
    vector<vector<int>> graph(n + 1);
    vector<int> indegree(n + 1), outdegree(n + 1), ways(n + 1);
    while (m--) {
        int from, to;
        cin >> from >> to;
        graph[from].push_back(to);
        ++outdegree[from];
        ++indegree[to];
    }

    queue<int> ready;
    for (int vertex = 1; vertex <= n; ++vertex) {
        if (indegree[vertex] == 0) {
            ready.push(vertex);
            ways[vertex] = 1;
        }
    }

    while (!ready.empty()) {
        int u = ready.front(); ready.pop();
        for (int v : graph[u]) {
            ways[v] = (ways[v] + ways[u]) % MOD;
            if (--indegree[v] == 0) ready.push(v);
        }
    }

    int answer = 0;
    for (int vertex = 1; vertex <= n; ++vertex)
        if (outdegree[vertex] == 0)
            answer = (answer + ways[vertex]) % MOD;
    cout << answer << '\n';
    return 0;
}
```

每条边只转移一次，时间 $O(n+m)$、空间 $O(n+m)$。若图中可能有环，处理点数不足 $n$ 时还应报告无合法拓扑序；本题的食物网保证相应结构可计数。

## 易错点

- 无向图不存在通常意义下的拓扑序。
- 入度数组计算漏边，或多次运行后未恢复。
- 看到队列只剩一个点就误判拓扑序唯一；唯一性要在每一步检查可选入度 0 点是否恰有一个。
- DAG DP 没有按拓扑序处理。
