# 图论

图论题首先要识别图的性质：有向还是无向、是否带权、权值是否非负、是否保证连通、是否为 DAG/树/二分图。算法选择取决于这些性质。

## 学习顺序

1. [图的概念、存储与遍历](../graph.md)
2. [最短路](shortest-path.md)、[最小生成树](mst.md)、[拓扑排序](toposort.md)
3. [欧拉道路](euler-trail.md)、[二分图判定](bipartite.md)
4. [强连通分量](scc.md)、[割点与桥](cut-vertices-bridges.md)
5. [树基础](../tree.md)、[LCA](../tree/lca.md)、[树上综合算法](../tree/techniques.md)

| 问题 | 典型算法 |
| --- | --- |
| 无权最短路 | BFS |
| 非负权单源最短路 | Dijkstra |
| 负权单源最短路 | Bellman–Ford / SPFA（最坏较慢） |
| 多源最短路 | Floyd–Warshall |
| 无向连通图最小生成树 | Kruskal / Prim |
| DAG 依赖顺序 | 拓扑排序 |
| 有向图互相可达分组 | SCC |
| 无向图关键点/边 | Tarjan 割点、桥 |
