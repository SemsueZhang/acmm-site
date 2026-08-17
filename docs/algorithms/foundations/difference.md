# 差分：批量区间修改的端点记账法

差分适用于“进行很多次区间加，最后统一恢复数组”的离线场景。它与前缀和互为逆运算。

## 为什么只改两个位置

对 1 下标数组定义

$$
d_1=a_1,\qquad d_i=a_i-a_{i-1}\ (i\ge 2).
$$

于是 $a_i=\sum_{k=1}^{i}d_k$。若要给闭区间 $[l,r]$ 的所有数加 $x$，执行

```cpp
diff[l] += x;
diff[r + 1] -= x;
```

正确性可以直接从前缀恢复过程看出：当下标走到 $l$ 时累计量多出 $x$，因此 $l\ldots r$ 都增加 $x$；走到 $r+1$ 时又减去 $x$，之后不再受影响。每次修改都满足这一不变量，多次修改只是把贡献相加。

二维差分同理。给闭矩形 $[x_1,x_2]\times[y_1,y_2]$ 加 1，需要修改四个角：

```text
diff[x1][y1]         += 1
diff[x2+1][y1]       -= 1
diff[x1][y2+1]       -= 1
diff[x2+1][y2+1]     += 1
```

可以把它理解为：从左上开始贡献，越过下边或右边时取消；右下区域被取消两次，所以补回一次。

## 真题：P3397 地毯

[洛谷 P3397 地毯](https://www.luogu.com.cn/problem/P3397) 要求在全部矩形铺设结束后，输出每个格子的覆盖次数。逐个修改矩形中的格子，最坏需要 $O(mn^2)$；二维差分把一块地毯压缩成四次端点修改。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, carpetCount;
    cin >> n >> carpetCount;
    vector<vector<int>> diff(n + 2, vector<int>(n + 2, 0));

    while (carpetCount--) {
        int x1, y1, x2, y2;
        cin >> x1 >> y1 >> x2 >> y2;
        ++diff[x1][y1];
        --diff[x2 + 1][y1];
        --diff[x1][y2 + 1];
        ++diff[x2 + 1][y2 + 1];
    }

    for (int row = 1; row <= n; ++row) {
        for (int column = 1; column <= n; ++column) {
            diff[row][column] += diff[row - 1][column]
                               + diff[row][column - 1]
                               - diff[row - 1][column - 1];
            cout << diff[row][column] << " \n"[column == n];
        }
    }
    return 0;
}
```

每块地毯修改 $O(1)$，恢复整个网格需要 $O(n^2)$，总时间 $O(m+n^2)$、空间 $O(n^2)$。

## 什么时候不能用

若每次修改后立刻询问某个区间，普通差分数组尚未恢复，无法快速回答。此时应使用树状数组或线段树。还要为 `r+1`、`x2+1` 和 `y2+1` 多开边界空间。
