# 数学

提高组数学重点不是背定理名称，而是知道定理的条件、怎样转成代码，以及取模和溢出的边界。

- [快速幂](fast-power.md)：模幂与矩阵快速幂的基础。
- [初等数论学习路线](number-theory.md)：按依赖顺序组织下列独立主题。
- [最大公约数与裴蜀定理](gcd-bezout.md)
- [质数、分解与线性筛](prime-sieve.md)
- [同余与乘法逆元](modular-inverse.md)
- [中国剩余定理](crt.md)
- [组合数学](combinatorics.md)：排列组合、鸽巢、二项式、容斥、错排与 Catalan 数。
- [矩阵与高斯消元](linear-algebra.md)：矩阵运算、快速幂与线性方程组。

所有“除法取模”都应先问除数是否存在逆元；所有乘法都应先估算是否超出 `long long`。
