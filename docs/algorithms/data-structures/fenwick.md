# 树状数组（Fenwick Tree）

树状数组适合维护具有“前缀可合并、可作差”性质的信息，最常用的是单点修改与区间和。代码短、常数小，但不如线段树通用。

## `lowbit` 与节点含义

`lowbit(x)=x&(-x)` 取出二进制最低位的 1 所代表的值。`tree[x]` 保存长度为 `lowbit(x)`、右端点为 $x$ 的区间和：

$$
tree[x]=\sum_{i=x-lowbit(x)+1}^{x}a_i.
$$

例如 $x=12=(1100)_2$，`lowbit(12)=4`，所以 `tree[12]` 管理 `[9,12]`。

## 单点增加与前缀查询

```cpp
struct Fenwick {
    int n;
    vector<long long> tree;
    Fenwick(int n) : n(n), tree(n + 1, 0) {}

    void add(int x, long long delta) {
        for (; x <= n; x += x & -x) tree[x] += delta;
    }
    long long prefixSum(int x) const {
        long long ans = 0;
        for (; x > 0; x -= x & -x) ans += tree[x];
        return ans;
    }
    long long rangeSum(int l, int r) const {
        return prefixSum(r) - prefixSum(l - 1);
    }
};
```

更新时向上访问所有包含位置 $x$ 的节点；查询时不断删除当前前缀末尾负责的一段。每次二进制有效位发生变化，最多 $O(\log n)$ 次。

### 例题：动态区间和

初始数组 `1 3 2 5`。把位置 2 增加 4 后数组为 `1 7 2 5`；查询 `[2,4]` 得 `prefix(4)-prefix(1)=15-1=14`。

## 例题：逆序对

先把值离散化。从左到右处理 $a_i$，已经出现的数共有 $i-1$ 个，其中不大于 $a_i$ 的数量为 `prefixSum(rank[i])`，所以比它大的数量是：

```cpp
answer += (i - 1) - bit.prefixSum(rank[i]);
bit.add(rank[i], 1);
```

总复杂度 $O(n\log n)$。

## 区间增加、单点查询

在差分数组上建树状数组。对 `[l,r]` 加 $x$，执行 `add(l,x)`、`add(r+1,-x)`；位置 $p$ 的增量就是差分前缀和 `prefixSum(p)`。

## 易错点

- 下标必须从 1 开始；若 `x=0`，`x += lowbit(x)` 永远不变，造成死循环。
- 需要赋值时，先求旧值，再增加 `new-old`；树状数组基本操作是“增加”。
- 使用区间和公式时忘记 `l-1`。
- 用它维护普通区间最值却仍想通过两个前缀作差；最大值没有逆运算。
