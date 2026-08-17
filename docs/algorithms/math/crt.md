# 中国剩余定理：合并同余条件

中国剩余定理（CRT）解决

$$
x\equiv a_i\pmod{m_i}\quad (1\le i\le n)
$$

在模数两两互质时的合并问题。令 $M=\prod_i m_i$、$M_i=M/m_i$，再取 $t_i$ 为 $M_i$ 模 $m_i$ 的逆元，则

$$
x\equiv\sum_i a_iM_it_i\pmod M.
$$

## 构造为什么正确

固定某个模数 $m_j$：当 $i\ne j$ 时，$M_i$ 含因子 $m_j$，对应项模 $m_j$ 为 0；当 $i=j$ 时，$M_jt_j\equiv1\pmod{m_j}$，该项为 $a_j$。因此构造同时满足全部同余式。

若另有一个解 $y$，则 $x-y$ 被每个两两互质的 $m_i$ 整除，从而被乘积 $M$ 整除，所以模 $M$ 意义下解唯一。

手算 $x\equiv2\pmod3$、$x\equiv3\pmod5$：$M=15$，两个部分的逆元均为 2，得到 $x\equiv2\cdot5\cdot2+3\cdot3\cdot2=38\equiv8\pmod{15}$。

## 真题：P1495 曹冲养猪

[洛谷 P1495 曹冲养猪](https://www.luogu.com.cn/problem/P1495) 给出两两互质的模数和余数，要求最小非负解，正是标准 CRT。下面用 `__int128` 保存中间乘积，避免先在 `long long` 中溢出。

```cpp
#include <bits/stdc++.h>
using namespace std;

long long extendedGcd(long long a, long long b, long long& x, long long& y) {
    if (b == 0) {
        x = 1;
        y = 0;
        return a;
    }
    long long nextX, nextY;
    long long g = extendedGcd(b, a % b, nextX, nextY);
    x = nextY;
    y = nextX - (a / b) * nextY;
    return g;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int equationCount;
    cin >> equationCount;
    vector<long long> modulus(equationCount), remainder(equationCount);
    long long product = 1;
    for (int i = 0; i < equationCount; ++i) {
        cin >> modulus[i] >> remainder[i];
        product *= modulus[i];
    }

    __int128 answer = 0;
    for (int i = 0; i < equationCount; ++i) {
        long long partialProduct = product / modulus[i];
        long long inverse, unused;
        extendedGcd(partialProduct, modulus[i], inverse, unused);
        inverse = (inverse % modulus[i] + modulus[i]) % modulus[i];
        answer = (answer + (__int128)remainder[i] * partialProduct % product
                         * inverse) % product;
    }
    cout << static_cast<long long>(answer) << '\n';
    return 0;
}
```

扩展欧几里得对每个方程花 $O(\log m_i)$，总时间 $O(\sum\log m_i)$，空间 $O(n)$。这里仍假设模数乘积能放入 `long long`；若模数不互质，逆元可能不存在，应改用扩展 CRT，并先检查余数差能否被 gcd 整除。

## 易错点

- 没有核对“两两互质”前提就使用标准 CRT。
- 把 $M_i$ 的逆元方向写反。
- 只防最终答案溢出，却让模数乘积或三项乘积先溢出。
- 题目要求最小正解时，没有区分 0 与模数乘积。
