# Trie

## 问题背景与适用场景

当我们要维护大量字符串，并且频繁进行“前缀相关”操作时，普通哈希表不够直接：

- 查询某单词是否存在
- 查询有多少单词以某前缀开头
- 动态插入、删除字符串

Trie（字典树）把“公共前缀”合并存储，能把上述操作都压到与字符串长度线性相关的复杂度。

## 核心思想与关键结论

把每个节点看作一个前缀状态：

- 从根到当前节点经过的字符，拼成一个前缀
- 边代表字符转移
- 节点维护统计信息（如 `pass`、`end`）

常用计数含义：

- `pass[u]`：有多少单词经过节点 `u`
- `end[u]`：有多少单词在节点 `u` 结束

关键结论：

- 插入、查询、前缀统计、删除的时间都与字符串长度 $L$ 成正比
- 适合字符集较小、前缀操作多的场景

## 分步执行过程（示例输入）

按顺序执行操作：

1. 插入 `"cat"`
2. 插入 `"car"`
3. 插入 `"dog"`
4. 查询前缀 `"ca"` 的出现次数
5. 删除 `"car"`
6. 再查前缀 `"ca"`

过程说明：

1. 插入 `"cat"`：创建路径 `c -> a -> t`，路径节点 `pass` 均加 1，末尾节点 `end` 加 1。
2. 插入 `"car"`：`c -> a` 已存在，只需新增 `r` 分支；共享前缀会复用节点。
3. 插入 `"dog"`：新开 `d -> o -> g` 路径。
4. 统计 `"ca"`：走到节点 `a`（位于 `c` 之后），返回其 `pass=2`。
5. 删除 `"car"`：先确认该词存在，然后沿路径把 `pass` 减 1，末尾 `end` 减 1。
6. 再统计 `"ca"`：此时返回 `1`（只剩 `"cat"`）。

## 示例题（基础应用）

### 题目（示例题）

初始为空字典，处理 $q$ 次操作：

- `1 word`：插入字符串
- `2 word`：查询字符串是否存在（存在输出 `Yes`，否则 `No`）
- `3 prefix`：输出以该前缀开头的字符串个数

### 思路分析

- 用 Trie 维护字符转移。
- 插入时沿路径创建节点并维护 `pass/end`。
- 精确查询看末尾节点 `end>0`。
- 前缀计数直接返回前缀节点的 `pass`。

相较于逐个字符串比较，Trie 把公共前缀计算复用掉，整体更高效。

## 伪代码

```text
insert(word):
	u = root
	pass[u]++
	for ch in word:
		if next[u][ch] 不存在:
			创建新节点
		u = next[u][ch]
		pass[u]++
	end[u]++

countPrefix(prefix):
	u = root
	for ch in prefix:
		if next[u][ch] 不存在:
			return 0
		u = next[u][ch]
	return pass[u]

search(word):
	u = root
	for ch in word:
		if next[u][ch] 不存在:
			return false
		u = next[u][ch]
	return end[u] > 0

erase(word):
	if search(word) == false:
		return
	u = root
	pass[u]--
	for ch in word:
		v = next[u][ch]
		pass[v]--
		u = v
	end[u]--
```

## C++17 参考实现（可运行）

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Trie {
	static const int SIGMA = 26; // 仅处理小写字母 a-z

	vector<array<int, SIGMA>> nxt;
	vector<int> pass, endCnt;

	Trie() {
		nxt.push_back({});
		nxt[0].fill(-1);
		pass.push_back(0);
		endCnt.push_back(0);
	}

	int newNode() {
		nxt.push_back({});
		nxt.back().fill(-1);
		pass.push_back(0);
		endCnt.push_back(0);
		return (int)nxt.size() - 1;
	}

	void insert(const string& s) {
		int u = 0;
		++pass[u];
		for (char ch : s) {
			int c = ch - 'a';
			if (nxt[u][c] == -1) nxt[u][c] = newNode();
			u = nxt[u][c];
			++pass[u];
		}
		++endCnt[u];
	}

	bool search(const string& s) const {
		int u = 0;
		for (char ch : s) {
			int c = ch - 'a';
			if (c < 0 || c >= SIGMA) return false;
			if (nxt[u][c] == -1) return false;
			u = nxt[u][c];
		}
		return endCnt[u] > 0;
	}

	int countPrefix(const string& p) const {
		int u = 0;
		for (char ch : p) {
			int c = ch - 'a';
			if (c < 0 || c >= SIGMA) return 0;
			if (nxt[u][c] == -1) return 0;
			u = nxt[u][c];
		}
		return pass[u];
	}

	// 惰性删除：只维护计数，不强制回收无用节点
	void erase(const string& s) {
		if (!search(s)) return;
		int u = 0;
		--pass[u];
		for (char ch : s) {
			int c = ch - 'a';
			int v = nxt[u][c];
			--pass[v];
			u = v;
		}
		--endCnt[u];
	}
};

int main() {
	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	int q;
	cin >> q;
	Trie trie;

	while (q--) {
		int op;
		string s;
		cin >> op >> s;
		if (op == 1) {
			trie.insert(s);
		} else if (op == 2) {
			cout << (trie.search(s) ? "Yes" : "No") << '\n';
		} else if (op == 3) {
			cout << trie.countPrefix(s) << '\n';
		} else if (op == 4) {
			// 额外支持删除操作（扩展）
			trie.erase(s);
		}
	}
	return 0;
}
```

## 时间复杂度与空间复杂度

设字符串长度为 $L$。

- 插入：$O(L)$
- 查询是否存在：$O(L)$
- 前缀计数：$O(L)$
- 删除（惰性计数）：$O(L)$

空间复杂度与节点总数有关，记为 $N_{node}$，数组 Trie 下每节点固定 $\Sigma$ 个分支：

$$
O(N_{node} \cdot \Sigma)
$$

其中 $\Sigma$ 是字符集大小（如 26）。

## 适当扩展

### 1. 统计前缀出现次数

通过 `pass` 可以直接回答“有多少单词经过该前缀”，这是 Trie 的经典增强。

### 2. 删除操作思路

竞赛里常用“惰性删除”：只减计数，不回收节点，代码简单且稳定。  
若需要节省内存，可在 `pass=0` 时回收子树，但实现复杂度会提升。

### 3. 数组 Trie 与指针 Trie 取舍

- 数组 Trie：访问快、常数小，适合字符集小且固定（如小写字母）。
- 指针/哈希 Trie：空间更灵活，适合字符集大或稀疏，但常数通常更大。

## 常见错误与边界情况

- 忘记维护根节点 `pass`，导致统计口径不一致。
- 插入重复单词时没累加 `end`，导致查询次数信息丢失。
- 删除前不判断是否存在，计数可能减成负数。
- 字符映射越界（例如输入含大写或符号但代码按 `a-z` 处理）。
- 把“前缀存在”误写成“单词存在”：前者看路径，后者必须看 `end>0`。
