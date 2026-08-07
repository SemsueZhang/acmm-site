# 字符串哈希

字符串哈希的目标，是给一个字符串算出较短的“指纹”。比较两段很长的字符串时，我们先比较指纹，就不必逐字符扫描。

它非常灵活，但有一个必须提前说明的限制：**不同字符串可能得到相同哈希值**，这叫作哈希碰撞。因此，哈希相等只能说明“两串极大概率相等”；需要数学意义上完全正确时，应使用 KMP、Z 函数、后缀数组等确定性算法，或在哈希相等后再比较原串。

## 从重复比较说起

设字符串为 `abacaba`，要多次判断两个等长子串是否相等。

- 直接比较长度为 $L$ 的子串：一次 $O(L)$；
- 若能像前缀和那样预处理，理想情况下可以一次 $O(1)$。

数值前缀和可以用“右前缀减左前缀”取出区间。字符串不能直接相减，但可以先把它看成一个 $B$ 进制数。

## 多项式滚动哈希

把每个字符映射为正整数，选择底数 $B$ 和模数 $M$。对 1-based 字符串，定义：

$$
H_i=(H_{i-1}\times B+value(s_i))\bmod M,
$$

其中 $H_0=0$。例如 `abc` 的哈希形式是：

$$
value(a)B^2+value(b)B+value(c).
$$

同时预处理 $P_i=B^i\bmod M$。子串 $s[l..r]$ 的哈希为：

$$
hash(l,r)=H_r-H_{l-1}\times P_{r-l+1}\pmod M.
$$

为什么要乘 $P_{r-l+1}$？因为 $H_r$ 中，前缀 $s[1..l-1]$ 后面还接了 $r-l+1$ 个字符；把 $H_{l-1}$ 左移相同位数，才能准确消去它。

以 `abac` 的 `[2,4] = "bac"` 为例：

$$
\begin{aligned}
H_4 &= aB^3+bB^2+aB+c,\\
H_1B^3 &= aB^3,\\
H_4-H_1B^3 &= bB^2+aB+c.
\end{aligned}
$$

## 双哈希模板

单模数存在被构造碰撞的风险。竞赛中常同时使用两个不同的大质数；只有两项都相等时，才认为子串相等。

```cpp
struct StringHash {
    static constexpr long long BASE = 911382323;
    static constexpr long long MOD1 = 1000000007;
    static constexpr long long MOD2 = 1000000009;

    vector<long long> h1, h2, p1, p2;

    explicit StringHash(const string& s) {
        int n = s.size();
        h1.assign(n + 1, 0);
        h2.assign(n + 1, 0);
        p1.assign(n + 1, 1);
        p2.assign(n + 1, 1);

        for (int i = 1; i <= n; ++i) {
            int value = (unsigned char)s[i - 1] + 1;
            h1[i] = (h1[i - 1] * BASE + value) % MOD1;
            h2[i] = (h2[i - 1] * BASE + value) % MOD2;
            p1[i] = p1[i - 1] * BASE % MOD1;
            p2[i] = p2[i - 1] * BASE % MOD2;
        }
    }

    // 返回 1-based 闭区间 s[l..r] 的双哈希
    pair<long long, long long> get(int l, int r) const {
        int len = r - l + 1;
        long long x1 = (h1[r] - h1[l - 1] * p1[len]) % MOD1;
        long long x2 = (h2[r] - h2[l - 1] * p2[len]) % MOD2;
        if (x1 < 0) x1 += MOD1;
        if (x2 < 0) x2 += MOD2;
        return {x1, x2};
    }
};
```

预处理时间、空间都是 $O(n)$，之后每次子串哈希查询为 $O(1)$。

!!! note "参数并非随便选择"
    底数不能是 $0$ 或 $1$，并且应小于模数。上面这组参数适合一般练习，但双哈希仍然不是“绝对无碰撞”。面对允许卡哈希的数据，可以随机选择合法底数，或改用确定性算法。

## 例题：P3370【模板】字符串哈希

[洛谷 P3370【模板】字符串哈希](https://www.luogu.com.cn/problem/P3370)：给出 $n$ 个字符串，求不同字符串的数量。

### 思路

对每个完整字符串计算双哈希，把二元组存入 `set`，最终集合大小就是不同哈希的数量。设所有字符串总长度为 $S$：

- 计算哈希共 $O(S)$；
- `set` 插入 $n$ 次，共 $O(n\log n)$；
- 额外空间 $O(n)$。

这道题也能直接把原字符串放进 `set<string>`，而且是确定性做法。这里使用双哈希，是为了练习“把长对象压缩为可快速比较的指纹”。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    const long long BASE = 911382323;
    const long long MOD1 = 1000000007;
    const long long MOD2 = 1000000009;

    int n;
    cin >> n;
    set<pair<long long, long long>> hashes;

    while (n--) {
        string s;
        cin >> s;
        long long h1 = 0, h2 = 0;
        for (unsigned char ch : s) {
            int value = ch + 1;
            h1 = (h1 * BASE + value) % MOD1;
            h2 = (h2 * BASE + value) % MOD2;
        }
        hashes.insert({h1, h2});
    }

    cout << hashes.size() << '\n';
    return 0;
}
```

## 应用一：二分最长公共前缀

给出两个后缀起点 $i,j$，求它们的最长公共前缀 LCP。

“长度为 $k$ 的前缀相等”具有单调性：若前 $k$ 个字符相等，那么前 $0,1,\ldots,k-1$ 个字符也相等。于是可以二分答案 $k$，每次用哈希 $O(1)$ 比较：

```cpp
int lcp(const StringHash& hs, int i, int j, int n) { // i、j 为 1-based
    int low = 0, high = n - max(i, j) + 1;
    while (low < high) {
        int mid = (low + high + 1) / 2;
        if (hs.get(i, i + mid - 1) == hs.get(j, j + mid - 1))
            low = mid;
        else
            high = mid - 1;
    }
    return low;
}
```

例如 `banana` 中，后缀 `anana` 和 `ana` 的前 3 个字符相等，而长度 4 不可行，所以 LCP 为 3。单次查询复杂度为 $O(\log n)$。

## 应用二：回文判定

给原串 $s$ 和反转串 $rev$ 分别建哈希。原串的 $s[l..r]$ 在反转串中对应：

$$
rev[n-r+1..n-l+1].
$$

两个哈希相等，则该区间极大概率是回文。这样可以 $O(1)$ 回答一次回文判定；若要求求出所有中心的回文半径，Manacher 更快且完全确定。

## 易错点

- 字符映射使用 0，使前导字符的信息被弱化；映射成正整数更稳妥。
- 子串长度写成 `r-l`，实际应为 `r-l+1`。
- C++ 负数取模仍可能是负数，相减后要加模数修正。
- 直接比较不同位权的前缀哈希；应使用统一的 `get(l,r)`。
- 只换底数却仍使用同一个模数，并把两项误当作完全独立。
- 把哈希相等写成“两个字符串必然相等”。

## 小结

字符串哈希的核心不是背模板，而是理解两件事：

1. 前缀哈希记录的是一个多项式；
2. 乘幂对齐后，才能像前缀和一样消去左侧前缀。

它适合大量子串比较、二分 LCP、回文判定和字符串去重；若题目不能接受任何碰撞风险，应选择对应的确定性算法。
