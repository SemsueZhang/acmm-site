# 状态压缩动态规划

当对象数 $n$ 很小（常见 $n\le20$）且只需记录“每个对象选或未选”，可用整数二进制位表示集合。第 $i$ 位为 1 表示对象 $i$ 已在集合中。

## 位操作

```cpp
bool has = (mask >> i) & 1;
int added = mask | (1 << i);
int removed = mask & ~(1 << i);
int count = __builtin_popcount((unsigned)mask);
```

若 $n>31$，使用 `1LL<<i` 和 `long long`；移位宽度不能达到或超过类型位数。

## 例题：旅行商问题（TSP）

从 0 号点出发，访问每个点恰好一次，求到达各终点的最小代价（先不要求回起点）。定义：

$$
dp[mask][u]=\text{访问集合 mask，且当前在 }u\text{ 的最小代价}.
$$

初始化 `dp[1<<0][0]=0`。从集合外选择下一点 $v$：

```cpp
const long long INF = (1LL << 60);
int total = 1 << n;
vector<vector<long long>> dp(total, vector<long long>(n, INF));
dp[1][0] = 0;
for (int mask = 0; mask < total; ++mask) {
    for (int u = 0; u < n; ++u) if ((mask >> u) & 1) {
        if (dp[mask][u] == INF) continue;
        for (int v = 0; v < n; ++v) if (!((mask >> v) & 1)) {
            int next = mask | (1 << v);
            dp[next][v] = min(dp[next][v], dp[mask][u] + cost[u][v]);
        }
    }
}
```

若需回到 0，答案为 `min(dp[all][u]+cost[u][0])`。状态数 $2^n n$，转移 $O(n)$，总时间 $O(2^n n^2)$，空间 $O(2^n n)$。

## 枚举子集

枚举 `mask` 的所有非空子集：

```cpp
for (int sub = mask; sub; sub = (sub - 1) & mask) {
    // sub 是 mask 的一个非空子集
}
```

所有 `mask` 的所有子集总数为 $3^n$，因为每个元素有“不在 mask、在 mask 但不在 sub、在 sub”三种状态。不能误判为 $O(2^n)$。

## 合法状态预处理

棋盘 DP 中可先枚举一行的所有掩码，筛掉相邻位同时为 1 的状态：`(mask & (mask<<1))==0`。再预处理任意两行是否冲突，避免 DP 内反复位运算。

## 易错点

- `1<<n` 用 `int` 溢出。
- 忘记要求当前位置 `u` 必须属于 `mask`。
- 从不可达 `INF` 状态继续加代价导致溢出。
- 集合含义不统一：有时表示已选，有时表示未选。
- 忽视指数复杂度；$2^{25}$ 再乘维度通常已经很大。
