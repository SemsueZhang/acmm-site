# 动态规划

动态规划（DP）把大量重复子问题压缩成有限状态，并按依赖顺序计算。学习顺序：

1. [DP 基础与线性 DP](basics.md)
2. [背包 DP](knapsack.md)
3. [区间 DP](interval.md)
4. [多维与树形 DP](tree-multidimensional.md)
5. [状态压缩 DP](bitmask.md)
6. [DP 常用优化](optimization.md)

一个可检查的 DP 方案必须说明：

- 状态每一维的含义；
- 转移覆盖所有情况且不重复/不漏；
- 初始化和无效状态；
- 计算顺序满足依赖；
- 答案在哪个状态；
- 状态数乘每状态转移数能否通过。
