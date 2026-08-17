# 算法讲义审校记录

本记录对应 2026-08-10 的系统修订。审校以“严谨性、易懂性、详细程度、代码可读性”四个维度进行；索引页和术语页不强行要求独立真题程序，其职责是说明范围、依赖和统一约定。

## 第一遍：教学内容审校

| 模块 | 已审校页面 | 结论 |
| --- | --- | --- |
| 总览 | `algorithms/index.md`、`practice.md` | 通过；主线范围改为引用 CCF 2025 修订版大纲，NOI 拓展继续独立标注。练习页作为题单，不按单算法教程验收。 |
| 基础算法 | `complexity-enumeration.md`、`prefix-sum.md`、`difference.md`、`discretization.md`、`greedy.md`、`divide-conquer.md`、`sweep-line.md`、`high-precision.md`、`sorting.md` | 通过；原先混写的前缀/差分/离散化和分治/扫描线已拆页。新页面均包含状态含义、推导、正确性、复杂度、真实例题、完整 C++17 程序和易错点。 |
| 线性结构 | `linear-list.md`、`stack-queue.md`、`monotonic.md`、`hash-table.md`、`heap.md` | 通过；前两页属于容器与抽象结构导论，后续算法页补齐支配关系、期望复杂度或堆不变量，避免把平均复杂度写成最坏保证。 |
| 集合与区间结构 | `union-find.md`、`sparse-table.md`、`fenwick.md`、`segment-tree.md` | 通过；明确了等价类、幂区间覆盖、`lowbit` 分解和懒标记不变量，并核对 1 下标、闭区间与适用运算条件。 |
| 进阶树结构 | `advanced-trees.md`、`cartesian-tree.md`、`fhq-treap.md` | 通过；总览只承担选型，独立页分别讲静态序列结构与动态可重集合，并明确 FHQ Treap 为期望复杂度。 |
| 树 | `tree.md`、`bst.md`、`lca.md`、`techniques.md`、`diameter.md`、`centroid.md`、`tree-difference.md` | 通过；综合页改为学习路线，直径、重心、DFS 序/树上差分拆页；修复树形 DP 的旧链接。LCA 增补倍增递推和查询不越过答案的证明。 |
| 图基础与经典算法 | `graph.md`、`shortest-path.md`、`mst.md`、`toposort.md` | 通过；补齐 Dijkstra 非负权前提、Kruskal 割性质、Kahn 算法判环依据和真题建模。图基础页作为概念/存储/遍历导论。 |
| 图的结构分解 | `euler-trail.md`、`bipartite.md`、`scc.md`、`cut-vertices-bridges.md` | 通过；原两篇混合文章拆为四篇，分别说明度数与连通条件、奇环等价性、Tarjan 栈语义、无向图 `low` 与父边编号。 |
| 动态规划 | `basics.md`、`knapsack.md`、`interval.md`、`multidimensional.md`、`tree-dp.md`、`bitmask.md`、`optimization.md` | 通过；拆分多维 DP 与树形 DP，统一先定义状态再证明转移；01 背包强调逆序枚举，优化页强调必须保持候选集合等价。 |
| 搜索 | `dfs-bfs.md`、`binary-search.md`、`advanced.md` | 通过；区分深度优先的回溯状态与广度优先的分层不变量，进阶搜索明确最坏复杂度和适用条件。 |
| 字符串 | `basics.md`、`kmp.md`、`border-period.md`、`trie.md`、`hash.md`、`manacher.md`、`z-function.md`、`suffix-array.md` | 通过；术语、下标、Border/周期关系与各算法页面一致；哈希继续明确碰撞概率，Z 函数和后缀数组继续标为 NOI 级拓展。 |
| 数学 | `number-theory.md`、`gcd-bezout.md`、`prime-sieve.md`、`modular-inverse.md`、`crt.md`、`fast-power.md`、`combinatorics.md`、`linear-algebra.md` | 通过；初等数论改为路线页，gcd、筛法、逆元、CRT 独立讲解；补充快速幂循环不变量和高斯消元的等价变换依据。 |
| 计算几何 | `geometry/basics.md`、`geometry/convex-hull.md` | 通过；保留为 NOI 级拓展，正文明确精度、退化和叉积方向约定，不计入 CSP-S 主线。 |

## 第二遍：一致性与工程审校

- 导航与链接：检查被删除旧页的引用，算法目录相对 Markdown 链接失效数为 0；`mkdocs.yml` 已指向拆分后的页面。
- 缩写与术语：首次出现时解释 DP、DFS、BFS、LCA、SCC、DAG、CRT、BST、FHQ 等缩写；概率算法不使用“绝对正确”措辞。
- 代码：提取算法目录中全部 58 个包含 `int main` 的代码块，以 `g++ -std=c++17 -fsyntax-only` 检查，全部通过。
- 运行用例：对欧拉路径、SCC 缩点、割点、树上点差分、树的重心、树的直径、CRT、多维 DP、二分图共 9 个程序执行代表性或边界小样例，结果全部与预期一致。
- 构建：`python -m mkdocs build` 成功；仓库自身没有导航或链接警告，仅保留 Material for MkDocs 关于未来 MkDocs 2.0 的上游兼容性提示。
- 差异：`git diff --check` 通过；删除的是被新独立页面取代的旧综合页，没有删除用户题库或活动内容。

## 本轮未完成的增强验证

Tarjan、Hierholzer、树上差分等高风险图树算法本轮完成了定向用例，但尚未建立可重复运行的随机对拍脚本。该增强项已写入 `TODO.md`，不影响当前示例在所述前提下的正确性结论。
