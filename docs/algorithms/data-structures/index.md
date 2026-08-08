# 数据结构

数据结构的价值在于用额外信息换取更快的查询或修改。学习时先回答三件事：节点保存什么、合并规则是什么、一次操作访问多少节点。

| 需求 | 常用结构 | 单次复杂度 |
| --- | --- | --- |
| 后进先出 / 先进先出 | 栈 / 队列 | $O(1)$ |
| 滑动窗口最值 | 单调队列 | 均摊 $O(1)$ |
| 动态取全局最值 | 堆 / 优先队列 | 插入、删除 $O(\log n)$ |
| 静态区间最值 | ST 表 | 预处理 $O(n\log n)$，查询 $O(1)$ |
| 集合合并与连通性 | 并查集 | 均摊近似 $O(1)$ |
| 单点修改、前缀统计 | 树状数组 | $O(\log n)$ |
| 通用区间修改与查询 | 线段树 | $O(\log n)$ |
| 前缀字符串查询 | Trie | $O(|s|)$ |
| 动态有序集合 | 平衡树 | 期望/均摊 $O(\log n)$ |

## 主题

- [线性表](../linear-list.md)、[栈与队列](stack-queue.md)、[单调结构](monotonic.md)
- [堆](heap.md)、[ST 表](sparse-table.md)
- [并查集](union-find.md)
- [树状数组](fenwick.md)、[线段树](segment-tree.md)
- [哈希表](hash-table.md)、[Trie](../strings/trie.md)
- [进阶树结构导读](advanced-trees.md)、[笛卡尔树](cartesian-tree.md)、[FHQ Treap](fhq-treap.md)
