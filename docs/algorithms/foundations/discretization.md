# 离散化：只保留真正需要的顺序信息

当数值范围很大、实际出现的值却很少时，可以把不同值排序后映射到连续编号。离散化保留**相等关系与大小关系**，但不保留相邻距离。

## 构造与不变量

将所有可能出现的值放入 `values`，排序并去重。原值 $x$ 的编号是它在有序数组中的位置：

```cpp
sort(values.begin(), values.end());
values.erase(unique(values.begin(), values.end()), values.end());
int id = lower_bound(values.begin(), values.end(), x) - values.begin();
```

由于排序保持严格大小关系，任意 $x<y$ 都有 $id(x)<id(y)$；去重又保证 $x=y$ 当且仅当编号相同。这两个性质就是离散化正确性的核心不变量。

但若原坐标为 1 和 100，它们的编号可能相邻，却不能认为距离是 1。扫描线维护真实长度时必须使用 `values[i + 1] - values[i]`。

## 真题：P1955 程序自动分析

[洛谷 P1955 程序自动分析](https://www.luogu.com.cn/problem/P1955) 中变量编号很大，而每组数据只出现有限个编号。目标是判断若干“相等/不等”约束能否同时成立。

建模分三步：

1. 收集本组出现的所有变量编号并离散化；
2. 先用并查集合并所有相等约束；
3. 再检查每条不等约束的两端是否被合并到同一集合。

前两步建立“必须相等”的等价类。若一条不等约束落在同一等价类中就产生矛盾；反之，可以给每个等价类赋不同值，所有约束都能满足，因此判定既必要又充分。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Constraint {
    int leftValue;
    int rightValue;
    int equal;
};

class DisjointSet {
public:
    explicit DisjointSet(int size) : parent(size), rank(size, 0) {
        iota(parent.begin(), parent.end(), 0);
    }

    int find(int vertex) {
        if (parent[vertex] != vertex) {
            parent[vertex] = find(parent[vertex]);
        }
        return parent[vertex];
    }

    void unite(int first, int second) {
        first = find(first);
        second = find(second);
        if (first == second) return;
        if (rank[first] < rank[second]) swap(first, second);
        parent[second] = first;
        if (rank[first] == rank[second]) ++rank[first];
    }

private:
    vector<int> parent;
    vector<int> rank;
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int testCount;
    cin >> testCount;
    while (testCount--) {
        int constraintCount;
        cin >> constraintCount;
        vector<Constraint> constraints(constraintCount);
        vector<int> values;
        values.reserve(2 * constraintCount);

        for (Constraint& constraint : constraints) {
            cin >> constraint.leftValue >> constraint.rightValue >> constraint.equal;
            values.push_back(constraint.leftValue);
            values.push_back(constraint.rightValue);
        }

        sort(values.begin(), values.end());
        values.erase(unique(values.begin(), values.end()), values.end());
        auto getId = [&](int value) {
            return int(lower_bound(values.begin(), values.end(), value) - values.begin());
        };

        DisjointSet dsu(values.size());
        for (const Constraint& constraint : constraints) {
            if (constraint.equal == 1) {
                dsu.unite(getId(constraint.leftValue), getId(constraint.rightValue));
            }
        }

        bool possible = true;
        for (const Constraint& constraint : constraints) {
            if (constraint.equal == 0 &&
                dsu.find(getId(constraint.leftValue)) ==
                dsu.find(getId(constraint.rightValue))) {
                possible = false;
                break;
            }
        }
        cout << (possible ? "YES" : "NO") << '\n';
    }
    return 0;
}
```

设约束数为 $n$。排序和每次二分映射带来 $O(n\log n)$ 时间；并查集部分为 $O(n\alpha(n))$，总空间 $O(n)$。若频繁查询编号，可改用哈希表预存原值到编号的映射。

## 易错点

- 必须收集以后可能出现的全部值，不能处理到一半再改变已有编号。
- `unique` 只把重复元素移到末尾，必须配合 `erase` 真正删除。
- 离散化不保留距离；涉及区间长度或空隙时要回到原坐标。
- 本题必须先处理所有相等约束，再检查不等约束，输入顺序没有逻辑优先级。
