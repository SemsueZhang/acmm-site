# 二分与倍增

二分查找的本质不是“数组有序”，而是答案空间中存在一道分界线：判定函数在分界线一侧为假，另一侧为真。

## 查找第一个不小于 $x$ 的位置

维护半开区间 `[l,r)`，答案始终在其中。若 `a[mid] >= x`，`mid` 可能是答案，保留它并令 `r=mid`；否则令 `l=mid+1`。

```cpp
int lowerBound(const vector<int>& a, int x) {
    int l = 0, r = (int)a.size();
    while (l < r) {
        int mid = l + (r - l) / 2;
        if (a[mid] >= x) r = mid;
        else l = mid + 1;
    }
    return l;
}
```

循环结束时 `l==r`，可能等于 `a.size()`，表示所有元素都小于 $x$。标准库 `lower_bound` 与此含义相同；`upper_bound` 返回第一个大于 $x$ 的位置。

## 二分答案例题：最小最大段和

把非负数组分成不超过 $m$ 个连续段，使“最大段和”尽量小。

设 `check(limit)` 表示能否让每段和都不超过 `limit`。`limit` 越大越容易满足，因此具有单调性。给定 `limit` 时，从左到右贪心装段：只要加入下一个数不超限就继续，否则新开一段。这样使用的段数最少。

答案下界是最大单个元素，上界是总和：

```cpp
bool check(const vector<int>& a, int m, long long limit) {
    int parts = 1;
    long long sum = 0;
    for (int x : a) {
        if (sum + x <= limit) sum += x;
        else { ++parts; sum = x; }
    }
    return parts <= m;
}

long long l = *max_element(a.begin(), a.end());
long long r = accumulate(a.begin(), a.end(), 0LL);
while (l < r) {
    long long mid = l + (r - l) / 2;
    if (check(a, m, mid)) r = mid;
    else l = mid + 1;
}
cout << l << '\n';
```

若 `check` 为 $O(n)$，总复杂度 $O(n\log S)$，$S$ 是答案值域。

## 实数二分

实数没有“相邻整数”，通常固定迭代 80～100 次，比用 `while (r-l>eps)` 更不易受精度问题影响。若每次区间减半，100 次后误差约缩小到 $2^{-100}$。

## 倍增

若操作可以组合，预处理 `jump[k][x]` 表示从状态 `x` 连续执行 $2^k$ 次后的状态。查询步数 $t$ 时枚举其二进制位：

```cpp
for (int k = 0; k < LOG; ++k)
    if ((t >> k) & 1LL) x = jump[k][x];
```

### 例题：函数迭代

给定每个点唯一的后继 `nxt[x]`，回答从 $x$ 出发走 $t$ 步到哪里。预处理：

```cpp
for (int x = 1; x <= n; ++x) jump[0][x] = nxt[x];
for (int k = 1; k < LOG; ++k)
    for (int x = 1; x <= n; ++x)
        jump[k][x] = jump[k - 1][jump[k - 1][x]];
```

预处理 $O(n\log T)$，单次查询 $O(\log T)$。LCA 的倍增表完全同源。

## 真题：P2678 跳石头

[洛谷 P2678 跳石头](https://www.luogu.com.cn/problem/P2678) 允许移走至多 $m$ 块石头，要求最大化最短跳跃距离。固定候选距离 `limit` 后，从起点向右扫描：若当前石头离上一个保留石头不足 `limit`，为了满足距离只能移走当前石头。

这个贪心使用的移除数最少，因此可以作为单调判定：`limit` 越大，越难满足。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int riverLength, stoneCount, removable;
    cin >> riverLength >> stoneCount >> removable;
    vector<int> position(stoneCount + 2);
    for (int i = 1; i <= stoneCount; ++i) cin >> position[i];
    position[stoneCount + 1] = riverLength;

    auto feasible = [&](int limit) {
        int removed = 0, lastKept = 0;
        for (int i = 1; i <= stoneCount + 1; ++i) {
            if (position[i] - position[lastKept] < limit) ++removed;
            else lastKept = i;
        }
        return removed <= removable;
    };

    int left = 0, right = riverLength + 1; // [left 可行, right 不可行)
    while (left + 1 < right) {
        int middle = left + (right - left) / 2;
        if (feasible(middle)) left = middle;
        else right = middle;
    }
    cout << left << '\n';
    return 0;
}
```

一次判定 $O(n)$，二分 $O(\log L)$ 次，总时间 $O(n\log L)$、空间 $O(n)$。

## 易错点

- 没有先证明 `check` 单调就二分。
- 区间定义混乱，`l<=r` 与 `l<r` 模板混用造成死循环。
- `mid=(l+r)/2` 溢出；使用 `l+(r-l)/2`。
- 下界没覆盖所有不可能答案，上界本身却不可行。
- 倍增表层数不足；若最大步数为 $T$，应满足 $2^{LOG}>T$。
