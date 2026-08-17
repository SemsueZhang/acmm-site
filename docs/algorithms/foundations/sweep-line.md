# 扫描线：把持续区间改写成有序事件

扫描线的核心是按一个坐标顺序处理事件，同时维护当前截面的状态。最基本的版本处理一维区间；矩形面积并等提高题仍使用同一骨架，只是把截面状态交给线段树维护。

## 区间如何变成事件

对半开区间 $[l,r)$ 创建 `(l,+1)` 和 `(r,-1)`。从左向右扫描时，维护变量 `active`：处理完坐标 $x$ 的全部事件后，它等于覆盖 $[x,nextX)$ 的区间数。

这个不变量来自端点含义：每个已经开始但尚未结束的区间贡献 1，其他区间贡献 0。所有事件按坐标处理一次，就不会遗漏或重复任何区间。

同一坐标有多个事件时，应先合并增量再更新答案，或明确规定端点顺序。闭区间 $[l,r]$ 若坐标为整数，可转换为事件 `(l,+1)` 与 `(r+1,-1)`。

## 真题：AtCoder ABC014 C AtColor

[AtCoder ABC014 C - AtColor](https://atcoder.jp/contests/abc014/tasks/abc014_3) 给出若干整数闭区间，要求最多有多少区间覆盖同一个位置。下面不用坐标上界，而是直接排序事件，因此也适用于坐标很大的情况。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int intervalCount;
    cin >> intervalCount;
    vector<pair<int, int>> events;
    events.reserve(2 * intervalCount);
    for (int i = 0; i < intervalCount; ++i) {
        int left, right;
        cin >> left >> right;
        events.push_back({left, +1});
        events.push_back({right + 1, -1});
    }

    sort(events.begin(), events.end());
    int active = 0;
    int answer = 0;
    for (int i = 0; i < int(events.size()); ) {
        int coordinate = events[i].first;
        int delta = 0;
        while (i < int(events.size()) && events[i].first == coordinate) {
            delta += events[i].second;
            ++i;
        }
        active += delta;
        answer = max(answer, active);
    }
    cout << answer << '\n';
    return 0;
}
```

事件数为 $2n$，排序占 $O(n\log n)$ 时间，扫描占 $O(n)$ 时间，空间 $O(n)$。

## 向二维推广

矩形面积并把每个半开矩形 $[x_1,x_2)\times[y_1,y_2)$ 变成两条竖直事件：在 $x_1$ 给 $y$ 区间覆盖次数加 1，在 $x_2$ 减 1。若相邻事件横坐标相差 $\Delta x$，当前 $y$ 轴覆盖长度为 $L$，新增面积就是 $\Delta x\cdot L$。

这里离散化后的线段树叶子代表 `[ys[i],ys[i+1])`，长度是原坐标差而不是编号差。矩形面积并通常需要“区间加 + 全局覆盖长度”的专用线段树，应在掌握普通扫描线和懒标记后再学习。

## 易错点

- 未先说明闭区间还是半开区间，导致同坐标端点处理错误。
- 同一坐标的事件没有成组处理，状态含义在组内暂时失真。
- 把离散化编号差当作真实几何长度。
- 只排序事件，却没有写清扫描过程中维护的状态和不变量。
