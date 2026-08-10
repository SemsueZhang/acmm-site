# 最大公约数与裴蜀定理

## 欧几里得算法为什么成立

设 $a=qb+r$。一个整数同时整除 $a,b$，当且仅当它同时整除 $b,r=a-qb$，因此

$$
\gcd(a,b)=\gcd(b,a\bmod b).
$$

不断把较大的参数替换为余数，第二个参数严格变小，最终到达 $gcd(g,0)=|g|$。这既证明了正确性，也证明了算法会终止。

```cpp
long long gcd(long long a, long long b) {
    a = abs(a);
    b = abs(b);
    while (b != 0) {
        long long remainder = a % b;
        a = b;
        b = remainder;
    }
    return a;
}
```

最小公倍数可写为 `a / gcd(a, b) * b`；先除再乘只能降低溢出风险，不能保证永不溢出。

## 裴蜀定理与扩展欧几里得

裴蜀定理说明所有整数线性组合 $ax+by$ 中，最小的正数是 $gcd(a,b)$。所以方程 $ax+by=c$ 有整数解，当且仅当 $gcd(a,b)\mid c$。

扩展欧几里得在递归返回时恢复系数。若已知

$$
bx_1+(a\bmod b)y_1=g,
$$

代入 $a\bmod b=a-\lfloor a/b\rfloor b$，得到

$$
ay_1+b(x_1-\lfloor a/b\rfloor y_1)=g.
$$

因此新系数为 $x=y_1$、$y=x_1-\lfloor a/b\rfloor y_1$。

```cpp
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
```

## 真题：P4549 裴蜀定理

[洛谷 P4549 裴蜀定理](https://www.luogu.com.cn/problem/P4549) 要求若干整数线性组合能得到的最小正整数。两个数时答案是 gcd；加入第三个数后，所有已有组合再与第三个数做线性组合，答案变成前缀 gcd。归纳可得最终答案是所有数绝对值的 gcd。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int count;
    cin >> count;
    long long answer = 0;
    for (int i = 0; i < count; ++i) {
        long long value;
        cin >> value;
        answer = std::gcd(answer, abs(value));
    }
    cout << answer << '\n';
    return 0;
}
```

欧几里得算法为 $O(\log\min(|a|,|b|))$；对 $n$ 个数依次求 gcd，时间 $O(n\log V)$、空间 $O(1)$。注意 C++ 中对最小负整数直接取 `abs` 可能溢出，竞赛中应结合输入范围选择类型。
