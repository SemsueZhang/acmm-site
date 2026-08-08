# 动态规划的常用优化

优化 DP 前先写出正确的朴素转移，并找出瓶颈是状态数还是转移枚举。这里介绍提高组常见的滚动数组、前缀最值、单调队列和斜率优化的识别方法。

## 滚动数组

若第 $i$ 层只依赖第 $i-1$ 层，可只保留两层；若能原地更新，再压成一层。压缩后必须重新判断循环方向，0/1 背包就是典型例子。

## 前缀和/前缀最值优化

若转移为 `dp[i]=sum(dp[l..r])`，预处理 DP 的前缀和即可 $O(1)$ 求区间和；若为 `max(dp[0..i-k])`，维护前缀最大值。关键是可选前驱区间端点随 $i$ 单调变化。

## 单调队列优化

转移形如：

$$
dp_i=c_i+\max_{i-k\le j<i}f(j),
$$

候选 $j$ 构成滑动窗口，可用单调队列维护 $f(j)$ 最大值，把 $O(nk)$ 降到 $O(n)$。

### 例题：限长跳跃最大得分

到达位置 $i$ 前必须从最近 $k$ 个位置之一跳来，落点得分为 $a_i$：

$$dp_i=a_i+\max(dp_{i-k},\dots,dp_{i-1}).$$

```cpp
deque<int> q;
dp[0] = 0;
q.push_back(0);
for (int i = 1; i <= n; ++i) {
    while (!q.empty() && q.front() < i - k) q.pop_front();
    dp[i] = a[i] + dp[q.front()];
    while (!q.empty() && dp[q.back()] <= dp[i]) q.pop_back();
    q.push_back(i);
}
```

队列保存下标以判断过期，并按 `dp` 值递减。

## 斜率优化的识别

若朴素转移可整理为：

$$
dp_i = A_i + \min_{j<i}(Y_j - K_iX_j),
$$

对固定 $i$，每个 $j$ 对应直线（或点），查询参数 $K_i$ 下最小值。若加入的 $X_j$ 与查询斜率 $K_i$ 都单调，可用凸包 + 单调指针/队列；不单调时可能需要二分或动态结构。

斜率比较应交叉相乘，避免浮点误差；乘积可能需要 `__int128` 防溢出。

!!! warning "不要套公式"
    斜率优化的前提包括转移式能线性化、候选集合正确、单调性成立。页面只给出识别框架；实题必须从原式逐项推导截距、横坐标和查询斜率。

## 真题：P1725 琪露诺

[洛谷 P1725 琪露诺](https://www.luogu.com.cn/problem/P1725) 中，到达位置 $i$ 的上一步必须来自 $[i-R,i-L]$，转移为窗口最大值：

$$
dp_i=a_i+\max_{i-R\le j\le i-L}dp_j.
$$

随着 $i$ 增加，候选右端点 `i-L` 依次加入，左端点 `i-R` 依次过期，正适合单调队列。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, minimumJump, maximumJump;
    cin >> n >> minimumJump >> maximumJump;
    vector<long long> score(n + 1), dp(n + 1, LLONG_MIN / 4);
    for (int i = 0; i <= n; ++i) cin >> score[i];

    deque<int> candidates;
    dp[0] = 0;
    long long answer = LLONG_MIN;

    for (int position = 1; position <= n; ++position) {
        int entering = position - minimumJump;
        if (entering >= 0 && dp[entering] > LLONG_MIN / 8) {
            while (!candidates.empty() &&
                   dp[candidates.back()] <= dp[entering])
                candidates.pop_back();
            candidates.push_back(entering);
        }
        while (!candidates.empty() &&
               candidates.front() < position - maximumJump)
            candidates.pop_front();
        if (!candidates.empty())
            dp[position] = score[position] + dp[candidates.front()];
        if (position + maximumJump > n) answer = max(answer, dp[position]);
    }
    cout << answer << '\n';
    return 0;
}
```

每个位置进入、离开队列至多一次，时间 $O(n)$、空间 $O(n)$。实现时要按题意确认终点判定范围，以及不可达状态不能进入候选队列。

## 优化检查表

1. 朴素 DP 是否已证明正确？
2. 被重复计算的是区间和、区间最值，还是一组线性函数最值？
3. 候选加入和过期顺序是否单调？
4. 相等值保留哪一个候选？
5. 优化后的复杂度是否真的降低，空间是否可承受？

## 易错点

- 空间滚动后仍访问已覆盖的旧状态。
- 单调队列先查询后删除过期元素。
- 斜率用 `double` 比较，在大整数下精度出错。
- 交叉乘法用 `long long` 溢出。
- 只因转移有两个下标就强行使用斜率优化。
