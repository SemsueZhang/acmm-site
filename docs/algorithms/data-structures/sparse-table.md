# ST 表（Sparse Table）

ST 表用于**静态数组**上的可重复贡献区间查询，最典型的是最大值、最小值和 gcd。预处理后查询 $O(1)$，但不支持修改。

更准确地说，两个重叠块的做法要求合并运算满足幂等性 $f(x,x)=x$，同时还要能结合。最大值、最小值与最大公约数（Greatest Common Divisor，GCD）满足；加法不满足。本页使用 0 下标闭区间推导最大值版本。

## 倍增预处理

定义 `st[k][i]` 为从 $i$ 开始、长度为 $2^k$ 的区间最大值：

$$
st_{k,i}=\max(st_{k-1,i},\ st_{k-1,i+2^{k-1}}).
$$

长度 $2^k$ 的区间由两个相邻的 $2^{k-1}$ 区间组成。

## 查询为什么可以重叠

查询 $[l,r]$，令 $k=\lfloor\log_2(r-l+1)\rfloor$。取左端长度 $2^k$ 的块和右端长度 $2^k$ 的块，它们能覆盖整个查询区间，中间可能重叠。最大值重复计算不影响答案：$\max(x,x)=x$。

这也解释了为什么同一模板不能直接求区间和：重叠元素会被加两次。

设查询长度为 $L$，选取的块长 $2^k\le L<2^{k+1}$。因为 $L<2\cdot2^k$，左块的右端至少到达右块的左端前一位，两个块的并集覆盖整个区间；又因为运算幂等，重叠部分出现两次不改变合并结果。这同时证明了覆盖完整性和重复贡献的安全性。

```cpp
struct SparseTable {
    vector<int> lg;
    vector<vector<int>> st;

    SparseTable(const vector<int>& a) {
        int n = a.size();
        lg.resize(n + 1);
        for (int i = 2; i <= n; ++i) lg[i] = lg[i / 2] + 1;
        int K = lg[n] + 1;
        st.assign(K, vector<int>(n));
        st[0] = a;
        for (int k = 1; k < K; ++k)
            for (int i = 0; i + (1 << k) <= n; ++i)
                st[k][i] = max(st[k - 1][i],
                               st[k - 1][i + (1 << (k - 1))]);
    }

    int query(int l, int r) const { // 0-based 闭区间
        int k = lg[r - l + 1];
        return max(st[k][l], st[k][r - (1 << k) + 1]);
    }
};
```

### 例题

数组 `1 5 2 4 3 7`，查询 `[1,4]`（值为 `5 2 4 3`），长度 4，直接取 `st[2][1]=5`。查询 `[1,5]` 长度 5，$k=2$，比较 `[1,4]` 与 `[2,5]` 两个长度 4 的块，得到 7。

预处理时间和空间都是 $O(n\log n)$，查询 $O(1)$。

## 真题：P3865 ST 表

[洛谷 P3865【模板】ST 表](https://www.luogu.com.cn/problem/P3865) 的数组不会修改，需要多次查询闭区间最大值，正好满足“静态、幂等运算”两个识别条件。

若有在线修改，预处理表会立刻失效；若查询区间和，重叠块会重复计数。数据结构选择必须同时核对“是否静态”和“合并是否幂等”，不能只看到很多区间询问就使用 ST 表。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, queries;
    cin >> n >> queries;
    vector<int> logarithm(n + 1);
    for (int i = 2; i <= n; ++i) logarithm[i] = logarithm[i / 2] + 1;

    int levels = logarithm[n] + 1;
    vector<vector<int>> st(levels, vector<int>(n + 1));
    for (int i = 1; i <= n; ++i) cin >> st[0][i];
    for (int level = 1; level < levels; ++level)
        for (int left = 1; left + (1 << level) - 1 <= n; ++left)
            st[level][left] = max(st[level - 1][left],
                st[level - 1][left + (1 << (level - 1))]);

    while (queries--) {
        int left, right;
        cin >> left >> right;
        int level = logarithm[right - left + 1];
        cout << max(st[level][left],
                    st[level][right - (1 << level) + 1]) << '\n';
    }
    return 0;
}
```

预处理 $O(n\log n)$，每次查询 $O(1)$，空间 $O(n\log n)$。

## 易错点

- 建表时没有保证 `i + (1<<k) <= n`，导致越界。
- 查询空区间或 `l>r`。
- 对区间和等不可重叠贡献操作套用两个重叠块。
- 数组有修改仍继续使用旧 ST 表。
