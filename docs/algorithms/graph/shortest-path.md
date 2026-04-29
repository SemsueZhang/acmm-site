# 最短路

最短路用于在带权或无权图中找到从源点到其他点的最短路径。

## 常见算法

- BFS: 无权图最短路
- Dijkstra: 非负权图
- Bellman-Ford: 可处理负权
- Floyd: 多源最短路

## 选择建议

- 稀疏图 + 非负权: Dijkstra
- 存在负权: Bellman-Ford 或 SPFA
- 点数较小: Floyd

## 复杂度

- BFS: $O(n + m)$
- Dijkstra(堆): $O((n + m)\log n)$
- Bellman-Ford: $O(nm)$
- Floyd: $O(n^3)$

## 易错点

- Dijkstra 不能处理负权边
- 注意路径重构的前驱数组

## 练习方向

- 最短路路径还原
- 多源最短路
- 差分约束系统
