# 字符串哈希

字符串哈希把字符串看成某个进制数。预处理前缀哈希后，可在 $O(1)$ 时间得到任意子串的指纹，常用于子串相等、重复模式和回文判定。

## 多项式滚动哈希

选底数 $B$ 和模数 $M$，字符映射为正整数。定义：

$$
H_i=(H_{i-1}\cdot B+value(s_i))\bmod M.
$$

同时预处理 $P_i=B^i\bmod M$。1-based 子串 $s[l..r]$ 的哈希为：

$$
hash(l,r)=H_r-H_{l-1}\cdot P_{r-l+1}\pmod M.
$$

第二项把前缀 $s[1..l-1]$ 左移到与 $H_r$ 相同的位权，再相减。

```cpp
struct StringHash {
    static const long long MOD = 1000000007;
    static const long long BASE = 911382323;
    vector<long long> h, power;

    StringHash(const string& s) : h(s.size() + 1), power(s.size() + 1, 1) {
        for (int i = 1; i <= (int)s.size(); ++i) {
            h[i] = (h[i - 1] * BASE + (unsigned char)s[i - 1] + 1) % MOD;
            power[i] = power[i - 1] * BASE % MOD;
        }
    }
    long long get(int l, int r) const { // 1-based 闭区间
        return (h[r] - h[l - 1] * power[r - l + 1] % MOD + MOD) % MOD;
    }
};
```

底数必须小于模数且不要选 0、1 这类退化值。也可利用 `unsigned long long` 自然溢出模 $2^{64}$，代码更短但仍可能碰撞。

## 例题：最长相同前缀

给两个后缀起点 $i,j$，求它们的最长公共前缀。长度具有单调性：若前 $k$ 个字符相等，则更短长度也相等。二分长度，每次用两个子串哈希 $O(1)$ 比较，总查询 $O(\log n)$。

例如 `banana` 中起点 2 的 `anana` 与起点 4 的 `ana`，长度 3 的哈希相等，长度 4 越过短后缀或不相等，因此 LCP 为 3。

## 回文判定

对原串和反转串分别建哈希。原串 `[l,r]` 在反转串中对应 `[n-r+1,n-l+1]`，哈希相同则很可能是回文。

## 碰撞与双哈希

哈希相同只是高概率相等。要求绝对正确时直接比较原串或使用确定性字符串算法；通常可用两个不同大质数模数组成哈希对，把碰撞概率降得很低。不能用同一个模数下简单改变底数后宣称完全独立。

## 易错点

- 字符映射包含 0，导致前导字符信息弱化。
- 子串长度指数写错一位。
- 相减后未加模数规范到非负。
- 比较不同长度子串时未统一位权却直接比前缀哈希。
- 把哈希相等当成数学上的必然相等。
