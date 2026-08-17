# 同余与乘法逆元

同余 $a\equiv b\pmod m$ 表示 $m\mid(a-b)$。加、减、乘都能保持同余，但模意义下没有普通除法；只有当除数存在乘法逆元时，才能把“除以 $a$”改写成“乘 $a^{-1}$”。

## 逆元何时存在

若 $ax\equiv1\pmod m$，则存在整数 $y$ 使 $ax+my=1$。根据裴蜀定理，这恰好等价于

$$
\gcd(a,m)=1.
$$

因此“互质”既是必要条件也是充分条件。扩展欧几里得求出 $ax+my=1$ 的一组系数后，$x$ 就是逆元；用 `(x % m + m) % m` 把它规范到 $[0,m-1]$。

当模数 $p$ 为质数且 $p\nmid a$ 时，费马小定理给出 $a^{p-1}\equiv1\pmod p$，所以也可用快速幂计算 $a^{p-2}$。合数模数下不能无条件套用这一公式。

## 真题：P1082 同余方程

[洛谷 P1082 同余方程](https://www.luogu.com.cn/problem/P1082) 要求最小正整数 $x$ 满足 $ax\equiv1\pmod b$。把同余改写为 $ax+by=1$ 后，问题正好变成扩展欧几里得求系数。

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

    long long a, modulus;
    cin >> a >> modulus;
    long long inverse, coefficient;
    long long g = extendedGcd(a, modulus, inverse, coefficient);
    if (g != 1) {
        cout << "No inverse\n";
        return 0;
    }
    inverse = (inverse % modulus + modulus) % modulus;
    cout << inverse << '\n';
    return 0;
}
```

时间与递归空间都是 $O(\log\min(a,m))$。通用程序必须检查 gcd；题目保证有解并不代表模板可以省略前提。

## 易错点

- 把整数除法直接搬到取模表达式中。
- 模数是质数，却忘记 $a$ 不能被模数整除。
- C++ 的负数余数仍可能为负，只做一次 `% modulus` 不够。
- 多次连乘没有在中间取模，或乘积先溢出再取模。
