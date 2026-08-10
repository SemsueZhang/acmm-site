# 树的直径

树的直径是树上长度最大的简单路径。无权树以边数计长；带权树以边权和计长，必须先说明边权是否非负。

## 两次搜索

在非负权树上：

1. 从任意节点 $s$ 出发，找到距离最远的节点 $a$；
2. 从 $a$ 再搜索一次，找到最远节点 $b$；
3. $a$ 到 $b$ 的路径是一条直径。

直观上，若从 $s$ 到某条直径的路径在节点 $t$ 接入，那么直径两个端点中至少有一个不会比当前选到的分支更近；沿更远方向走最终能到达某条直径端点。第二次搜索从端点出发，最远节点必是另一端点。

更形式化地，可把树按从 $s$ 的距离分层。设 $u,v$ 是任意一条直径端点，比较 $s$ 到 $u,v$ 的公共路径与分叉部分，可以证明离 $s$ 最远的节点可以替换为某条直径端点而不缩短直径。第二次同理得到另一端点。

该证明依赖边权非负。若允许负权，走得更远可能让路径权值变小，两次搜索结论不能直接使用；此时应使用树形 DP，并明确是否允许空路径。

## 执行过程

因为树上两点路径唯一，无需 Dijkstra：一次 DFS/BFS 累加边权即可。若还要恢复直径路径，在第二次搜索中记录父节点，从 $b$ 逆推到 $a$。

## 例题：带权树直径模板

[洛谷 U212296 树的直径（带权）](https://www.luogu.com.cn/problem/U212296) 给出一棵正权树并询问直径长度。正权保证两次搜索适用。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Edge {
    int to;
    long long weight;
};

pair<int, long long> farthestFrom(
        int start, const vector<vector<Edge>>& graph) {
    vector<long long> distance(graph.size(), -1);
    stack<int> pending;
    pending.push(start);
    distance[start] = 0;

    int farthest = start;
    while (!pending.empty()) {
        int vertex = pending.top();
        pending.pop();
        if (distance[vertex] > distance[farthest]) farthest = vertex;
        for (const Edge& edge : graph[vertex]) {
            if (distance[edge.to] != -1) continue;
            distance[edge.to] = distance[vertex] + edge.weight;
            pending.push(edge.to);
        }
    }
    return {farthest, distance[farthest]};
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int vertexCount;
    cin >> vertexCount;
    vector<vector<Edge>> graph(vertexCount + 1);
    for (int i = 1; i < vertexCount; ++i) {
        int u, v;
        long long weight;
        cin >> u >> v >> weight;
        graph[u].push_back({v, weight});
        graph[v].push_back({u, weight});
    }

    int endpoint = farthestFrom(1, graph).first;
    cout << farthestFrom(endpoint, graph).second << '\n';
    return 0;
}
```

两次搜索都访问每个点和每条边常数次，时间 $O(n)$、空间 $O(n)$。代码使用显式栈，避免深链递归导致系统栈溢出。

## 迁移与易错点

- “直径一定经过根”是错误的；任意根只用于组织遍历。
- 一棵树可能有多条直径，算法只需找出其中一条。
- [洛谷 P1099 树网的核](https://www.luogu.com.cn/problem/P1099) 会继续在直径上选择受长度限制的子段；这要求恢复主链并处理挂在主链外的深度，不是求出直径长度就结束。
- 一般图可能有环且路径不唯一，不能套用树的两次搜索证明。
