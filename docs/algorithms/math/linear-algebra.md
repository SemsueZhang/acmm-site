# 矩阵与高斯消元

## 向量与矩阵

向量可看成有序数列；矩阵是二维数表。矩阵加减要求形状相同，逐项运算。$A$ 为 $n\times m$、$B$ 为 $m\times p$ 时，乘积 $C=AB$ 为 $n\times p$：

$$
C_{i,j}=\sum_{k=1}^{m}A_{i,k}B_{k,j}.
$$

矩阵乘法满足结合律但通常不满足交换律。单位矩阵 $I$ 的对角线为 1，满足 $AI=IA=A$；转置把行列交换。

### 例题：两步路径计数

无权有向图邻接矩阵为 $A$，则 $(A^2)_{i,j}=\sum_k A_{i,k}A_{k,j}$，正好统计从 $i$ 经一个中间点到 $j$ 的两步路径数。一般地，$(A^t)_{i,j}$ 统计长度恰为 $t$ 的游走数。

矩阵快速幂见[快速幂](fast-power.md)。

## 高斯消元

线性方程组可写成增广矩阵。逐列选择主元，把该列其他行消为 0，最终得到阶梯形或最简形。

### 浮点版本步骤

对第 `col` 列：

1. 在未处理行中选绝对值最大的系数作为主元，减小误差。
2. 与当前行交换。
3. 主元行除以主元，使其变为 1。
4. 用主元行消去其他行的该列。

每一步只做交换两行、某行乘非零常数、某行加上另一行的倍数。这三类初等行变换都不会改变方程组的解集，因此消元前后的方程组等价。算法结束后，主元列把对应变量唯一确定；矛盾行表示无解，自由变量表示存在多个解。这就是判断三种解况的正确性依据。

```cpp
const double EPS = 1e-10;
int row = 0;
vector<int> where(m, -1);
for (int col = 0; col < m && row < n; ++col) {
    int pivot = row;
    for (int i = row; i < n; ++i)
        if (abs(a[i][col]) > abs(a[pivot][col])) pivot = i;
    if (abs(a[pivot][col]) < EPS) continue;
    swap(a[pivot], a[row]);
    where[col] = row;

    double div = a[row][col];
    for (int j = col; j <= m; ++j) a[row][j] /= div;
    for (int i = 0; i < n; ++i) if (i != row) {
        double factor = a[i][col];
        for (int j = col; j <= m; ++j)
            a[i][j] -= factor * a[row][j];
    }
    ++row;
}
```

这里 `a` 有 $n$ 行、$m+1$ 列，最后一列是常数项。

## 判断解的情况

- 某行所有变量系数为 0，常数却非 0：无解。
- 无矛盾且每个变量都有主元：唯一解。
- 无矛盾但存在无主元变量：无穷多解（自由变量）。

### 手算例题

$x+y=3$，$2x-y=0$。第二行减两倍第一行得到 $-3y=-6$，所以 $y=2,x=1$。

若第二式改为 $2x+2y=7$，消元得到 $0=1$，无解；若为 $2x+2y=6$，得到 $0=0$，有无穷多解。

## 模意义下消元

在质数模 $p$ 下，把“除以主元”替换为乘主元逆元。若模数不是质数，非零元素不一定可逆，需要更谨慎的整数消元方法。

## 真题：P3389 高斯消元

[洛谷 P3389【模板】高斯消元法](https://www.luogu.com.cn/problem/P3389) 给出 $n$ 个方程和 $n$ 个未知数；若不存在唯一解则输出 `No Solution`。每列选择绝对值最大的主元，可以减轻浮点误差。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<vector<double>> matrix(n, vector<double>(n + 1));
    for (auto& row : matrix)
        for (double& value : row) cin >> value;

    const double EPS = 1e-9;
    for (int column = 0; column < n; ++column) {
        int pivot = column;
        for (int row = column; row < n; ++row)
            if (fabs(matrix[row][column]) > fabs(matrix[pivot][column]))
                pivot = row;
        if (fabs(matrix[pivot][column]) < EPS) {
            cout << "No Solution\n";
            return 0;
        }
        swap(matrix[pivot], matrix[column]);

        double divisor = matrix[column][column];
        for (int j = column; j <= n; ++j) matrix[column][j] /= divisor;
        for (int row = 0; row < n; ++row) {
            if (row == column) continue;
            double factor = matrix[row][column];
            for (int j = column; j <= n; ++j)
                matrix[row][j] -= factor * matrix[column][j];
        }
    }

    cout << fixed << setprecision(2);
    for (int i = 0; i < n; ++i) cout << matrix[i][n] << '\n';
    return 0;
}
```

消元需要三层循环，时间 $O(n^3)$、空间 $O(n^2)$。更一般的线性方程组还需区分无解与无穷多解；本题把二者都归为“非唯一解”。

## 易错点

- 矩阵维度不匹配仍相乘。
- 快速幂初始结果不是单位矩阵。
- 浮点数直接与 0 用 `==` 比较。
- 只算出一组数，却没有检查矛盾行和自由变量。
- 模数下对不可逆主元直接做除法。
