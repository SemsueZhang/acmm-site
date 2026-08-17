# 栈、队列与双端队列

这些结构存储的数据相同，区别在于允许从哪一端访问。

## 栈：后进先出

`stack` 只允许在栈顶压入、查看和弹出。函数调用栈、括号匹配、表达式求值都依赖“最近未完成任务最先处理”。

### 例题：括号序列是否合法

扫描字符串。遇到左括号入栈；遇到右括号时，它必须与最近的左括号匹配，因此检查栈顶。结束后栈必须为空。

```cpp
bool valid(const string& s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') st.push(c);
        else {
            if (st.empty()) return false;
            char t = st.top(); st.pop();
            if ((c == ')' && t != '(') || (c == ']' && t != '[') ||
                (c == '}' && t != '{')) return false;
        }
    }
    return st.empty();
}
```

每个字符只入栈、出栈一次，时间 $O(n)$，空间 $O(n)$。

## 队列：先进先出

`queue` 从尾部加入、头部取出。BFS 中先发现的点距离源点更近，所以用队列能保证按距离层处理。

```cpp
queue<int> q;
q.push(start);
while (!q.empty()) {
    int u = q.front(); q.pop();
    // 处理 u，再把未访问邻居 push 进去
}
```

## 双端队列

`deque` 两端都能 $O(1)$ 插入删除。0-1 BFS 中，权重 0 的边到达的新状态放队首，权重 1 的边放队尾，可在 $O(n+m)$ 时间求边权只有 0/1 的最短路。

### 例题：0-1 BFS

```cpp
deque<int> dq;
vector<int> dist(n + 1, INF);
dist[s] = 0;
dq.push_front(s);
while (!dq.empty()) {
    int u = dq.front(); dq.pop_front();
    for (auto [v, w] : g[u]) { // w 只能为 0 或 1
        if (dist[v] > dist[u] + w) {
            dist[v] = dist[u] + w;
            if (w == 0) dq.push_front(v);
            else dq.push_back(v);
        }
    }
}
```

实现中同一顶点可能被重复放入，但每次有效松弛都会改善距离；也可在队列中附带当时距离，弹出时忽略过期状态。

## 真题：P1449 后缀表达式

[洛谷 P1449 后缀表达式](https://www.luogu.com.cn/problem/P1449) 给出逆波兰表达式。数字出现时入栈；运算符出现时弹出右操作数 `right`，再弹出左操作数 `left`，计算 `left op right` 后把结果压回。

减法和除法不能交换两个操作数，这是本题最常见的错误。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    stack<long long> values;
    long long number = 0;
    bool readingNumber = false;
    char ch;

    while (cin >> ch && ch != '@') {
        if (isdigit((unsigned char)ch)) {
            number = number * 10 + (ch - '0');
            readingNumber = true;
        } else if (ch == '.') {
            values.push(number);
            number = 0;
            readingNumber = false;
        } else {
            long long right = values.top(); values.pop();
            long long left = values.top(); values.pop();
            if (ch == '+') values.push(left + right);
            if (ch == '-') values.push(left - right);
            if (ch == '*') values.push(left * right);
            if (ch == '/') values.push(left / right);
        }
    }
    cout << values.top() << '\n';
    return 0;
}
```

每个数字和运算符各进出栈常数次，时间 $O(n)$、空间 $O(n)$。

## 常见错误

- 调用 `top/front/back` 前没有判断为空。
- BFS 在出队时才标记访问，导致同一点被重复入队；普通 BFS 应在入队时标记。
- 把 `vector.erase(begin())` 当队列使用，它会移动所有元素，单次 $O(n)$。
- 0-1 BFS 用于非 0/1 权边，结论不成立。
