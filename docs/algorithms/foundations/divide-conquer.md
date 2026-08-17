# 分治：拆分、递归与合并

分治把一个问题拆成若干更小的同类问题，分别求解后合并。它成立的关键不是“用了递归”，而是跨越子问题边界的答案能够在合并阶段高效统计。

## 归并分治的不变量

对区间 $[l,r]$：先递归处理左右两半，再在线性时间内合并。函数返回时维持两个不变量：

1. 区间 $[l,r]$ 已按值有序；
2. 该区间内部的目标答案已被完整统计。

若合并需要 $O(n)$，递推式为 $T(n)=2T(n/2)+O(n)$：每层处理的元素总数是 $n$，递归树有 $O(\log n)$ 层，总时间 $O(n\log n)$。

## 真题：P1908 逆序对

[洛谷 P1908 逆序对](https://www.luogu.com.cn/problem/P1908) 中，$i<j$ 且 $a_i>a_j$ 时，$(i,j)$ 是逆序对。枚举所有数对需要 $O(n^2)$。

递归已经统计了完全位于左半或右半的逆序对。合并两个有序区间时，若右侧当前值小于左侧当前值，那么左侧当前位置到末尾都比它大，一次贡献 `middle - i + 1` 个跨区间逆序对。反之取左侧值不会新增逆序对。三类逆序对互不重叠，因此答案完整且不重复。

```cpp
#include <bits/stdc++.h>
using namespace std;

long long mergeAndCount(vector<int>& values, vector<int>& buffer,
                        int left, int right) {
    if (left >= right) return 0;
    int middle = left + (right - left) / 2;
    long long answer = mergeAndCount(values, buffer, left, middle)
                     + mergeAndCount(values, buffer, middle + 1, right);

    int i = left;
    int j = middle + 1;
    int write = left;
    while (i <= middle && j <= right) {
        if (values[i] <= values[j]) {
            buffer[write++] = values[i++];
        } else {
            buffer[write++] = values[j++];
            answer += middle - i + 1;
        }
    }
    while (i <= middle) buffer[write++] = values[i++];
    while (j <= right) buffer[write++] = values[j++];
    for (int position = left; position <= right; ++position) {
        values[position] = buffer[position];
    }
    return answer;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> values(n), buffer(n);
    for (int& value : values) cin >> value;
    cout << mergeAndCount(values, buffer, 0, n - 1) << '\n';
    return 0;
}
```

时间复杂度 $O(n\log n)$，额外空间 $O(n)$。逆序对最多为 $n(n-1)/2$，答案必须使用 `long long`。相等的两个数不是逆序对，因此比较必须写成 `values[i] <= values[j]` 时先取左侧。
