# 贪心

贪心算法在每一步选择当前看来最优的方案，并且不撤销。难点不在“选最小/最大”，而在于证明局部选择一定能通向全局最优。

## 证明工具：交换论证

常见证明方式是：取任意一个最优解，如果它第一步没有使用贪心选择，就把其中某个选择与贪心选择交换，并证明交换后仍合法且答案不变差。于是总存在一个以贪心选择开头的最优解；不断重复即可。

## 例题一：最多不相交区间

给出若干闭区间，最多选择多少个两两不相交的区间？约定端点相等也算相交。

策略：按右端点从小到大排序，每次选择第一个左端点严格大于上次右端点的区间。

为什么不是“选最短区间”？短区间可能位于中间，反而同时挡住左右两侧。最早结束的区间给后续留下的空间最大。

交换证明：设贪心首选区间为 $G$，某个最优解首选 $O$。因为 $G$ 的右端点不晚于 $O$，用 $G$ 替换 $O$ 后，原来在 $O$ 后面的区间仍能放在 $G$ 后面，区间数不减少。

```cpp
sort(seg.begin(), seg.end(), [](auto a, auto b) {
    if (a.second != b.second) return a.second < b.second;
    return a.first < b.first;
});
int ans = 0;
long long lastRight = LLONG_MIN;
for (auto [l, r] : seg) {
    if (l > lastRight) {
        ++ans;
        lastRight = r;
    }
}
```

排序 $O(n\log n)$，扫描 $O(n)$。

## 例题二：P1090 合并果子

[洛谷 P1090 合并果子](https://www.luogu.com.cn/problem/P1090) 中，每次合并两堆果子，代价是两堆重量之和；新堆可继续合并。要求把所有果子合并成一堆的最小总代价。

每次取最轻的两堆合并。重量越早参与合并，被重复计入的次数越多，因此应让小重量承担更深的层数。用小根堆维护当前最小的两堆：

```cpp
priority_queue<long long, vector<long long>, greater<long long>> pq;
for (long long x : a) pq.push(x);
long long ans = 0;
while (pq.size() > 1) {
    long long x = pq.top(); pq.pop();
    long long y = pq.top(); pq.pop();
    ans += x + y;
    pq.push(x + y);
}
```

例如 `1,2,9`：先合并 `1+2=3`，总代价 $3+12=15$；若先合并 `2+9=11`，总代价 $11+12=23$。

完整实现如下。答案可能超过 `int`，因此堆元素和总费用都使用 `long long`。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    priority_queue<long long, vector<long long>, greater<long long>> heap;
    for (int i = 0; i < n; ++i) {
        long long weight;
        cin >> weight;
        heap.push(weight);
    }

    long long answer = 0;
    while (heap.size() > 1) {
        long long first = heap.top(); heap.pop();
        long long second = heap.top(); heap.pop();
        answer += first + second;
        heap.push(first + second);
    }
    cout << answer << '\n';
    return 0;
}
```

每次合并执行两次删除和一次插入，共 $n-1$ 轮，时间 $O(n\log n)$、空间 $O(n)$。

## 什么时候不能贪心

若当前选择会改变未来选择的价值，而且不能通过交换保持最优，通常需要 DP。例如 0/1 背包按“单位价值”排序是错误的：容量 4，物品 $(3,5),(2,3),(2,3)$，选单位价值最高的第一件只能得 5，后两件却能得 6。

## 常见模型

- 区间调度：常按结束时间排序。
- 最小等待时间：常按处理时间升序。
- Huffman 合并：反复取最小的两个。
- Kruskal：按边权从小到大取不成环的边。

每次写贪心前都应明确：选择规则、可行性、交换或归纳证明、相等时的处理。
