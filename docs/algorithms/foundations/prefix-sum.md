# 前缀和：把区间询问变成端点相减

前缀和适用于“数组不再修改，但要反复询问区间总和”的场景。它不是一个需要背诵的公式，而是在预处理阶段保存可复用的累积结果。

## 从朴素询问出发

若每次询问 $[l,r]$ 都遍历一遍，单次需要 $O(r-l+1)$，$m$ 次最坏为 $O(nm)$。定义

$$
s_i=\sum_{k=1}^{i}a_k,\qquad s_0=0,
$$

则 $s_r$ 包含 $a_1\ldots a_r$，$s_{l-1}$ 包含其中不属于答案的 $a_1\ldots a_{l-1}$，所以

$$
\sum_{k=l}^{r}a_k=s_r-s_{l-1}.
$$

这也给出了正确性证明：右端点前缀中的每一项恰好出现一次，减去左端点之前的项后，只剩 $[l,r]$，没有遗漏或重复。

## 手算一次

数组为 `2 1 4 3 5` 时，连同 $s_0$ 的前缀和为 `0 2 3 7 10 15`。询问 $[2,4]$ 得到 $s_4-s_1=10-2=8$，恰好是 $1+4+3$。

二维情形使用同一思想。令 $s_{i,j}$ 表示左上角 $(1,1)$ 到右下角 $(i,j)$ 的矩形和，则

$$
s_{i,j}=a_{i,j}+s_{i-1,j}+s_{i,j-1}-s_{i-1,j-1}.
$$

最后一项是容斥：上方与左方的交集被加了两次，需要减去一次。矩形 $[x_1,x_2]\times[y_1,y_2]$ 的答案也用四个前缀矩形容斥得到。

## 真题：P8218 求区间和

[洛谷 P8218 求区间和](https://www.luogu.com.cn/problem/P8218) 给出一个静态序列和多次区间和询问。关键词是“没有修改”和“多次询问”，正对应一次预处理、常数时间回答。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<long long> prefix(n + 1, 0);
    for (int i = 1; i <= n; ++i) {
        long long value;
        cin >> value;
        prefix[i] = prefix[i - 1] + value;
    }

    int queryCount;
    cin >> queryCount;
    while (queryCount--) {
        int left, right;
        cin >> left >> right;
        cout << prefix[right] - prefix[left - 1] << '\n';
    }
    return 0;
}
```

预处理时间 $O(n)$，每次询问 $O(1)$，空间 $O(n)$。若修改和询问交错出现，静态前缀和会失效，应考虑树状数组或线段树。

## 易错点

- 先统一下标与区间约定；本文全部使用 1 下标闭区间。
- 必须保留 $s_0=0$，这样 $l=1$ 无需特判。
- 前缀和可能达到“元素上界乘以元素个数”，通常应使用 `long long`。
- 二维容斥中，左上交集的符号是“减一次”。
