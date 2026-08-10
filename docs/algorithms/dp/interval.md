# 区间动态规划

区间 DP 的状态对应连续区间，常见于“不断合并相邻区间”或“从区间两端取元素”。计算顺序通常按区间长度从短到长。

本页只讨论能够被某个“最后分界”拆成两个独立连续子问题的模型。区间使用 1 下标闭区间；若左右部分在合并前仍互相影响，只有 `dp[l][r]` 两维通常不够，不能强行套用。

## 例题：石子合并

一排 $n$ 堆石子，每次只能合并相邻两堆，代价是新堆石子数，求合并成一堆的最小总代价。

定义 `dp[l][r]` 为把区间 $[l,r]$ 合成一堆的最小代价。最后一次操作一定把某个分界 $k$ 两侧已合成的两堆合并：

$$
dp[l][r]=\min_{l\le k<r}\{dp[l][k]+dp[k+1][r]+sum(l,r)\}.
$$

单堆无需合并，`dp[i][i]=0`。区间和由前缀和 $O(1)$ 得到。

为什么枚举最后一次合并不会漏？任意完整方案都有唯一的最后一次操作，它把最终区间分成 `[l,k]` 与 `[k+1,r]` 两堆；在最后一步之前，两侧内部必须各自已经合成一堆。将两侧替换为各自最优方案只会使总代价更小，因此最优方案一定出现在枚举式中。反过来，任意分界的两个子方案加最后一次合并都构成合法方案。

```cpp
const long long INF = (1LL << 60);
vector<vector<long long>> dp(n + 1, vector<long long>(n + 1));
for (int len = 2; len <= n; ++len) {
    for (int l = 1; l + len - 1 <= n; ++l) {
        int r = l + len - 1;
        dp[l][r] = INF;
        for (int k = l; k < r; ++k)
            dp[l][r] = min(dp[l][r],
                dp[l][k] + dp[k + 1][r] + pre[r] - pre[l - 1]);
    }
}
cout << dp[1][n] << '\n';
```

### 手算

石子 `1,2,3`：先合并前两堆，代价 3，再与 3 合并代价 6，总计 9；先合并后两堆代价 5，再与 1 合并代价 6，总计 11，因此答案 9。

状态数 $O(n^2)$，每个状态枚举 $O(n)$ 个分界，时间 $O(n^3)$，空间 $O(n^2)$。

## 回文区间模型

判断子串是否回文可定义 `pal[l][r] = (s[l]==s[r] && pal[l+1][r-1])`。短区间必须先算，因此同样按长度递增。长度 1 为真，长度 2 需单独处理或让空区间为真。

## 环形区间

首尾相邻时常把数组复制一遍，长度变成 $2n$，只计算长度不超过 $n$ 的区间，最后对所有起点的长度 $n$ 区间取答案。

## 真题：P1880 石子合并

[洛谷 P1880 石子合并](https://www.luogu.com.cn/problem/P1880) 是环形版本，并同时要求最小与最大代价。把数组复制一遍后，对长度不超过 $n$ 的区间做相同转移，最后枚举断环位置。

识别线索是“只能合并相邻堆”保证中间过程始终对应连续区间；环形结构则通过复制把每个断点变成线性区间。数据范围允许 $O(n^3)$，因此无需引入不在提高级主线中的高级区间 DP 优化。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> stones(2 * n + 1), prefix(2 * n + 1);
    for (int i = 1; i <= n; ++i) {
        cin >> stones[i];
        stones[i + n] = stones[i];
    }
    for (int i = 1; i <= 2 * n; ++i)
        prefix[i] = prefix[i - 1] + stones[i];

    const int INF = 0x3f3f3f3f;
    vector<vector<int>> minimum(2 * n + 1, vector<int>(2 * n + 1));
    vector<vector<int>> maximum(2 * n + 1, vector<int>(2 * n + 1));

    for (int length = 2; length <= n; ++length) {
        for (int left = 1; left + length - 1 <= 2 * n; ++left) {
            int right = left + length - 1;
            minimum[left][right] = INF;
            int intervalSum = prefix[right] - prefix[left - 1];
            for (int split = left; split < right; ++split) {
                minimum[left][right] = min(minimum[left][right],
                    minimum[left][split] + minimum[split + 1][right]
                    + intervalSum);
                maximum[left][right] = max(maximum[left][right],
                    maximum[left][split] + maximum[split + 1][right]
                    + intervalSum);
            }
        }
    }

    int answerMinimum = INF, answerMaximum = 0;
    for (int left = 1; left <= n; ++left) {
        int right = left + n - 1;
        answerMinimum = min(answerMinimum, minimum[left][right]);
        answerMaximum = max(answerMaximum, maximum[left][right]);
    }
    cout << answerMinimum << '\n' << answerMaximum << '\n';
    return 0;
}
```

状态 $O(n^2)$、每个状态枚举分界 $O(n)$，总时间 $O(n^3)$、空间 $O(n^2)$。

## 易错点

- 外层按左端点递增，导致子区间尚未计算。
- 分界枚举到 `k=r`，使右区间为空或越界。
- 忘记单点区间初始化。
- 合并代价重复加到子问题，或漏掉最后一次合并代价。
- 环形问题复制后答案仍只取 `[1,n]`，漏掉更优断点。
