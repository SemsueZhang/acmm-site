# DP 基础与线性 DP

动态规划（Dynamic Programming，DP）不是一条固定公式，而是把大量重复出现的子问题压缩为有限状态，并按依赖顺序计算。本页面向第一次系统学习 DP 的读者，只讨论线性阶段上的最优化与计数；背包、区间、树形和状态压缩分别在后续页面展开。

## 什么时候考虑 DP

问题具有以下特征时值得尝试：大问题可由小问题组成（最优子结构），且许多递归分支会遇到相同子问题（重叠子问题）。DP 不一定只求最值，也能计数、判断可行性或构造方案。

更可操作的识别方式是先问：能否规定一种生成答案的顺序，使得每走一步，只需保留少量关于过去的信息？若能，状态就是这些“会影响未来、但不必保存全部历史”的信息。若两个不同历史在未来所有合法选择和代价上完全等价，它们应合并为同一状态。

## 设计状态

状态必须包含“未来决策所需的全部历史信息”，但不保存无关细节。常用步骤：

1. 明确阶段，例如处理到第 $i$ 个元素。
2. 找出未来还关心的历史属性，作为其他维度。
3. 写出最后一步来自哪些前驱。
4. 定义不可达值并确定遍历顺序。

## 例题：最长上升子序列（LIS）

给定数组，求严格上升子序列最大长度。定义：

$$
dp_i=\text{以 }a_i\text{ 结尾的 LIS 长度}.
$$

最后一个元素固定为 $a_i$ 后，倒数第二个元素可以是任意 $j<i$ 且 $a_j<a_i$：

$$
dp_i=1+\max_{j<i,a_j<a_i}dp_j.
$$

若不存在合法 $j$，`dp[i]=1`。答案是所有 `dp[i]` 的最大值，而不一定是 `dp[n]`。

正确性分两部分。任何以 $a_i$ 结尾的上升子序列，删去最后一项后，要么为空，要么以某个 $j<i$ 且 $a_j<a_i$ 结尾，所以转移枚举没有漏掉最优解；反过来，任意合法 `dp[j]+1` 都能把 $a_i$ 接到对应子序列末尾，得到合法候选，因此转移不会产生虚假答案。

```cpp
vector<int> dp(n, 1);
int answer = 0;
for (int i = 0; i < n; ++i) {
    for (int j = 0; j < i; ++j)
        if (a[j] < a[i]) dp[i] = max(dp[i], dp[j] + 1);
    answer = max(answer, dp[i]);
}
```

例 `3 1 2 5 4` 的 `dp` 为 `1 1 2 3 3`，答案 3。时间 $O(n^2)$，空间 $O(n)$。

### $O(n\log n)$ 优化直觉

维护 `tail[len]`：长度为 `len+1` 的上升子序列可达到的最小末尾。末尾越小，越容易接后续元素。对每个 `x`，用 `lower_bound` 找第一个不小于它的位置替换。

```cpp
vector<int> tail;
for (int x : a) {
    auto it = lower_bound(tail.begin(), tail.end(), x);
    if (it == tail.end()) tail.push_back(x);
    else *it = x;
}
cout << tail.size() << '\n';
```

`tail` 并不一定是一条真实 LIS，只用于保存每种长度的最好末尾。非严格上升要改用 `upper_bound`。

## 计数 DP

走网格只能向右或向下，`ways[i][j]=ways[i-1][j]+ways[i][j-1]`。有障碍的格子置 0。计数通常很大，应按题意取模；先相加再取模时也要防溢出。

## 记忆化与递推

记忆化搜索只计算从答案可达的状态，写法接近题意；自底向上递推没有递归开销，顺序更清晰。二者状态与转移本质相同。

## 真题：P1020 导弹拦截

[洛谷 P1020 导弹拦截](https://www.luogu.com.cn/problem/P1020) 第一问是最长不上升子序列；第二问根据 Dilworth 定理，最少的不上升序列划分数等于最长严格上升子序列长度。

可以把第一问的每个高度取相反数，把“最长不上升”转成“最长不下降”，使用 `upper_bound`；第二问直接使用严格 LIS 的 `lower_bound`。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> heights;
    int height;
    while (cin >> height) heights.push_back(height);

    vector<int> nonDecreasingNegated;
    for (int value : heights) {
        int transformed = -value;
        auto position = upper_bound(nonDecreasingNegated.begin(),
                                    nonDecreasingNegated.end(), transformed);
        if (position == nonDecreasingNegated.end())
            nonDecreasingNegated.push_back(transformed);
        else
            *position = transformed;
    }

    vector<int> increasing;
    for (int value : heights) {
        auto position = lower_bound(increasing.begin(), increasing.end(), value);
        if (position == increasing.end()) increasing.push_back(value);
        else *position = value;
    }

    cout << nonDecreasingNegated.size() << '\n';
    cout << increasing.size() << '\n';
    return 0;
}
```

两次扫描各为 $O(n\log n)$，空间 $O(n)$。严格与非严格的差别最终落实在 `lower_bound` 和 `upper_bound`，必须从状态定义判断，不能凭记忆互换。

这道题的迁移价值在于：当“选出的序列保持原下标顺序，只限制值的单调性”时，先把题意精确翻译成严格／非严格的子序列，再选择对应的二分边界。数据规模排除了 $O(n^2)$ 枚举前驱，因此必须维护各长度的最优结尾值。

## 易错点

- 状态含义不完整，例如只记录位置却漏掉剩余资源。
- 最值 DP 把不可达状态初始化为 0，产生虚假路径。
- 转移顺序读到了本轮刚更新且本不该使用的状态。
- 只输出最后一个状态，实际答案应对所有终点取最值。
