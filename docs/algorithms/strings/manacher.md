# Manacher 算法

Manacher 在线性时间内求每个位置为中心的最长回文半径。它复用已知回文区间的对称信息，避免从每个中心重复向两侧扩展。

## 统一奇偶回文

在原字符之间和两端插入分隔符。例如 `abba` 变成 `#a#b#b#a#`。变换后所有回文长度都是奇数：原串偶回文 `abba` 也有明确中心 `#`。

为避免越界，可再加两个不同哨兵：`^#a#b#b#a#$`。令 `p[i]` 表示以 `i` 为中心、向一侧能扩展的最大步数。

## 镜像复用

维护目前右端点最远的回文区间，其中心为 `center`、右端（开边界）为 `right`。

- 若 `i < right`，镜像位置 `mirror=2*center-i`，可先令 `p[i]=min(right-i,p[mirror])`。
- 然后从该半径继续暴力扩展。
- 若 `i+p[i]` 超过 `right`，更新最右区间。

```cpp
vector<int> manacher(const string& s) {
    string t = "^";
    for (char c : s) { t += '#'; t += c; }
    t += "#$";

    vector<int> p(t.size());
    int center = 0, right = 0;
    for (int i = 1; i + 1 < (int)t.size(); ++i) {
        int mirror = 2 * center - i;
        if (i < right) p[i] = min(right - i, p[mirror]);
        while (t[i + 1 + p[i]] == t[i - 1 - p[i]]) ++p[i];
        if (i + p[i] > right) {
            center = i;
            right = i + p[i];
        }
    }
    return p;
}
```

## 为什么是 $O(n)$

镜像能确定的部分不再比较。`while` 成功扩展且产生新工作时，会推动全局 `right` 向右；`right` 总共最多移动 $O(n)$ 次。失败比较每个中心至多一次，因此总时间 $O(n)$。

## 例题：最长回文子串

`s="babad"`。变换后算法会得到某个中心半径 3，对应原串长度也是 3，可取 `bab` 或 `aba`。最大回文长度就是 `max(p[i])`。

若最大半径为 `len`、变换串中心为 `i`，原串起点可由 `(i-len)/2` 得到：

```cpp
int bestCenter = max_element(p.begin(), p.end()) - p.begin();
int len = p[bestCenter];
int start = (bestCenter - len) / 2;
cout << s.substr(start, len) << '\n';
```

### 统计回文子串数

在上述带 `#` 变换中，中心 $i$ 对原串贡献 `(p[i]+1)/2` 个非空回文子串；对所有中心求和即可。重复出现位置算不同子串。

## 易错点

- `right` 到底表示包含端点还是开边界定义不统一。
- 镜像下标在 `i>=right` 时仍被访问。
- 哨兵字符可能出现在原字符串中。
- 变换串位置映射回原串时错一位。
- 把回文子串数量和不同回文字符串数量混淆；后者不能只靠 Manacher 半径直接去重。
