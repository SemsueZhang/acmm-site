# 哈希表

哈希表用函数 $h(key)$ 把大范围的键映射到有限桶中。理想情况下插入、删除、查找平均 $O(1)$；不同键可能映射到同一桶，这叫哈希冲突。

## 冲突处理

- **链地址法**：每个桶保存一组键。`unordered_map`/`unordered_set` 的思想接近这一类。
- **开放寻址法**：冲突后按规则寻找下一个空槽；删除时需设置墓碑，不能直接变回“从未使用”。

负数取模在 C++ 中可能为负，映射到数组桶前要规范化：`(x % mod + mod) % mod`。

## 例题：最长连续序列

给出无序整数数组，求由连续整数构成的最长长度。例如 `100,4,200,1,3,2` 的答案是 4（`1,2,3,4`）。

把所有数放进哈希集合。只有当 `x-1` 不存在时，`x` 才是某段连续序列的起点，再向右扩展。每个值至多属于一次有效扩展，期望时间 $O(n)$。

```cpp
unordered_set<int> s(a.begin(), a.end());
int ans = 0;
for (int x : s) {
    if (!s.count(x - 1)) {
        int y = x;
        while (s.count(y)) ++y;
        ans = max(ans, y - x);
    }
}
```

若 `x==INT_MIN`，计算 `x-1` 会溢出；稳妥做法是使用 `long long` 键或单独处理边界。

## 数值哈希与安全性

竞赛中 `unordered_map` 可能被特制数据制造大量冲突，退化到 $O(n^2)$。普通题通常可用；对抗性强或时限紧时，可用 `map` 获得确定的 $O(\log n)$，或采用带随机种子的自定义哈希。

字符串滚动哈希属于“给字符串构造数值指纹”，详见[字符串哈希](../strings/hash.md)。哈希相等不等价于字符串一定相等，严格判定可用双哈希或再比较原串。

## 真题：P4305 不重复数字

[洛谷 P4305 不重复数字](https://www.luogu.com.cn/problem/P4305) 要按第一次出现的顺序输出不同数字。排序虽然能去重，却会破坏原顺序；哈希集合只负责回答“以前是否出现”，输出仍按输入顺序进行。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int testCases;
    cin >> testCases;
    while (testCases--) {
        int n;
        cin >> n;
        unordered_set<int> seen;
        seen.reserve(n * 2);
        for (int i = 0; i < n; ++i) {
            int value;
            cin >> value;
            if (seen.insert(value).second) cout << value << ' ';
        }
        cout << '\n';
    }
    return 0;
}
```

平均时间 $O(n)$、空间 $O(n)$；若需要确定的最坏复杂度，可改用 `set`，时间变为 $O(n\log n)$。

## 易错点

- 用 `mp[key]` 只想查询，却无意中插入默认值；只查询时使用 `find`/`count`。
- 遍历 `unordered_map` 时依赖元素顺序；其遍历顺序无保证。
- 忽略最坏复杂度和哈希碰撞。
- 自定义结构体作为键时，只写哈希函数却没定义相等关系。
