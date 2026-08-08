# 组合数学

计数题先明确“对象是否可区分、顺序是否重要、是否允许重复”，再选择加法原理、乘法原理或容斥。

## 排列与组合

- 从 $n$ 个不同元素选 $k$ 个并排列：$A_n^k=n!/(n-k)!$。
- 只选集合、不计顺序：$\binom nk=n!/(k!(n-k)!)$。

Pascal 递推不需要除法，适合任意模数：

$$
\binom nk=\binom{n-1}{k-1}+\binom{n-1}{k}.
$$

若模数是质数且预处理范围小于模数，可预处理阶乘与逆阶乘，$O(1)$ 求组合数。

```cpp
fac[0] = 1;
for (int i = 1; i <= N; ++i) fac[i] = fac[i - 1] * i % MOD;
ifac[N] = modPow(fac[N], MOD - 2, MOD);
for (int i = N; i >= 1; --i) ifac[i - 1] = ifac[i] * i % MOD;
auto C = [&](int n, int k) -> long long {
    if (k < 0 || k > n) return 0;
    return fac[n] * ifac[k] % MOD * ifac[n - k] % MOD;
};
```

## 多重集排列

总共 $n$ 个元素，其中同类数量为 $c_1,c_2,\dots$，不同排列数：

$$\frac{n!}{\prod c_i!}.$$

例如 `AABC` 有 $4!/2!=12$ 种排列。取模时分母需要逆元，必须满足可逆条件。

## 圆排列与错排

$n$ 个不同元素围成一圈，若旋转视为相同，有 $(n-1)!$ 种。

错排要求每个元素都不在原位置。设 $D_n$ 为方案数：

$$D_n=(n-1)(D_{n-1}+D_{n-2}),\quad D_0=1,D_1=0.$$

例：3 封信放入错误信封只有 `(2,3,1)` 与 `(3,1,2)` 两种，$D_3=2$。

## 鸽巢原理

把 $N$ 个对象放进 $k$ 个盒子，至少一个盒子有 $\lceil N/k\rceil$ 个。常用于证明“必然存在”，而不是直接构造。

例：任取 13 个人，至少两人出生月份相同，因为只有 12 个月。

## 二项式定理

$$
(x+y)^n=\sum_{k=0}^{n}\binom nkx^ky^{n-k}.
$$

它把多项式系数与组合选择联系起来：选择 $k$ 个括号取 $x$，其余取 $y$，共有 $\binom nk$ 种。

## 容斥原理

两个集合：$|A\cup B|=|A|+|B|-|A\cap B|$。多个集合时，对非空集合族交集按所选集合数奇加偶减。

### 例题：不超过 $n$ 且能被 2 或 3 整除

答案为：

$$\left\lfloor\frac n2\right\rfloor+\left\lfloor\frac n3\right\rfloor-\left\lfloor\frac n6\right\rfloor.$$

最后一项减去同时被 2、3 整除而重复计算的数。多个条件的容斥通常枚举条件子集，复杂度 $O(2^k)$。

## Catalan 数

Catalan 数计数许多“前缀始终合法”的结构：合法括号序列、出栈序列、凸多边形三角剖分等。

$$
C_n=\frac1{n+1}\binom{2n}{n},\qquad
C_n=\sum_{i=0}^{n-1}C_iC_{n-1-i}.
$$

例：两对括号有 `(())`、`()()` 两种，$C_2=2$。若取模下 $n+1$ 不可逆，应使用递推或其他适合模数的方法，不能直接做除法。

## 真题：P2822 组合数问题

[洛谷 P2822 组合数问题](https://www.luogu.com.cn/problem/P2822) 多次询问满足 $0\le i\le n$、$0\le j\le\min(i,m)$ 且 $\binom ij$ 能被给定 $k$ 整除的数对数量。

这里只关心是否模 $k$ 为 0，无需计算巨大组合数。先用 Pascal 递推预处理组合数模 $k$，再对“是否为 0”的二维表做前缀和。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int testCases, modulus;
    cin >> testCases >> modulus;
    const int LIMIT = 2000;
    vector<vector<int>> combination(LIMIT + 1,
        vector<int>(LIMIT + 1));
    vector<vector<int>> prefix(LIMIT + 2,
        vector<int>(LIMIT + 2));

    combination[0][0] = 1 % modulus;
    for (int n = 1; n <= LIMIT; ++n) {
        combination[n][0] = 1 % modulus;
        for (int m = 1; m <= n; ++m)
            combination[n][m] = (combination[n - 1][m - 1]
                               + combination[n - 1][m]) % modulus;
    }

    for (int n = 0; n <= LIMIT; ++n) {
        for (int m = 0; m <= LIMIT; ++m) {
            int divisible = (m <= n && combination[n][m] == 0);
            prefix[n + 1][m + 1] = prefix[n][m + 1] + prefix[n + 1][m]
                                 - prefix[n][m] + divisible;
        }
    }

    while (testCases--) {
        int n, m;
        cin >> n >> m;
        m = min(m, n);
        cout << prefix[n + 1][m + 1] << '\n';
    }
    return 0;
}
```

预处理时间、空间均为 $O(N^2)$，每次询问 $O(1)$。组合数模数不要求是质数，因为递推只有加法，不涉及逆元。

## 易错点

- 可区分与不可区分对象混淆。
- 圆排列是否还把镜像视为相同没有按题意判断。
- 合数模数下直接用费马逆元。
- 容斥漏空集或正负号反转。
- 组合数参数越界未返回 0。
