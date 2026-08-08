# 单调栈与单调队列

单调结构主动删除“以后永远不可能成为答案”的元素，使每个元素最多进入和离开一次，因此总复杂度是 $O(n)$。

## 单调栈：寻找最近更大/更小元素

### 例题：右侧第一个更小元素

给定数组，对每个位置求右侧第一个严格更小元素的位置。维护一个值单调不减的下标栈；新元素 $a_i$ 比栈顶小时，说明它就是栈顶位置等待的第一个更小元素。

```cpp
vector<int> ans(n, -1), st;
for (int i = 0; i < n; ++i) {
    while (!st.empty() && a[i] < a[st.back()]) {
        ans[st.back()] = i;
        st.pop_back();
    }
    st.push_back(i);
}
```

数组 `4 2 3 1`：读到 2 时弹出 4；读到 1 时依次弹出 3 和 2。每个下标只压入和弹出一次，时间 $O(n)$。

相等元素是否弹出取决于题目要“严格更小”还是“不大于”。这是最常见的错误来源。

## 单调队列：滑动窗口最值

### 例题：每个长度为 $k$ 的窗口最大值

双端队列保存可能成为最大值的下标，并保证对应值从队首到队尾单调递减：

1. 队首若已离开窗口，弹出。
2. 队尾若不大于新值，永远不会再成为最大值，弹出。
3. 新下标入队，队首就是当前最大值。

```cpp
deque<int> q;
for (int i = 0; i < n; ++i) {
    while (!q.empty() && q.front() <= i - k) q.pop_front();
    while (!q.empty() && a[q.back()] <= a[i]) q.pop_back();
    q.push_back(i);
    if (i >= k - 1) cout << a[q.front()] << '\n';
}
```

必须存下标而不是只存值，否则无法判断元素是否离开窗口。总时间 $O(n)$，空间 $O(k)$。

## DP 优化中的单调队列

若转移形如

$$
dp_i = c_i + \max_{i-k\le j<i} dp_j,
$$

右侧是长度固定窗口中的最大值，可用同样方法把 $O(nk)$ 降为 $O(n)$。先删除过期下标，再取队首转移，最后把新的 `dp[i]` 加入队列。

## 真题：P5788 单调栈

[洛谷 P5788【模板】单调栈](https://www.luogu.com.cn/problem/P5788) 要求每个位置右侧第一个严格更大的元素下标。扫描到 `a[i]` 时，栈中所有比它小的元素终于找到了答案，因此依次弹出并令答案为 `i`；没有被弹出的下标最终答案为 0。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> a(n + 1), answer(n + 1), stackIndex;
    for (int i = 1; i <= n; ++i) cin >> a[i];

    for (int i = 1; i <= n; ++i) {
        while (!stackIndex.empty() && a[stackIndex.back()] < a[i]) {
            answer[stackIndex.back()] = i;
            stackIndex.pop_back();
        }
        stackIndex.push_back(i);
    }

    for (int i = 1; i <= n; ++i)
        cout << answer[i] << " \n"[i == n];
    return 0;
}
```

栈中下标对应的值单调不增。每个下标入栈、出栈至多一次，总时间 $O(n)$、空间 $O(n)$。

## 易错点

- 单调方向与目标最值相反。
- 过期条件写成 `< i-k` 而非 `<= i-k`。
- 处理重复值时随意使用 `<` 或 `<=`，造成边界答案变化。
- 误认为 `while` 嵌套导致 $O(n^2)$；均摊分析看的是每个元素总共被弹出几次。
