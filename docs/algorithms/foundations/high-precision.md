# 高精度整数

当整数超过 `long long`（约 $9\times10^{18}$）时，可用数组按位存储。为了便于进位，通常低位在前；还可用 $10^9$ 为一组降低常数。这里用十进制逐位表示来讲清原理。

## 加法

模拟竖式，从低位到高位计算 `a[i]+b[i]+carry`，当前位取模 10，进位除以 10。

### 例题：计算 $999+37$

低位数组分别是 `[9,9,9]`、`[7,3]`：个位得到 16，写 6 进 1；十位得到 13，写 3 进 1；百位得到 10，写 0 进 1；最后补 1，结果 `[6,3,0,1]`，倒序输出 `1036`。

```cpp
string add(string a, string b) {
    reverse(a.begin(), a.end());
    reverse(b.begin(), b.end());
    string c;
    int carry = 0;
    for (int i = 0; i < (int)max(a.size(), b.size()) || carry; ++i) {
        int sum = carry;
        if (i < (int)a.size()) sum += a[i] - '0';
        if (i < (int)b.size()) sum += b[i] - '0';
        c.push_back(char('0' + sum % 10));
        carry = sum / 10;
    }
    reverse(c.begin(), c.end());
    return c.empty() ? "0" : c;
}
```

## 减法

先比较绝对值并确定符号，再保证 $a\ge b$。逐位计算 `a[i]-b[i]-borrow`；若小于 0，就加 10 并向高位借 1。最后删除高位多余的 0，但至少保留一位。

## 乘法

每对数位相乘并累加到 `c[i+j]`，然后统一进位。两个长度分别为 $n,m$ 的整数相乘复杂度为 $O(nm)$。

```cpp
vector<int> multiply(const vector<int>& a, const vector<int>& b) {
    vector<int> c(a.size() + b.size() + 1, 0);
    for (int i = 0; i < (int)a.size(); ++i)
        for (int j = 0; j < (int)b.size(); ++j)
            c[i + j] += a[i] * b[j];
    for (int i = 0; i + 1 < (int)c.size(); ++i) {
        c[i + 1] += c[i] / 10;
        c[i] %= 10;
    }
    while (c.size() > 1 && c.back() == 0) c.pop_back();
    return c;
}
```

## 高精度除以单精度

按正常书写顺序从高位到低位：当前被除数 `cur = remainder * 10 + digit`，商位为 `cur / b`，新余数为 `cur % b`。复杂度 $O(n)$。

## 易错点

- 结果为 0 时把所有数位都删掉。
- 减法未先比较大小，借位逻辑无法表示负数。
- 使用更大进制分组时，中间乘积没有用 `long long`。
- 输入前导零没有规范化，导致大小比较错误。
