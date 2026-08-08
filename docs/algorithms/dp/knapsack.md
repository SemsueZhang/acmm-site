# 背包动态规划

背包模型：物品有体积 $w_i$、价值 $v_i$，容量为 $W$，求可获得的最大价值。不同模型的核心区别是每件物品可使用几次。

## 0/1 背包：每件至多一次

二维状态 `dp[i][j]` 表示前 $i$ 件物品、容量不超过 $j$ 的最大价值：

$$
dp[i][j]=\max(dp[i-1][j],\ dp[i-1][j-w_i]+v_i).
$$

压成一维后，容量必须**从大到小**，确保转移使用的 `dp[j-w]` 仍属于处理本物品之前的旧层。

```cpp
vector<long long> dp(W + 1, 0);
for (auto [w, v] : items)
    for (int j = W; j >= w; --j)
        dp[j] = max(dp[j], dp[j - w] + v);
```

### 手算例题

容量 5，物品 `(2,3),(3,4),(4,5)`。前两件可同时选，价值 7；第三件单独价值 5，答案 7。处理第一件后 `dp[2..5]` 都至少为 3；倒序处理第二件时，`dp[5]` 从旧层的 `dp[2]+4` 得到 7，恰好各用一次。

## 完全背包：每件无限次

转移公式形式相似，但容量**从小到大**，允许本轮刚更新的 `dp[j-w]` 再使用同一物品。

```cpp
for (auto [w, v] : items)
    for (int j = w; j <= W; ++j)
        dp[j] = max(dp[j], dp[j - w] + v);
```

循环方向不是背诵规则：它决定转移读取旧层（0/1）还是当前层（完全）。

## 多重背包：每件有限次

物品最多 $c$ 件。朴素枚举使用数量是 $O(nWc)$。二进制拆分把 $c$ 拆成 $1,2,4,\dots$ 以及余数，例如 13 拆成 `1+2+4+6`，每组视为一件 0/1 物品，复杂度 $O(W\sum\log c_i)$。

## “恰好装满”的初始化

若题目要求总体积**恰好**为 $W$，不能把所有 `dp[j]` 初始化为 0：这会把不可达容量当成价值 0 的方案。

```cpp
const long long NEG = -(1LL << 60);
vector<long long> dp(W + 1, NEG);
dp[0] = 0;
```

转移前还应确保前驱不是 `NEG`。

## 方案数与方案恢复

把 `max` 改为加法可统计方案数，但需明确物品是否有区别、顺序是否算不同。恢复选择方案时，可保留二维 DP，从 `(n,W)` 逆推：若 `dp[i][j] != dp[i-1][j]`，说明一种最优方案选择了物品 $i$。

## 真题：P1048 采药

[洛谷 P1048 采药](https://www.luogu.com.cn/problem/P1048) 中，每株草药有采集时间和价值，每株最多采一次，总时间有限。这三个条件分别对应 0/1 背包的体积、价值和容量。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int totalTime, herbs;
    cin >> totalTime >> herbs;
    vector<int> dp(totalTime + 1);

    while (herbs--) {
        int timeCost, value;
        cin >> timeCost >> value;
        for (int time = totalTime; time >= timeCost; --time)
            dp[time] = max(dp[time], dp[time - timeCost] + value);
    }
    cout << dp[totalTime] << '\n';
    return 0;
}
```

容量倒序保证当前草药不会重复使用。时间 $O(MT)$、空间 $O(T)$，其中 $M$ 是草药数量、$T$ 是总时间。

## 易错点

- 0/1 背包容量正序，导致一件物品被重复选。
- 完全背包容量倒序，错误地限制每件一次。
- “容量不超过”和“恰好装满”初始化相同。
- 价值和溢出，或用 `NEG + value` 继续转移。
- 计数背包的物品循环、容量循环交换后，统计对象发生变化。
