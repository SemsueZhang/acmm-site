# 分治与扫描线

## 分治

分治把规模为 $n$ 的问题拆成若干更小的同类问题，递归求解，再合并答案。典型结构是：

```text
solve(l, r):
    若区间足够小，直接返回
    mid = (l + r) / 2
    solve(l, mid)
    solve(mid + 1, r)
    合并跨过 mid 的答案
```

若拆成两个 $n/2$ 子问题，合并花 $O(n)$，递推式 $T(n)=2T(n/2)+O(n)$，总复杂度 $O(n\log n)$。

### 例题：逆序对

若 $i<j$ 且 $a_i>a_j$，则 $(i,j)$ 是一个逆序对。归并排序合并两个有序区间时，若右侧当前元素小于左侧当前元素，那么左侧从当前位置到末尾的所有元素都与它构成逆序对。

```cpp
long long mergeCount(vector<int>& a, vector<int>& tmp, int l, int r) {
    if (l >= r) return 0;
    int mid = l + (r - l) / 2;
    long long ans = mergeCount(a, tmp, l, mid)
                  + mergeCount(a, tmp, mid + 1, r);
    int i = l, j = mid + 1, k = l;
    while (i <= mid && j <= r) {
        if (a[i] <= a[j]) tmp[k++] = a[i++];
        else {
            tmp[k++] = a[j++];
            ans += mid - i + 1;
        }
    }
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r) tmp[k++] = a[j++];
    for (int p = l; p <= r; ++p) a[p] = tmp[p];
    return ans;
}
```

`2 4 1 3` 的左右两半分别排好后，合并时 `1` 小于 `2`，一次贡献两个逆序对 `(2,1),(4,1)`；`3` 小于 `4`，再贡献一个。答案为 3。

## 扫描线

扫描线把二维事件按某一坐标排序，然后从左到右处理；数据结构维护与扫描线相交的一维信息。关键是把“持续存在的对象”转成进入事件和离开事件。

### 例题：最多同时进行的活动

每个活动占用半开区间 $[l,r)$，求任意时刻最多有多少活动同时进行。创建事件 `(l,+1)` 和 `(r,-1)`，按坐标排序；同坐标时先处理离开，再处理进入，因为 $[l,r)$ 在 $r$ 时刻已经结束。

```cpp
vector<pair<int,int>> events;
for (auto [l, r] : seg) {
    events.push_back({l, +1});
    events.push_back({r, -1});
}
sort(events.begin(), events.end(), [](auto a, auto b) {
    if (a.first != b.first) return a.first < b.first;
    return a.second < b.second; // -1 在 +1 前
});
int now = 0, ans = 0;
for (auto [x, delta] : events) {
    now += delta;
    ans = max(ans, now);
}
```

复杂度 $O(n\log n)$。矩形面积并、交点计数等提高题仍遵循“事件排序 + 维护截面”的骨架，只是一维维护部分会换成离散化、树状数组或线段树。

## 进阶例题：矩形面积并

每个矩形用半开区域 $[x_1,x_2)\times[y_1,y_2)$ 表示。为它创建两条竖直事件：在 $x_1$ 对 $y$ 区间覆盖次数加 1，在 $x_2$ 减 1。设处理完上一个横坐标事件后，$y$ 轴被覆盖总长度为 `coveredY`，那么扫到新坐标 $x$ 前新增面积：

$$
(x-x_{prev})\times coveredY.
$$

同一 $x$ 的事件要成组处理：先用旧覆盖长度结算到当前 $x$ 的面积，再统一更新覆盖状态。$y$ 坐标很大时先离散化；线段树叶子代表相邻坐标形成的段 `[ys[i],ys[i+1])`，节点维护覆盖次数和真实覆盖长度，而不是把离散编号差当长度。

例如矩形 $[0,2)\times[0,1)$ 与 $[1,3)\times[0,2)$：

- $x\in[0,1)$ 时覆盖长度 1，贡献 1；
- $x\in[1,2)$ 时覆盖长度 2，贡献 2；
- $x\in[2,3)$ 时覆盖长度 2，贡献 2。

面积并为 5，也等于两矩形面积 $2+4$ 减去重叠面积 1。事件排序 $O(n\log n)$，若线段树每次更新 $O(\log n)$，总复杂度 $O(n\log n)$。

## 易错点

- 分治只统计左右子问题，漏掉跨越中点的答案。
- 合并后没有恢复有序性，导致上一层前提失效。
- 逆序对答案可能达到 $n(n-1)/2$，必须用 `long long`。
- 扫描线没有先确定区间是闭区间还是半开区间，同坐标事件顺序错误。
- 离散坐标上维护长度时，节点代表的是坐标间隙而不是单个坐标点。
