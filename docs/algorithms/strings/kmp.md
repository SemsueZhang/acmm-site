# KMP

## 问题背景与适用场景

在很多字符串题中，我们需要判断模式串 `P` 是否出现在文本串 `S` 中，或者找出所有出现位置。  
朴素做法一旦失配就回到下一个起点重试，最坏会退回很多次，复杂度可达 $O(nm)$。

KMP（Knuth-Morris-Pratt）通过“利用已经匹配过的信息”避免文本指针回退，把匹配复杂度降为线性，适合：

- 子串查找（第一次出现位置 / 全部出现位置）
- 字符串周期性判断（最短循环节）
- 前后缀相关统计（结合前缀函数）

其中 $n=|S|,\ m=|P|$。

## 核心思想与关键结论

### 1. 前缀函数（prefix function）

定义 `pi[i]`：在模式串 `P[0..i]` 中，最长的“真前缀 = 真后缀”长度。

例如 `P = "ababaca"`：

- `pi[0]=0`
- `pi[1]=0`
- `pi[2]=1`
- `pi[3]=2`
- `pi[4]=3`
- `pi[5]=0`
- `pi[6]=1`

### 2. 失配回退直觉

匹配到 `P[j]` 失配时，不必从 `j=0` 重来，而是把 `j` 回退到 `pi[j-1]`。  
因为 `P[0..pi[j-1)-1]` 已经等于当前后缀，跳过去不会漏解。

### 3. 与 next 数组等价理解

很多资料写 `next`，很多写 `pi`。它们本质都在表达“失配后模式串应该跳到哪里”。  
常见差异只在下标和初值定义，不在算法思想本身。

关键结论：

- 构建 `pi` 只需线性时间 $O(m)$
- 匹配过程也只需线性时间 $O(n)$
- 总复杂度 $O(n+m)$

## 分步执行过程（示例输入）

示例：

- 文本串 `S = "ababcabcabababd"`
- 模式串 `P = "ababd"`

先构建 `P` 的 `pi`：

- `pi = [0, 0, 1, 2, 0]`

匹配时令 `i` 扫描 `S`，`j` 表示已匹配的模式串长度。

1. `i=0..3` 时，`S` 前四个字符与 `P` 前四个字符匹配，`j` 逐步变为 4。
2. 在 `i=4` 处失配（`S[i]='c'`，`P[j]='d'`）：
   - 回退 `j = pi[3] = 2`
   - 继续比较，若仍失配，再回退 `j = pi[1] = 0`
3. 文本串指针 `i` 不回退，继续向后扫。
4. 当扫到 `i=14` 时，`j` 达到 5（即 `m`），说明匹配成功，起点是

$$
i-m+1 = 14-5+1 = 10
$$

得到模式串在文本串中的一次出现位置 `10`（0-based）。

## 示例题（基础应用）

### 题目（示例题）

给定文本串 `S` 和模式串 `P`，输出 `P` 在 `S` 中所有出现位置（从 0 开始）。

### 思路分析

- 先用模式串构建 `pi`。
- 扫描文本串并维护 `j`。
- 每当 `j==m`，记录一次答案位置 `i-m+1`，然后令 `j=pi[j-1]` 继续找下一个。

这样能自然处理重叠匹配，例如 `S="aaaaa"`，`P="aaa"` 会得到位置 `0,1,2`。

## 伪代码

```text
function build_pi(P):
	m = len(P)
	pi[0] = 0
	j = 0
	for i in [1 .. m-1]:
		while j > 0 and P[i] != P[j]:
			j = pi[j-1]
		if P[i] == P[j]:
			j = j + 1
		pi[i] = j
	return pi

function kmp_search(S, P):
	pi = build_pi(P)
	ans = empty list
	j = 0
	for i in [0 .. len(S)-1]:
		while j > 0 and S[i] != P[j]:
			j = pi[j-1]
		if S[i] == P[j]:
			j = j + 1
		if j == len(P):
			ans.push_back(i - len(P) + 1)
			j = pi[j-1]
	return ans
```

## C++17 参考实现（可运行）

```cpp
#include <bits/stdc++.h>
using namespace std;

// 构建前缀函数 pi，pi[i] 表示 P[0..i] 的最长真前后缀长度
vector<int> buildPi(const string& p) {
	int m = (int)p.size();
	vector<int> pi(m, 0);
	for (int i = 1, j = 0; i < m; ++i) {
		// 失配时不断回退，直到可匹配或退到 0
		while (j > 0 && p[i] != p[j]) j = pi[j - 1];
		if (p[i] == p[j]) ++j;
		pi[i] = j;
	}
	return pi;
}

// 返回模式串 p 在文本串 s 中的所有匹配起点（0-based）
vector<int> kmpSearch(const string& s, const string& p) {
	vector<int> ans;
	if (p.empty()) return ans; // 约定：空模式串不处理

	vector<int> pi = buildPi(p);
	int n = (int)s.size(), m = (int)p.size();
	for (int i = 0, j = 0; i < n; ++i) {
		while (j > 0 && s[i] != p[j]) j = pi[j - 1];
		if (s[i] == p[j]) ++j;
		if (j == m) {
			ans.push_back(i - m + 1);
			j = pi[j - 1]; // 继续寻找下一个（含重叠）匹配
		}
	}
	return ans;
}

int main() {
	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	string s, p;
	cin >> s >> p;

	vector<int> pos = kmpSearch(s, p);
	if (pos.empty()) {
		cout << -1 << '\n';
	} else {
		for (int i = 0; i < (int)pos.size(); ++i) {
			if (i) cout << ' ';
			cout << pos[i];
		}
		cout << '\n';
	}
	return 0;
}
```

## 时间复杂度与空间复杂度

- 构建前缀函数：$O(m)$
- 匹配过程：$O(n)$
- 总时间复杂度：$O(n+m)$
- 额外空间复杂度：$O(m)$

## 适当扩展：循环串匹配思路

若要判断 `B` 是否是 `A` 的循环位移，可把 `A+A` 作为文本串，在其中做一次 KMP 查找 `B`：

- 若 `|A| \neq |B|`，一定不是
- 若 `B` 是 `A` 的循环位移，则 `B` 必定是 `A+A` 的子串

这是 KMP 在线性匹配场景下的经典扩展。

## 常见错误与边界情况

- 把 `pi` 和某版本 `next` 的下标定义混用，导致回退位置错一位。
- 失配时只回退一次而不是循环回退（应使用 `while`）。
- 找到一个匹配后忘记 `j = pi[j-1]`，导致漏掉重叠答案。
- 忽略空模式串约定，访问 `p[0]` 可能越界。
- 只记公式不记直觉：回退的本质是“已匹配后缀可复用”。
