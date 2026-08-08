# 搜索进阶

高级搜索不是换一个模板，而是减少状态数、避免重复状态，或改变扩展顺序以更早接近答案。

## 剪枝

- **可行性剪枝**：剩余资源已不可能完成目标。
- **最优性剪枝**：当前代价加理论最好下界仍不优于已有答案。
- **重复性剪枝**：同层相同选择只尝试一次。
- **搜索顺序优化**：先尝试限制多、希望大的分支，尽早获得好答案以加强剪枝。

### 例题：装箱最少箱数

物品按重量降序放入容量固定的箱子。DFS 枚举放入已有箱或新箱；若当前箱数已不小于最优答案就停止。同一层中，剩余容量相同的箱子等价，只试一个。

降序不是正确性的要求，但大物品更难安置，先放能显著减少分支。

## 记忆化搜索

若同一状态会从多条路径到达，把计算结果缓存。它等价于自顶向下 DP。

```cpp
long long solve(int state) {
    if (isBase(state)) return baseValue(state);
    if (memo.count(state)) return memo[state];
    long long ans = INF;
    for (int next : transitions(state))
        ans = min(ans, cost(state, next) + solve(next));
    return memo[state] = ans;
}
```

状态中必须包含所有会影响后续答案的信息，否则缓存会把不同问题错误合并。存在环时还需访问状态或改用图最短路，不能无限递归。

## 双向 BFS

起点、终点明确且操作可逆时，同时从两端 BFS，在中间相遇。普通 BFS 深度 $d$、分支数 $b$ 时约访问 $b^d$ 个状态；双向搜索理想情况下约 $2b^{d/2}$。

每轮优先扩展较小的一侧，并用两个距离哈希表判定相遇。若边为有向边，从终点侧必须使用反向边。

### 例题：单词变换

每步改变一个字符且新单词必须在字典中。从起始词和目标词同时扩展，每生成一个新词就检查是否已被另一侧访问；相遇时距离相加即为最少步数。

## 启发式搜索 A*

A* 用 $f(s)=g(s)+h(s)$ 排序：$g$ 是已付出的真实代价，$h$ 是到目标的估计。若 $h$ 从不高估剩余最短代价（可采纳），A* 第一次从优先队列取出目标时得到最优解。

网格四方向单位移动时，忽略障碍的曼哈顿距离是合法的 $h$。`h=0` 时 A* 退化为 Dijkstra。错误地高估可能更快，却不再保证最优。

## 迭代加深

依次限制最大深度为 $0,1,2,\dots$ 做深度受限 DFS。它兼具 DFS 的低内存和 BFS 的最浅解保证，适合分支较少但答案深度未知的状态空间。虽然浅层被重复搜索，但指数树中最后一层通常占绝大多数节点。

## 真题：P1379 八数码难题

[洛谷 P1379 八数码难题](https://www.luogu.com.cn/problem/P1379) 的一个状态是 9 个格子的排列，转移是把 0 与上下左右格交换。普通 BFS 已能覆盖至多 $9!$ 个排列，并保证第一次到达目标时步数最少。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string start;
    cin >> start;
    const string target = "123804765";

    queue<string> states;
    unordered_map<string, int> distance;
    states.push(start);
    distance[start] = 0;

    int dx[4] = {-1, 1, 0, 0};
    int dy[4] = {0, 0, -1, 1};
    while (!states.empty()) {
        string current = states.front();
        states.pop();
        if (current == target) {
            cout << distance[current] << '\n';
            return 0;
        }

        int zero = current.find('0');
        int x = zero / 3, y = zero % 3;
        for (int direction = 0; direction < 4; ++direction) {
            int nextX = x + dx[direction], nextY = y + dy[direction];
            if (nextX < 0 || nextX >= 3 || nextY < 0 || nextY >= 3)
                continue;
            string next = current;
            swap(next[zero], next[nextX * 3 + nextY]);
            if (distance.count(next)) continue;
            distance[next] = distance[current] + 1;
            states.push(next);
        }
    }
    return 0;
}
```

这份代码用于建立正确的状态建模。进一步优化时，可从起点与终点做双向 BFS，或用“每个数字到目标位置的曼哈顿距离之和”作为 A* 启发函数；二者都不改变状态与转移定义。

## 易错点

- 剪枝条件只凭直觉，误剪合法最优解；每条剪枝都要有上下界证明。
- 双向 BFS 两侧距离定义不一致。
- A* 的启发函数高估，仍宣称答案最优。
- 记忆化的“未计算”哨兵与合法答案相同。
- 迭代加深没有避免当前路径上的环。
