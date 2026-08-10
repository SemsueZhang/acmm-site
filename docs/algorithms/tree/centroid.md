# 树的重心

删除节点 $u$ 后，树会分成若干连通块。令 $f(u)$ 为这些连通块大小的最大值，使 $f(u)$ 最小的节点称为树的重心。等价地，重心删除后每个连通块大小都不超过 $n/2$。

## 用一次 DFS 计算

任意选 1 为根，设 `subtreeSize[u]` 为 $u$ 子树大小。删除 $u$ 后：

- 每个孩子 $v$ 方向的连通块大小为 `subtreeSize[v]`；
- 父亲方向的连通块大小为 `n - subtreeSize[u]`。

取这些数的最大值就是 $f(u)$。这个划分完整且互不重叠，因为树上路径唯一，删除 $u$ 后不同邻居方向不可能再连通。

## 为什么重心至多两个

若某个邻居方向包含超过 $n/2$ 个节点，向该方向移动会让最大块严格变小；所以最优点不会停在这种位置。若有两个重心，它们必须相邻，并把树分成两个恰好 $n/2$ 的部分；不可能存在第三个。

## 真题：P1395 会议

[洛谷 P1395 会议](https://www.luogu.com.cn/problem/P1395) 要选择一个会议点，使所有节点到它的距离和最小。无权树中，重心同时具有这个性质：若从 $u$ 沿边走向含 $s$ 个节点的连通块，距离和变化为 $(n-s)-s=n-2s$；只有当 $s>n/2$ 时移动才会更优。因此不存在过半方向的重心就是距离和最小点。

```cpp
#include <bits/stdc++.h>
using namespace std;

int vertexCount;
vector<vector<int>> graph;
vector<int> subtreeSize;
int bestVertex = 1;
int bestLargestPart;

void findCentroid(int vertex, int parent) {
    subtreeSize[vertex] = 1;
    int largestPart = 0;
    for (int next : graph[vertex]) {
        if (next == parent) continue;
        findCentroid(next, vertex);
        subtreeSize[vertex] += subtreeSize[next];
        largestPart = max(largestPart, subtreeSize[next]);
    }
    largestPart = max(largestPart, vertexCount - subtreeSize[vertex]);
    if (largestPart < bestLargestPart ||
        (largestPart == bestLargestPart && vertex < bestVertex)) {
        bestLargestPart = largestPart;
        bestVertex = vertex;
    }
}

long long distanceSum(int start) {
    vector<int> distance(vertexCount + 1, -1);
    queue<int> pending;
    pending.push(start);
    distance[start] = 0;
    long long sum = 0;
    while (!pending.empty()) {
        int vertex = pending.front();
        pending.pop();
        sum += distance[vertex];
        for (int next : graph[vertex]) {
            if (distance[next] != -1) continue;
            distance[next] = distance[vertex] + 1;
            pending.push(next);
        }
    }
    return sum;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> vertexCount;
    graph.assign(vertexCount + 1, {});
    subtreeSize.assign(vertexCount + 1, 0);
    for (int i = 1; i < vertexCount; ++i) {
        int u, v;
        cin >> u >> v;
        graph[u].push_back(v);
        graph[v].push_back(u);
    }

    bestLargestPart = vertexCount;
    findCentroid(1, 0);
    cout << bestVertex << ' ' << distanceSum(bestVertex) << '\n';
    return 0;
}
```

求重心和计算距离各为 $O(n)$，总空间 $O(n)$。若 $n$ 很大且树可能是一条链，应把递归 DFS 改成显式栈。
