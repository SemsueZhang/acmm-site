# 快速幂

快速幂利用指数的二进制表示，把 $a^b$ 从 $O(b)$ 次乘法降到 $O(\log b)$。

若 $b$ 为偶数，$a^b=(a^2)^{b/2}$；若为奇数，再额外乘一个 $a$。迭代时 `base` 表示当前二进制位对应的 $a^{2^k}$，`result` 累积指数位为 1 的部分。

```cpp
long long modPow(long long a, long long b, long long mod) {
    a %= mod;
    long long result = 1 % mod;
    while (b > 0) {
        if (b & 1LL) result = (__int128)result * a % mod;
        a = (__int128)a * a % mod;
        b >>= 1;
    }
    return result;
}
```

这里用 `__int128` 防止两个接近 $10^{18}$ 的数相乘溢出；若模数在 $10^9$ 量级，`long long` 乘积通常足够。

### 手算 $3^{13}$

$13=(1101)_2=8+4+1$。依次得到 $3^1,3^2,3^4,3^8$，只把第 0、2、3 位对应幂乘入结果，因此计算量与二进制位数有关。

## 矩阵快速幂

矩阵乘法满足结合律，因此同样可以二进制快速幂。斐波那契数满足：

$$
\begin{bmatrix}F_{n+1}\\F_n\end{bmatrix}
=
\begin{bmatrix}1&1\\1&0\end{bmatrix}^n
\begin{bmatrix}1\\0\end{bmatrix}.
$$

固定 $k\times k$ 矩阵一次乘法 $O(k^3)$，快速幂为 $O(k^3\log n)$。适合“状态维度很小、递推次数极大”的线性递推。

## 易错点

- `mod==1` 时初始结果应为 `1%mod=0`。
- 指数为 0 时答案是乘法单位元：整数为 1，矩阵为单位矩阵。
- 底数为负时要用 `(a%mod+mod)%mod` 规范。
- 在需要精确整数幂时继续取模。
- 矩阵乘法维度顺序错误，或单位矩阵初始化漏对角线。
