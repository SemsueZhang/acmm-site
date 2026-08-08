# 初等数论

## gcd、lcm 与裴蜀定理

欧几里得算法基于 $\gcd(a,b)=\gcd(b,a\bmod b)$：

```cpp
long long gcd(long long a, long long b) {
    while (b) { long long r = a % b; a = b; b = r; }
    return abs(a);
}
```

`lcm(a,b)=a/gcd(a,b)*b`，先除再乘降低溢出风险。

裴蜀定理：存在整数 $x,y$ 使 $ax+by=\gcd(a,b)$。因此方程 $ax+by=c$ 有整数解当且仅当 $\gcd(a,b)\mid c$。

扩展欧几里得在求 gcd 的同时给出系数：

```cpp
long long exgcd(long long a, long long b, long long& x, long long& y) {
    if (b == 0) { x = 1; y = 0; return a; }
    long long x1, y1;
    long long g = exgcd(b, a % b, x1, y1);
    x = y1;
    y = x1 - (a / b) * y1;
    return g;
}
```

## 质数、分解与筛法

试除分解只需枚举到 $p^2\le n$；循环结束若 $n>1$，剩余部分是一个质因子。判断 `p*p<=n` 可能溢出，可写 `p<=n/p`。

埃氏筛从每个质数 $p$ 的 $p^2$ 开始标记倍数，复杂度 $O(n\log\log n)$。线性筛保证每个合数被其最小质因子筛一次：

```cpp
vector<int> primes, minPrime(n + 1);
for (int i = 2; i <= n; ++i) {
    if (!minPrime[i]) { minPrime[i] = i; primes.push_back(i); }
    for (int p : primes) {
        if (p > minPrime[i] || 1LL * i * p > n) break;
        minPrime[i * p] = p;
    }
}
```

## 同余与逆元

$a\equiv b\pmod m$ 表示 $m\mid(a-b)$。加减乘可直接保持同余，但除法必须乘逆元。

若 $ax\equiv1\pmod m$，则 $x$ 是 $a$ 模 $m$ 的逆元。逆元存在当且仅当 $\gcd(a,m)=1$。可用扩展欧几里得求：若 `gcd(a,m)==1`，返回 `(x%m+m)%m`。

当 $p$ 为质数且 $p\nmid a$，费马小定理给出 $a^{p-1}\equiv1\pmod p$，所以 $a^{-1}\equiv a^{p-2}\pmod p$。

## 欧拉函数与欧拉定理

$\varphi(n)$ 是 $1..n$ 中与 $n$ 互质的整数个数。若 $n=\prod p_i^{k_i}$：

$$
\varphi(n)=n\prod_{p_i\mid n}\left(1-\frac1{p_i}\right).
$$

若 $\gcd(a,n)=1$，欧拉定理：$a^{\varphi(n)}\equiv1\pmod n$。费马小定理是 $n$ 为质数时的特例。

### 例题：$3^{100}\bmod 10$

$\varphi(10)=4$ 且 $\gcd(3,10)=1$，所以指数可按 4 缩减：$100\bmod4=0$，答案 $3^4\bmod10=1$。

若底数与模数不互质，不能直接用欧拉定理缩指数。

## 威尔逊定理

整数 $p>1$ 为质数当且仅当 $(p-1)!\equiv-1\pmod p$。它有理论价值，但直接计算阶乘判素数是 $O(p)$，实战判素数通常不用它。

## 中国剩余定理（CRT）

求满足 $x\equiv a_i\pmod{m_i}$ 的 $x$。当模数两两互质时，令 $M=\prod m_i$、$M_i=M/m_i$，求 $M_i$ 模 $m_i$ 的逆元 $t_i$：

$$
x\equiv\sum_i a_iM_it_i\pmod M.
$$

### 手算例题

$x\equiv2\pmod3$，$x\equiv3\pmod5$。$M=15$。对模 3，$M_1=5$，逆元为 2；对模 5，$M_2=3$，逆元为 2：

$$x\equiv2\cdot5\cdot2+3\cdot3\cdot2=38\equiv8\pmod{15}.$$

模数不互质时需用扩展 CRT 合并两条同余，并检查余数差能否被 gcd 整除。

## 真题：P1082 同余方程

[洛谷 P1082 同余方程](https://www.luogu.com.cn/problem/P1082) 要求最小正整数 $x$ 满足 $ax\equiv1\pmod b$。把同余改写为：

$$
ax+by=1.
$$

扩展欧几里得求出一组系数后，把 $x$ 规范到 $[0,b-1]$。题目保证逆元存在；通用实现还应检查 gcd 是否为 1。

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
    long long gcd = extendedGcd(b, a % b, nextX, nextY);
    x = nextY;
    y = nextX - (a / b) * nextY;
    return gcd;
}

int main() {
    long long a, modulus, x, y;
    cin >> a >> modulus;
    long long gcd = extendedGcd(a, modulus, x, y);
    if (gcd != 1) {
        cout << "No inverse\n";
        return 0;
    }
    x = (x % modulus + modulus) % modulus;
    cout << x << '\n';
    return 0;
}
```

欧几里得算法每次把参数替换为余数，时间 $O(\log\min(a,b))$，递归空间同阶。

## 易错点

- 负数 `%` 后仍为负。
- 在不互质时使用费马/欧拉逆元。
- CRT 中乘积溢出；必要时用 `__int128`。
- 把“定理给出必要条件”误当充分条件，或忘记定理前提。
- 素数筛把 0、1 当作质数。
