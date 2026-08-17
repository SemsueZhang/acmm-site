# 质数、分解与线性筛

## 从试除到筛法

若合数 $n=ab$，则 $a,b$ 不可能都大于 $\sqrt n$，所以判定一个数是否为质数只需试除到 $p\le n/p$。这一写法还避免了 `p * p` 溢出。

需要回答大量质数问题时，逐个试除会重复工作。埃氏筛从质数 $p$ 的 $p^2$ 开始标记倍数，复杂度 $O(n\log\log n)$。线性筛进一步规定：每个合数只由它的最小质因子筛掉一次。

## 线性筛的不变量

从小到大处理 $i$，维护已经发现的质数表。对质数 $p$ 枚举 $i\cdot p$，当 $p$ 等于 $i$ 的最小质因子后停止。

设合数 $x$ 的最小质因子为 $p$，则它唯一会在 $i=x/p$ 时由 $p$ 筛到。更小的质数不是 $x$ 的因子，更大的质数会因为已经越过 `minPrime[i]` 而停止。因此每个合数恰好处理一次，正确性和线性复杂度同时得到保证。

```cpp
vector<int> primes;
vector<int> minPrime(limit + 1, 0);
for (int value = 2; value <= limit; ++value) {
    if (minPrime[value] == 0) {
        minPrime[value] = value;
        primes.push_back(value);
    }
    for (int prime : primes) {
        if (prime > minPrime[value] || 1LL * value * prime > limit) break;
        minPrime[value * prime] = prime;
    }
}
```

## 真题：P3383 线性筛素数

[洛谷 P3383【模板】线性筛素数](https://www.luogu.com.cn/problem/P3383) 需要回答第 $k$ 个质数。一次筛出上界内所有质数后，每次询问直接按下标取值。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int limit, queryCount;
    cin >> limit >> queryCount;
    vector<int> minPrime(limit + 1, 0);
    vector<int> primes;
    for (int value = 2; value <= limit; ++value) {
        if (minPrime[value] == 0) {
            minPrime[value] = value;
            primes.push_back(value);
        }
        for (int prime : primes) {
            if (prime > minPrime[value] || 1LL * value * prime > limit) break;
            minPrime[value * prime] = prime;
        }
    }

    while (queryCount--) {
        int index;
        cin >> index;
        cout << primes[index - 1] << '\n';
    }
    return 0;
}
```

预处理时间 $O(n)$、空间 $O(n)$，每次询问 $O(1)$。0 和 1 都不是质数；乘积判断必须提升到 `long long`。
