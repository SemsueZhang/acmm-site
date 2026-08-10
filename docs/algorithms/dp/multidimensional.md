# 多维动态规划：以最长公共子序列为例

一维状态不足以描述未来时，需要增加维度。维度不是越多越好：每一维都必须对应一项会影响后续选择的信息。本页用最长公共子序列（Longest Common Subsequence，LCS）说明如何从“两段前缀”自然得到二维状态。

## 问题与失败尝试

给定两个字符串 $A$、$B$，从每个字符串中删除若干字符但保持剩余字符相对顺序，求两者能够得到的最长相同序列。

只记录“处理到 $A$ 的第 $i$ 个字符”不够，因为能否匹配还取决于 $B$ 已经使用到哪里；贪心匹配最早出现的相同字符也可能堵死后续更长的方案。因此用两个前缀端点共同描述子问题。

## 状态与转移

设 $n=|A|,m=|B|$，定义

$$
dp[i][j]=A[0\ldots i-1]\text{ 与 }B[0\ldots j-1]\text{ 的 LCS 长度}.
$$

空前缀与任何字符串的 LCS 长度为 0，所以 `dp[0][j]=dp[i][0]=0`。

- 若 `A[i-1]==B[j-1]`，可以把这个公共字符接到两个更短前缀的答案后，得到 `dp[i-1][j-1]+1`。
- 若二者不同，公共子序列不可能同时使用两个末尾字符，至少舍弃一个，得到 `max(dp[i-1][j],dp[i][j-1])`。

```cpp
vector<vector<int>> lcsTable(const string& first, const string& second) {
    int n = static_cast<int>(first.size());
    int m = static_cast<int>(second.size());
    vector<vector<int>> dp(n + 1, vector<int>(m + 1));
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (first[i - 1] == second[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp;
}
```

## 正确性

若两个末尾字符不同，任意公共子序列至少不使用其中一个，所以它一定属于两个更短前缀子问题之一；这两个子问题的方案反过来仍是当前前缀的合法方案。

若末尾字符相同，取 `dp[i-1][j-1]` 的最优序列并接上该字符，得到长度加一的合法方案。另一方面，任意当前公共子序列删除最后一个字符后，长度至多为 `dp[i-1][j-1]`；因此相等分支不会低估或高估答案。按 $i+j$ 归纳，所有状态正确。

## 手算：`abcde` 与 `ace`

匹配 `a` 后答案成为 1；`b`、`d` 在另一个串的相应前缀中无法产生新匹配，只继承上方或左侧的最优值；`c`、`e` 分别令左上状态加一，最终 `dp[5][3]=3`，一条最优子序列是 `ace`。

状态数为 $O(nm)$，每个状态 $O(1)$ 转移，时间与空间均为 $O(nm)$。只求长度时可将空间滚动为 $O(m)$；需要恢复具体序列时保留整表或另存决策。

## 真实题目：P1439 最长公共子序列

[洛谷 P1439【模板】最长公共子序列](https://www.luogu.com.cn/problem/P1439) 的两个序列都是 $1\ldots n$ 的排列，数据规模不允许 $O(n^2)$。把第一个排列的值映射为位置，再把第二个排列替换为这些位置，公共子序列就与位置序列的严格上升子序列一一对应。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> position(n + 1);
    for (int index = 1; index <= n; ++index) {
        int value;
        cin >> value;
        position[value] = index;
    }

    vector<int> minimumTail;
    for (int index = 1; index <= n; ++index) {
        int value;
        cin >> value;
        int mappedPosition = position[value];
        auto place = lower_bound(minimumTail.begin(), minimumTail.end(),
                                 mappedPosition);
        if (place == minimumTail.end()) minimumTail.push_back(mappedPosition);
        else *place = mappedPosition;
    }
    cout << minimumTail.size() << '\n';
    return 0;
}
```

映射保持了第二个排列中的取值顺序；在第一个排列中保持相对顺序恰好等价于映射位置严格递增。程序时间 $O(n\log n)$、空间 $O(n)$。这个优化依赖两个序列都是无重复排列，一般字符串不能这样降为 LIS。

## 易错点

- 把子序列误写成必须连续的子串。
- 状态前缀长度与字符下标混用，忘记访问 `i-1`。
- 一般 LCS 无条件套用排列到 LIS 的优化。
- 滚动数组更新时覆盖仍需读取的左上角状态。
