# 欧拉道路与 Hierholzer 算法

欧拉道路经过图中每条边恰好一次；起点与终点相同时称为欧拉回路。它关心的是边，不是“每个顶点恰好一次”的哈密顿路。本页讨论无向图与有向图的存在条件，并完整实现有向图欧拉路径。

## 先从度数推导必要条件

在一条无向欧拉道路中，每次到达中间顶点后都必须沿另一条未使用边离开，所以中间点使用的边两两配对，度数为偶数。只有起点和终点可以各留下一个未配对边：

- 欧拉回路：所有非孤立点度数为偶数；
- 非回路的欧拉道路：恰有两个奇度点，它们分别是起点和终点。

有向图把“进入”与“离开”分别计数：回路中每点入度等于出度；非回路中起点出度比入度大 1，终点入度比出度大 1，其余相等。

度数条件还不充分。忽略孤立点后，所有需要使用的边必须位于同一个连通部分；实现中最稳妥的检查是最终路径是否包含恰好 $m+1$ 个顶点。

## 为什么贪心一直走可能卡住

若随意走边并在到达终点时立即结束，可能过早封闭一个小环，把其他边留在外面。Hierholzer 算法允许先形成局部回路，再在回路上的顶点处插入尚未使用边形成的回路。

递归写法更简洁：从 $u$ 不断取未使用出边递归；直到 $u$ 再也没有边可走时，才把 $u$ 放进答案。这个后序序列是路径的逆序，最后需要反转。

```cpp
void buildEulerTrail(int u) {
    while (nextEdge[u] < static_cast<int>(graph[u].size())) {
        int v = graph[u][nextEdge[u]++];
        buildEulerTrail(v);
    }
    reversedTrail.push_back(u);
}
```

## 正确性

每条边在取出时恰好使用一次。一个顶点加入逆序答案时，它已经没有未使用出边，因此在之后构造出的正序路径中，该顶点后方需要的边都已安排完成。递归返回时，每段答案都是首尾可拼接的闭合段或起终点段；把这些段按返回顺序拼接，得到使用全部已访问边的连续道路。

若最终记录了 $m+1$ 个顶点，则相邻顶点对应的 $m$ 次移动恰好覆盖全部边；反之长度不足说明有边无法从所选起点纳入同一条道路，度数条件或连通条件至少一个未满足。

每条边只被取出一次，排序外的时间为 $O(n+m)$，邻接表、递归栈与答案占 $O(n+m)$。

## 手算例子

有向边 `1→2, 2→1, 1→3`。从 1 走到 2、回到 1，再走到 3；后序加入顺序为 `3,1,2,1`，反转得到 `1,2,1,3`。起点 1 的出度比入度大 1，终点 3 的入度比出度大 1。

## 真实题目：P7771 欧拉路径

[洛谷 P7771【模板】欧拉路径](https://www.luogu.com.cn/problem/P7771) 给出有向图，要求字典序最小的欧拉路径。先根据入度差确定起点；若所有点入度等于出度，则从有出边的最小编号点开始。每个邻接表逆序排序并从末尾取边，可按从小到大的顺序遍历。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    cin >> n >> m;
    vector<vector<int>> graph(n + 1);
    vector<int> indegree(n + 1), outdegree(n + 1);
    for (int edge = 0; edge < m; ++edge) {
        int from, to;
        cin >> from >> to;
        graph[from].push_back(to);
        ++outdegree[from];
        ++indegree[to];
    }

    int start = -1, finish = -1;
    bool degreeValid = true;
    for (int vertex = 1; vertex <= n; ++vertex) {
        int difference = outdegree[vertex] - indegree[vertex];
        if (difference == 1 && start == -1) start = vertex;
        else if (difference == -1 && finish == -1) finish = vertex;
        else if (difference != 0) degreeValid = false;
    }
    if ((start == -1) != (finish == -1)) degreeValid = false;
    if (start == -1) {
        start = 1;
        while (start <= n && graph[start].empty()) ++start;
        if (start > n) start = 1;
    }

    for (int vertex = 1; vertex <= n; ++vertex) {
        sort(graph[vertex].rbegin(), graph[vertex].rend());
    }

    vector<int> stackVertex{start}, reversedTrail;
    while (!stackVertex.empty()) {
        int u = stackVertex.back();
        if (!graph[u].empty()) {
            int v = graph[u].back();
            graph[u].pop_back();
            stackVertex.push_back(v);
        } else {
            reversedTrail.push_back(u);
            stackVertex.pop_back();
        }
    }

    if (!degreeValid || static_cast<int>(reversedTrail.size()) != m + 1) {
        cout << "No\n";
        return 0;
    }
    reverse(reversedTrail.begin(), reversedTrail.end());
    for (int vertex : reversedTrail) cout << vertex << ' ';
    cout << '\n';
    return 0;
}
```

排序复杂度总计 $O(m\log m)$，构造为 $O(n+m)$，空间 $O(n+m)$。若题目不要求字典序，可以省去排序。

## 易错点

- 只检查度数，不检查是否真正使用全部边。
- 在进入顶点时输出，而不是无边可走时后序加入。
- 无向边的两个方向没有共享边编号，导致同一条边使用两次。
- 欧拉回路情形随意从孤立点开始。
