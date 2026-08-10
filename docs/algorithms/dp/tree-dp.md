# 树形动态规划

树形动态规划（Tree DP）利用树在切断父子边后各子问题互相独立的性质，先计算孩子子树，再合并到父节点。本页以树的最大权独立集为主线；树上背包和换根 DP 只作为下一步方向。

## 为什么需要以子树为状态

树没有天然的从左到右顺序。任取根后，每个节点的孩子子树互不相交，只有通过父节点的选择产生关系，因此“以 $u$ 为根的子树，并规定 $u$ 的状态”是自然子问题。

## 状态与转移

给每个点权值 $w_u$，选择若干点且相邻点不能同时选择，最大化权值和。定义：

- `dp[u][0]`：在 $u$ 的子树中不选择 $u$ 的最大权值；
- `dp[u][1]`：在 $u$ 的子树中选择 $u$ 的最大权值。

初始化 `dp[u][0]=0`、`dp[u][1]=w[u]`。对每个孩子 $v$：

$$
dp[u][0] \mathrel{+}=\max(dp[v][0],dp[v][1]),
$$

$$
dp[u][1] \mathrel{+}=dp[v][0].
$$

选择 $u$ 时孩子都不能选；不选择 $u$ 时，每个孩子可独立选择较优状态。

## 正确性：独立性与完备性

固定 $u$ 是否选择后，不同孩子子树之间没有边，因此它们的决策互不影响，总最优值等于各子树最优值之和。两种状态覆盖了 $u$ 的全部可能；每个孩子只在约束允许的状态中取最优，不会产生非法解。叶子初始化正确，按后序归纳即可证明所有节点状态正确。

## 手算例子

中心点权值为 5，三个叶子权值均为 2。选择中心时三个叶子都不能选，值为 5；不选中心时三个叶子相互独立，值为 $2+2+2=6$，所以答案为 6。这个例子也反驳了“总选局部权值最大的点”的错误贪心。

## 真实题目：P1352 没有上司的舞会

[洛谷 P1352 没有上司的舞会](https://www.luogu.com.cn/problem/P1352) 中员工与直接上司不能同时参加。员工是点，上下级关系是树边，快乐值是点权，合法参会集合正是树的独立集。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<long long> happiness(n + 1);
    for (int employee = 1; employee <= n; ++employee) {
        cin >> happiness[employee];
    }

    vector<vector<int>> children(n + 1);
    vector<bool> hasParent(n + 1, false);
    for (int edge = 1; edge < n; ++edge) {
        int employee, boss;
        cin >> employee >> boss;
        children[boss].push_back(employee);
        hasParent[employee] = true;
    }

    int root = 1;
    while (hasParent[root]) ++root;

    vector<array<long long, 2>> dp(n + 1);
    function<void(int)> solveSubtree = [&](int u) {
        dp[u][0] = 0;
        dp[u][1] = happiness[u];
        for (int child : children[u]) {
            solveSubtree(child);
            dp[u][0] += max(dp[child][0], dp[child][1]);
            dp[u][1] += dp[child][0];
        }
    };

    solveSubtree(root);
    cout << max(dp[root][0], dp[root][1]) << '\n';
    return 0;
}
```

每个节点和每条上下级关系只处理一次，时间 $O(n)$；邻接表、递归栈和 DP 数组共占 $O(n)$。若树退化成长链，递归深度为 $O(n)$；栈限制较严时应改用显式栈生成后序顺序。

## 向后学习

- 状态再增加“选了几个点”，合并孩子时会形成树上背包。
- 若要求以每个点为根的答案，可先算子树贡献，再把父侧贡献传给孩子，这称为换根 DP。

二者仍以“切断父子边后各部分独立”为基础，但各自需要独立页面和完整推导。

## 易错点

- 无向邻接表 DFS 时没有跳过父亲，造成无限递归。
- 选择父节点后仍取孩子两种状态的最大值。
- 只输出 `dp[root][1]`，忘记根也可以不选。
- 合并孩子时原地更新带计数维度的数组，导致同一孩子重复贡献。
